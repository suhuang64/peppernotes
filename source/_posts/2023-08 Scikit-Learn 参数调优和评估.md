---
title: Scikit-Learn 参数调优和评估
layout: post
date: 2023-08-12 00:06:00
updated: 2023-08-12 00:06:00
comments: true
tags:
- Python
- Scikit-Learn
- 机器学习
- 超参数调优
- 模型评估
- 交叉验证
categories:
- [数据科学, 机器学习]
permalink: /2023/08/sklearn-paramenter-tune/
---

在机器学习的实践中，选择一个合适的模型只是冰山一角。即使选择了正确的模型，如果不对其进行适当的参数调优，那么模型的性能可能会大打折扣。参数调优对于模型的最终表现起到关键的作用。Sklearn，作为一个广泛使用的Python机器学习库，为我们提供了一系列强大的工具，如`GridSearchCV` 和 `RandomizedSearchCV`，帮助我们系统地进行参数调优。但是，如何有效地使用这些工具进行参数调优和模型评估呢？在本文中，我们将深入探讨这些问题，并为读者提供一些实践建议。

<!--more-->

## 1. 决策树模型的参数调优和评估

### 1.1 GridSearchCV

对于决策树模型的参数调优和评估，可以使用scikit-learn提供的交叉验证（Cross-validation）和网格搜索（GridSearch）工具。以下是具体的步骤：

1. **定义参数网格**:
   为每个参数定义一组值来进行尝试。例如：
   ```python
   param_grid = {
       'criterion': ["squared_error", "friedman_mse", "absolute_error", "poisson"],
       'splitter': ['best', 'random'],
       'max_depth': [None, 10, 20, 30],
       'min_samples_split': [2, 5, 10],
       'min_samples_leaf': [1, 2, 4],
       'max_features': [None, 'auto', 'sqrt', 'log2']
   }
   ```

2. **初始化模型**:
   ```python
   from sklearn.tree import DecisionTreeRegressor
   dt = DecisionTreeRegressor()
   ```

3. **使用交叉验证的网格搜索**:
   使用 `GridSearchCV` 来进行网格搜索和交叉验证。
   ```python
   from sklearn.model_selection import GridSearchCV
   
   grid_search = GridSearchCV(estimator=dt, param_grid=param_grid, 
                              cv=5, n_jobs=-1, verbose=2)
   ```

   - `cv=5` 指定了5折交叉验证。
   - `n_jobs=-1` 表示使用所有可用的CPU核心。
   - `verbose=2` 表示过程中显示详细信息。

4. **拟合数据**:
   使用 `grid_search` 对象来拟合你的数据。
   ```python
   grid_search.fit(X_train, y_train)
   ```

5. **查看最佳参数**:
   完成拟合后，你可以查看为模型找到的最佳参数组合。
   ```python
   best_params = grid_search.best_params_
   print(best_params)
   ```

6. **使用最佳模型**:
   使用网格搜索找到的最佳参数训练的模型可以直接从 `grid_search` 获取。
   ```python
   best_dt = grid_search.best_estimator_
   ```

7. **预测和评估**:
   使用此最佳模型进行预测并评估其性能。
   ```python
   y_pred = best_dt.predict(X_test)
   ```

**注意事项**:
- 网格搜索可能会耗费很长时间，尤其是当参数网格较大或数据集较大时。因此，从一个较小的参数网格开始，然后根据初步的结果进一步细化可能是一个好策略。    
- 如果你希望更随机地搜索参数空间，而不是尝试所有的参数组合，可以考虑使用 `RandomizedSearchCV`。这个方法会从每个参数的分布中随机采样，并可能更快地找到一个不错的参数组合。
- 使用交叉验证的好处是你可以得到模型在不同子集上的性能评估，这有助于检测模型是否过拟合并提供了对模型泛化能力的更全面的估计。
### 1.2 RandomizedSearchCV

`RandomizedSearchCV` 是一种用于超参数调优的方法，它不尝试参数网格中的所有参数组合，而是从参数网格中随机抽样固定数量的参数组合。这种方法可能更快地找到一个不错的参数组合，特别是当参数空间很大时。

以下是使用 `RandomizedSearchCV` 的步骤：

1. **定义参数分布**:
   与 `GridSearchCV` 不同，可以为参数定义一个分布而不是明确的值列表。
   
   ```python
   from scipy.stats import randint

   param_dist = {
       'criterion': ["squared_error", "friedman_mse", "absolute_error", "poisson"],
       'splitter': ['best', 'random'],
       'max_depth': [None] + list(randint(1, 20).rvs(10)),  # None加上10个随机整数值在1到20之间
       'min_samples_split': randint(2, 10),  # 2到10之间的整数分布
       'min_samples_leaf': randint(1, 10)   # 1到10之间的整数分布
   }
   ```

