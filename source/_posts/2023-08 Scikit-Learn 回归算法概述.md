---
title: Scikit-Learn 回归算法概述
layout: post
date: 2023-08-11 23:53:00
updated: 2023-08-11 23:53:00
comments: true
tags:
- Python
- Scikit-Learn
- 机器学习
- 回归分析
- 回归算法
- 模型训练
categories:
- [数据科学, 机器学习]
permalink: /2023/08/sklearn-regression-algorithm/
---

在机器学习领域，回归分析是一个核心的话题，它的目的是预测一个连续的输出值基于一个或多个输入特征。回归方法有着广泛的应用，从预测股票价格、房地产市场的评估，到天气预测等。尽管有各种各样的回归算法可以选择，但每种方法都有其独特的优势和局限性。本文将简要介绍几种流行的回归方法，包括线性回归、岭回归、Lasso回归、多项式回归、决策树回归、随机森林回归、梯度提升回归、支持向量回归、K-近邻回归，以及神经网络回归。通过详细的代码示例，读者可以更深入地了解每种方法的工作原理和实际应用。

<!--more-->

Scikit-Learn（sklearn）是一个流行的机器学习库，它包含大量的回归算法。这些算法可用于预测数字目标变量。以下是一些主要的 sklearn 回归算法：

1. **线性回归（Linear Regression）**：最简单的回归方法，通过在数据中找到最佳拟合直线（对于一维输入）或超平面（对于多维输入）来建立预测模型。
2. **岭回归（Ridge Regression）**：这是线性回归的一个扩展，它包括一个L2正则化项，可以帮助防止过拟合。
3. **Lasso回归（Lasso Regression）**：另一个线性回归的扩展，它包括一个L1正则化项，这可以导致稀疏解，其中一些特征的权重将被设定为零。
4. **弹性网（Elastic Net）**：这是一种结合了岭回归和Lasso回归的方法，它包括L1和L2两种类型的正则化项。
5. **多项式回归（Polynomial Regression）**：这是线性回归的扩展，它允许模型在输入特征的高次幂上进行拟合，从而能够捕获数据中的非线性关系。
6. **决策树回归（Decision Tree Regression）**：使用决策树模型进行预测的方法。它可以捕获复杂的非线性关系，并且具有解释性强的特点。
7. **随机森林回归（Random Forest Regression）**：这是一种集成方法，它结合了多个决策树回归模型的预测结果。
8. **梯度提升回归（Gradient Boosting Regression）**：另一种集成方法，它通过顺序地添加新的决策树，每一步都尝试纠正前一步的错误。
9. **支持向量回归（Support Vector Regression）**：这是支持向量机的回归版本，它试图在预测误差在某个阈值内的同时，最大化特征空间中的间隔。
10. **最近邻回归（K-Nearest Neighbors Regression）**：预测新数据点的目标值是通过查找训练集中最接近它的数据点并平均其目标值来进行的。
11. **神经网络回归（Neural Network Regression，或 MLP Regression）**：这种方法使用神经网络模型进行预测，能够处理更复杂的非线性关系。

## 1. 线性回归（Linear Regression）

```python
# 引入所需要的库
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn import datasets

# 生成一些随机的回归数据
X, y = datasets.make_regression(n_samples=100, n_features=1, noise=0.1)

# 划分数据为训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# 初始化线性回归模型
model = LinearRegression()

# 使用训练数据对模型进行拟合
model.fit(X_train, y_train)

# 在测试集上进行预测
predictions = model.predict(X_test)

# 输出预测的结果
print(predictions)
```

## 2. 岭回归（Ridge Regression）

```python
# 引入所需要的库
from sklearn.model_selection import train_test_split
from sklearn.linear_model import Ridge
from sklearn import datasets

# 生成一些随机的回归数据
X, y = datasets.make_regression(n_samples=100, n_features=1, noise=0.1)

# 划分数据为训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# 初始化岭回归模型
model = Ridge(alpha=1.0)  # alpha 参数控制正则化的强度

# 使用训练数据对模型进行拟合
model.fit(X_train, y_train)

# 在测试集上进行预测
predictions = model.predict(X_test)

# 输出预测的结果
print(predictions)
```

