---
title: 使用 Python 质量压缩 PDF
layout: post
date: 2026-08-21 12:00:00
updated: 2026-08-21 12:00:00
comments: true
tags:
- Python
- PDF
- PyMuPDF
- 文件压缩
categories:
- [技术, Python]
permalink: /2026/08/compress-pdf-with-python/
---

PDF 文件经常因为嵌入了大量高清图片而体积过大。直接把整页转换成图片再重新生成 PDF，虽然可以显著减小体积，却会让文字和矢量图形失去原有的清晰度，也可能破坏页面尺寸、链接和文档结构。

本文提供一个基于 [PyMuPDF](https://pymupdf.readthedocs.io/) 的脚本：只重新编码 PDF 内嵌的图片，同时尽量保留文字、矢量内容、页面尺寸和超链接。脚本还会在压缩前后计算文本与页面几何信息的指纹，避免生成内容异常的文件。

<!--more-->

## 一、脚本特点

这个脚本有以下特点：

* 通过质量参数控制图片重新编码质量，数值范围为 `1-100`；
* 不设置目标文件体积，最终大小取决于原始 PDF 中图片的数量、尺寸和编码方式；
* 文字和矢量内容不会被栅格化；
* 尽量保留原始页面尺寸、旋转信息和超链接；
* 使用临时文件写入，避免压缩失败时覆盖原始文件；
* 压缩完成后校验页数、文本指纹和页面几何信息。

需要注意的是，图片会重新编码，因此这不是像素级无损压缩。质量越高，通常画质越好，但文件体积也越大。

## 二、完整代码

将下面的代码保存为 `compress_pdf.py`：

```python
#!/usr/bin/env python3
"""按指定图片压缩质量压缩 PDF，尽量保留文字、矢量内容、页面尺寸和链接。

图片会重新编码以减小体积，因此不是像素级无损；文字和版式不会被栅格化。

示例：
python compress_pdf.py input.pdf 92
python compress_pdf.py input.pdf 85 output.pdf
"""

from __future__ import annotations

import argparse
import hashlib
import os
import sys
import tempfile
from pathlib import Path

try:
    import fitz  # PyMuPDF
except ImportError as exc:  # pragma: no cover - 仅在环境缺包时触发
    raise SystemExit(
        "缺少 PyMuPDF。请使用当前 Python 环境安装：\n"
        "/Users/pepper/.venv/codex/bin/pip install PyMuPDF"
    ) from exc


def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="按指定压缩质量压缩 PDF 内嵌图片，不设置目标文件体积。"
    )
    parser.add_argument("input", type=Path, help="输入 PDF 文件")
    parser.add_argument(
        "quality",
        type=int,
        help="压缩质量/比例，1-100；数值越高，画质越好、文件通常越大",
    )
    parser.add_argument(
        "output",
        type=Path,
        nargs="?",
        help="输出 PDF 文件；省略时自动生成 *_compressed.pdf",
    )
    return parser.parse_args()


def text_fingerprint(
    pdf_path: Path,
) -> tuple[int, str, tuple[tuple[float, float, int], ...]]:
    """返回页数、文本指纹和页面几何信息。"""
    with fitz.open(pdf_path) as document:
        page_text = [page.get_text("text") for page in document]
        geometry = tuple(
            (
                round(page.rect.width, 3),
                round(page.rect.height, 3),
                page.rotation,
            )
            for page in document
        )
        digest = hashlib.sha256(
            "\n\f\n".join(page_text).encode("utf-8")
        ).hexdigest()
        return len(page_text), digest, geometry


def compress_once(input_path: Path, temporary_path: Path, quality: int) -> None:
    """使用 PyMuPDF 重编码图片并写入临时 PDF。"""
    with fitz.open(input_path) as document:
        document.rewrite_images(
            dpi_threshold=0,
            dpi_target=0,
            quality=quality,
            # lossy=True：允许 JPEG 等有损编码以显著减小体积。
            # lossless=True：同时处理原来使用 PNG/Flate 等无损编码的图片。
            lossy=True,
            lossless=True,
            bitonal=True,
            color=True,
            gray=True,
        )
        document.save(
            temporary_path,
            garbage=4,
            clean=True,
            deflate=True,
            deflate_images=False,
            deflate_fonts=True,
            use_objstms=True,
            compression_effort=100,
        )


def compress_pdf(input_path: Path, output_path: Path, quality: int) -> int:
    input_path = input_path.expanduser().resolve()
    output_path = output_path.expanduser().resolve()

    if not input_path.is_file():
        raise FileNotFoundError(f"找不到输入文件：{input_path}")
    if input_path.suffix.lower() != ".pdf":
        raise ValueError("输入文件必须是 PDF")
    if output_path == input_path:
        raise ValueError("输出路径不能与输入路径相同，以免覆盖原文件")
    if not 1 <= quality <= 100:
        raise ValueError("压缩质量/比例必须是 1-100 的整数")

    output_path.parent.mkdir(parents=True, exist_ok=True)
    source_fingerprint = text_fingerprint(input_path)
    temporary_path: Path | None = None

    try:
        with tempfile.NamedTemporaryFile(
            prefix=f".{output_path.stem}.",
            suffix=".tmp.pdf",
            dir=output_path.parent,
            delete=False,
        ) as temporary_file:
            temporary_path = Path(temporary_file.name)

        # PyMuPDF 需要接管输出路径，因此删除占位临时文件后再保存。
        temporary_path.unlink()
        compress_once(input_path, temporary_path, quality)
        size = temporary_path.stat().st_size
        os.replace(temporary_path, output_path)
        temporary_path = None

        output_fingerprint = text_fingerprint(output_path)
        if output_fingerprint != source_fingerprint:
            try:
                output_path.unlink()
            except FileNotFoundError:
                pass
            raise RuntimeError("压缩后文本或页面几何信息与原文件不一致，已删除异常输出。")

        return size
    finally:
        if temporary_path is not None:
            try:
                temporary_path.unlink()
            except FileNotFoundError:
                pass


def main() -> int:
    args = parse_args()
    input_path = args.input
    output_path = args.output
    if output_path is None:
        output_path = input_path.with_name(f"{input_path.stem}_compressed.pdf")

    try:
        size = compress_pdf(
            input_path=input_path,
            output_path=output_path,
            quality=args.quality,
        )
    except (FileNotFoundError, ValueError, RuntimeError) as exc:
        print(f"错误：{exc}", file=sys.stderr)
        return 1

    print(f"已生成：{output_path.expanduser().resolve()}")
    print(f"文件大小：{size:,} 字节（{size / 1_000_000:.2f} MB）")
    print(f"压缩质量/比例：{args.quality}")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

## 三、安装依赖

建议在当前 Python 环境中安装 PyMuPDF：

```bash
/Users/pepper/.venv/codex/bin/pip install PyMuPDF
```

如果使用的是其他虚拟环境，将命令中的 Python 和 pip 路径替换为对应环境即可。

## 四、使用方法

最简单的用法是指定输入 PDF 和压缩质量：

```bash
python compress_pdf.py input.pdf 92
```

省略输出路径时，脚本会在原文件同目录生成 `input_compressed.pdf`。也可以显式指定输出文件：

```bash
python compress_pdf.py input.pdf 85 output.pdf
```

质量参数的取值范围为 `1-100`。一般可以先从 `92` 或 `85` 开始尝试，再根据图片清晰度和文件大小调整：

| 质量参数 | 适用场景 |
| --- | --- |
| `90-100` | 扫描文档、医学图片、需要较高画质的材料 |
| `75-89` | 日常阅读、内部传阅、兼顾清晰度和体积 |
| `1-74` | 对体积要求较高、对图片质量要求较低的场景 |

## 五、使用时的注意事项

这个脚本主要压缩 PDF 内嵌图片。如果原文件主要由文字、矢量图形或已经高度压缩的图片组成，压缩后的体积变化可能不明显。

另外，`quality` 控制的是图片重新编码质量，不是“压缩到原文件的百分之多少”。例如，设置为 `85` 并不意味着输出文件一定是原文件大小的 `85%`。

脚本会校验文字内容和页面几何信息，但仍建议在处理重要文件后人工抽查，特别是包含医学影像、细小文字或二维码的 PDF。原始文件也应保留，便于在发现图片质量不合适时重新选择更高的质量参数。

## 六、总结

相比“PDF 转图片再合成 PDF”的方式，直接重编码内嵌图片能够更好地保留文字、矢量内容和页面结构。这个脚本适合对包含扫描页、截图或照片的 PDF 做批量预处理；如果需要提交出版、打印或归档，建议根据最终用途选择合适的质量参数，并进行完整的视觉检查。