2. **初始化模型**:
   
   ```python
   from sklearn.tree import DecisionTreeRegressor
   dt = DecisionTreeRegressor()
   ```

3. **使用随机搜索的交叉验证**:
   
   ```python
   from sklearn.model_selection import RandomizedSearchCV
   
   random_search = RandomizedSearchCV(estimator=dt, param_distributions=param_dist, 
                                      n_iter=100, cv=5, n_jobs=-1, verbose=2, random_state=42)
   ```

   - `n_iter=100` 表示尝试100组参数组合。
   - `cv=5` 指定了5折交叉验证。
   - `n_jobs=-1` 表示使用所有可用的CPU核心。
   - `verbose=2` 表示过程中显示详细信息。
   - `random_state=42` 设置随机种子以确保可重复性。

4. **拟合数据**:
   
   ```python
   random_search.fit(X_train, y_train)
   ```

5. **查看最佳参数**:
   
   ```python
   best_params = random_search.best_params_
   print(best_params)
   ```

6. **使用最佳模型**:
   
   ```python
   best_dt = random_search.best_estimator_
   ```

7. **预测和评估**:
   
   ```python
   y_pred = best_dt.predict(X_test)
   ```

**注意事项**:
- 由于 `RandomizedSearchCV` 只尝试指定数量的参数组合，它的运行时间往往比 `GridSearchCV` 短。
- 虽然随机搜索可能会遗漏最佳的参数组合，但在实践中，它往往能找到一个非常接近最佳的参数组合，而且消耗的时间远少于网格搜索。
- 如果你知道某些参数更有可能影响模型的性能，你可以为这些参数指定更细的网格或分布，而为其他参数指定较宽的范围或较少的选项。

## 2. 随机森林模型的参数调优和评估

这里就将一下 `GridSearchCV` 的方法，`RandomizedSearchCV` 可以根据上面决策树的内容举一反三。

以下是具体操作的步骤：

1. **定义参数网格**:
   为所关心的每个参数定义一个值的范围或列表。例如：
   ```python
   param_grid = {
       'n_estimators': [10, 50, 100, 200],
       'max_depth': [None, 10, 20, 30],
       'min_samples_split': [2, 5, 10],
       'min_samples_leaf': [1, 2, 4],
       'bootstrap': [True, False]
   }
   ```

2. **初始化模型**:
   ```python
   from sklearn.ensemble import RandomForestRegressor
   rf = RandomForestRegressor()
   ```

3. **使用交叉验证的网格搜索**:
   使用 `GridSearchCV` 从 `scikit-learn` 来自动进行网格搜索和交叉验证。
   ```python
   from sklearn.model_selection import GridSearchCV
   
   grid_search = GridSearchCV(estimator=rf, param_grid=param_grid, 
                              cv=3, n_jobs=-1, verbose=2)
   ```

   - `cv=3` 指定了3折交叉验证。
   - `n_jobs=-1` 使用所有可用的CPU核心。
   - `verbose=2` 表示过程中显示详细信息。

4. **拟合数据**:
   使用 `grid_search` 对象来拟合你的数据。
   ```python
   grid_search.fit(X_train, y_train)
   ```

5. **查看最佳参数**:
   一旦拟合完成，你可以查看最佳的参数组合。
   ```python
   best_params = grid_search.best_params_
   print(best_params)
   ```

6. **使用最佳模型**:
   使用最佳参数训练的模型可以直接从 `grid_search` 获取。
   ```python
   best_rf = grid_search.best_estimator_
   ```

7. **预测和评估**:
   使用此最佳模型进行预测并评估其性能。
   ```python
   y_pred = best_rf.predict(X_test)
   ```

**注意事项**:
- 网格搜索可能需要很长时间，取决于你的数据大小、参数网格的大小和可用计算资源。因此，从一个较小的参数网格开始可能是明智的，然后根据初步结果进一步调整。
- 如果你想要在搜索的同时考虑更随机的参数选择，可以尝试使用 `RandomizedSearchCV` 而不是 `GridSearchCV`。这允许你为每个参数设置一个分布，并从中随机采样，而不是尝试所有的组合，这通常更快。
- 在真实世界的应用中，使用交叉验证和参数搜索是很有价值的，因为它们可以帮助你确保模型不仅在训练数据上表现良好，而且还能很好地泛化到未见过的数据。

## 总结

参数调优不仅是机器学习实践中的一个重要步骤，而且是确保模型达到最佳性能的关键。通过使用Sklearn提供的工具，我们可以系统地搜索模型的参数空间，找到最佳的参数组合。尽管网格搜索方法可以提供全面的参数组合评估，但在某些情况下，随机搜索可能是一个更高效的选择，尤其是当参数空间非常大时。总之，有效的参数调优需要结合多种策略和工具，同时根据具体的问题和数据进行适当的调整。通过细致的参数调优和评估，我们可以确保我们的模型达到其潜在的最大性能，为我们的任务提供可靠的预测。