这段代码与线性回归的示例相似，区别在于这里我们用的是 `Ridge` 而不是 `LinearRegression`。`Ridge` 接受一个 `alpha` 参数，这个参数控制正则化的强度，也就是避免过拟合的程度。

## 3. Lasso回归（Lasso Regression）

```python
# 引入所需要的库
from sklearn.model_selection import train_test_split
from sklearn.linear_model import Lasso
from sklearn.preprocessing import MinMaxScaler
from sklearn import datasets

# 生成一些随机的回归数据
X, y = datasets.make_regression(n_samples=100, n_features=1, noise=0.1)

# 划分数据为训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# 初始化MinMaxScaler
scaler = MinMaxScaler()

# 使用训练集数据拟合scaler，并对训练集和测试集数据进行转换
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 初始化Lasso回归模型
model = Lasso(alpha=1.0)  # alpha 参数控制正则化的强度

# 使用训练数据对模型进行拟合
model.fit(X_train_scaled, y_train)

# 在测试集上进行预测
predictions = model.predict(X_test_scaled)

# 输出预测的结果
print(predictions)
```

这段代码与上述岭回归的示例非常相似，只是我们使用了 `Lasso` 类而不是 `Ridge` 类。和岭回归一样，`Lasso` 也接受一个 `alpha` 参数，这个参数控制正则化的强度。

## 4. 多项式回归（Polynomial Regression）

```python
# 引入所需要的库
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures, FunctionTransformer
from sklearn.pipeline import FeatureUnion
from sklearn import datasets
import numpy as np

# 生成一些随机的回归数据
X, y = datasets.make_regression(n_samples=100, n_features=1, noise=0.1, random_state=42)

# 划分数据为训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 初始化线性回归模型
model = LinearRegression()

# 创建并拟合各种多项式特征
combined_features = FeatureUnion([("poly_features_1", PolynomialFeatures(1)),
                                  ("poly_features_2", PolynomialFeatures(2)),
                                  ("poly_features_3", PolynomialFeatures(3))])

X_train_combined = combined_features.fit_transform(X_train)
X_test_combined = combined_features.transform(X_test)

model.fit(X_train_combined, y_train)
predictions = model.predict(X_test_combined)

print(f"Combined polynomial regression predictions: {predictions}")
```

## 5. 决策树回归（Decision Tree Regression）

```python
# 引入所需要的库
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeRegressor
from sklearn.preprocessing import MinMaxScaler
from sklearn import datasets

# 生成一些随机的回归数据
X, y = datasets.make_regression(n_samples=100, n_features=1, noise=0.1, random_state=42)

# 划分数据为训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 使用MinMaxScaler来缩放数据
scaler = MinMaxScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 初始化并训练决策树回归模型
tree_reg = DecisionTreeRegressor(random_state=42)
tree_reg.fit(X_train_scaled, y_train)

# 预测测试数据
predictions = tree_reg.predict(X_test_scaled)
print(f"Decision tree regression predictions: {predictions}")
```

## 6. 随机森林回归（Random Forest Regression）

```python
# 引入所需要的库
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
from sklearn.preprocessing import MinMaxScaler
from sklearn import datasets

# 生成一些随机的回归数据
X, y = datasets.make_regression(n_samples=100, n_features=1, noise=0.1, random_state=42)

# 划分数据为训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 使用MinMaxScaler来缩放数据
scaler = MinMaxScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 初始化并训练随机森林回归模型
rf_reg = RandomForestRegressor(n_estimators=100, random_state=42)
rf_reg.fit(X_train_scaled, y_train)

# 预测测试数据
predictions = rf_reg.predict(X_test_scaled)
print(f"Random forest regression predictions: {predictions}")
```

## 7. 梯度提升回归（Gradient Boosting Regression）

```python
# 引入所需要的库
from sklearn.model_selection import train_test_split
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.preprocessing import MinMaxScaler
from sklearn import datasets

# 生成一些随机的回归数据
X, y = datasets.make_regression(n_samples=100, n_features=1, noise=0.1, random_state=42)

# 划分数据为训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 使用MinMaxScaler来缩放数据
scaler = MinMaxScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 初始化并训练梯度提升回归模型
gb_reg = GradientBoostingRegressor(n_estimators=100, learning_rate=0.1, max_depth=1, random_state=42)
gb_reg.fit(X_train_scaled, y_train)

# 预测测试数据
predictions = gb_reg.predict(X_test_scaled)
print(f"Gradient boosting regression predictions: {predictions}")
```

梯度提升是一个强大的集成方法，它结合了多个弱预测模型（通常是决策树），以提高预测精度。通过逐步添加新模型，梯度提升方法试图纠正前一个模型的错误。在这个示例中，我们使用了100个决策树模型，学习率设置为0.1，树的最大深度为1。这些参数都可以根据你的具体问题进行调整。

## 8. 支持向量回归（Support Vector Regression）

```python
# 引入所需要的库
from sklearn.model_selection import train_test_split
from sklearn.svm import SVR
from sklearn.preprocessing import MinMaxScaler
from sklearn import datasets

# 生成一些随机的回归数据
X, y = datasets.make_regression(n_samples=100, n_features=1, noise=0.1, random_state=42)

# 划分数据为训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 使用MinMaxScaler来缩放数据
scaler = MinMaxScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 初始化并训练支持向量回归模型
svr_reg = SVR(kernel='rbf', C=1.0, epsilon=0.1)
svr_reg.fit(X_train_scaled, y_train)

# 预测测试数据
predictions = svr_reg.predict(X_test_scaled)
print(f"Support vector regression predictions: {predictions}")
```

支持向量回归是一种强大的回归方法，它使用支持向量机（SVM）的原理来解决回归问题。在这个示例中，我们使用了径向基函数（rbf）作为核函数，惩罚系数（C）设置为1.0，损失函数的ε-insensitive zone参数为0.1。这些参数都可以根据你的具体问题进行调整。

## 9. 最近邻回归（K-Nearest Neighbors Regression）

```python
# 引入所需要的库
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsRegressor
from sklearn.preprocessing import MinMaxScaler
from sklearn import datasets

# 生成一些随机的回归数据
X, y = datasets.make_regression(n_samples=100, n_features=1, noise=0.1, random_state=42)

# 划分数据为训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 使用MinMaxScaler来缩放数据
scaler = MinMaxScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 初始化并训练K-近邻回归模型
knn_reg = KNeighborsRegressor(n_neighbors=5)
knn_reg.fit(X_train_scaled, y_train)

# 预测测试数据
predictions = knn_reg.predict(X_test_scaled)
print(f"K-nearest neighbors regression predictions: {predictions}")
```

K-近邻回归是一种基于实例的学习方法，它在预测新实例时考虑了其最近的 `k` 个邻居。这个示例中，我们设置 `k`（`n_neighbors` 参数）为5，这意味着模型在进行预测时会考虑最近的5个邻居的平均值。这个参数可以根据你的具体问题进行调整。

## 10. 神经网络回归（Neural Network Regression，或 MLP Regression）

```python
# 引入所需要的库
from sklearn.model_selection import train_test_split
from sklearn.neural_network import MLPRegressor
from sklearn.preprocessing import MinMaxScaler
from sklearn import datasets

# 生成一些随机的回归数据
X, y = datasets.make_regression(n_samples=100, n_features=1, noise=0.1, random_state=42)

# 划分数据为训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 使用MinMaxScaler来缩放数据
scaler = MinMaxScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 初始化并训练神经网络回归模型
mlp_reg = MLPRegressor(hidden_layer_sizes=(10, ), activation='relu', solver='adam', max_iter=500, random_state=42)
mlp_reg.fit(X_train_scaled, y_train)

# 预测测试数据
predictions = mlp_reg.predict(X_test_scaled)
print(f"Neural network regression predictions: {predictions}")
```

## 总结

回归分析是机器学习中的一个强大工具，无论是对于简单的线性关系，还是复杂的非线性模式，都有合适的算法可供选择。本文提供了各种回归方法的基本介绍和实现示例，帮助读者理解和比较它们的特点和应用场景。当然，选择最合适的回归方法应考虑实际问题的需求、数据的性质以及模型的解释性。而在实际应用中，经常需要进一步的参数调优、特征选择和工程实践，以确保模型的最佳性能。希望本文为读者提供了一个扎实的起点，从而深入探索和应用这些强大的回归工具。
