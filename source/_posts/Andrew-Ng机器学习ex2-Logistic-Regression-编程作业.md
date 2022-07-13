---
title: Andrew Ng机器学习ex2(Logistic Regression)编程作业
date: 2022-06-21 13:24:57
categories: 机器学习
tags:
    - 机器学习
    - Machine Learning
    - Coursera
    - 编程作业
    - Octave
    - Logistic Regression
    - Classification
mathjax: true
---

本周的课程是 Logistic Regression，是用来解决分类问题（Classification）的基本算法。作业也是两个分类任务，完整代码在 [这里](https://github.com/Jortana/coursera-machine-learning/tree/main/ex2-octave)。

<!-- more -->

## 1 逻辑回归 Logistic Regression

第一个任务是根据学生的两次成绩，判断他是否会被录取，在这个任务中我们需要完成：

* 数据可视化
* 完成 sigmoid 函数
* 计算 cost function 和 gradient
* 用 fminunc 优化计算
* 预测并评估

### 1.1 数据可视化 Visualizing the data

这一步比较简单，如果对画图函数不是很了解也可以照着作业中给出的答案完成，画图函数的参数是比较简单的，而且基本上可以说一目了然。我们需要在 `plotData.m` 中合适的位置输入：

```Octave
pos = find(y == 1);
neg = find(y == 0);

plot(X(pos, 1), X(pos, 2), 'k+', 'LineWidth', 2, 'MarkerSize', 7);
plot(X(neg, 1), X(neg, 2), 'ko', 'MarkerFaceColor', 'y', 'MarkerSize', 7);
```

### 1.2 完成 sigmoid 函数 Warmup exercise: sigmoid function

这一步我们需要自己实现 sigmoid 函数，并且我们实现的这个函数需要支持输入 **数、向量和矩阵**，对于向量和矩阵，我们需要对其中的每一个元素进行计算。

sigmoid 函数的定义长这样：
$$
g(z)=\frac{1}{1 + e^{-z}}
$$
这一个小任务也是比较简单，注意我们要支持向量和矩阵，所以用两个 for 循环进行遍历即可，在 `sigmoid.m` 中合适的位置输入：

```Octave
for i = 1:size(z, 1),
  for j = 1:size(z, 2),
    g(i, j) = 1 / (1 + exp(-z(i, j)));
  end;
end;
```

注意，在 Octave 中，for 循环后要加 `,`，并且要配合 `end` 使用。

### 1.3 计算代价函数和梯度 Cost function and gradient

因为我们后面需要用到 fminunc，所以这一次计算代价函数的同时也需要计算梯度，以方便后续的使用。

代价函数的计算公式长这样：
$$
J(\theta) = \frac{1}{m}\sum_{i=1}^{m}[-y^{(i)}\log(h_{\theta}(x^{(i)})) - (1-y^{(i)})\log(1-h_{\theta}(x^{(i)}))]
$$
梯度的计算公式长这样：
$$
\frac{\partial J(\theta)}{\partial \theta_{j}} = \frac{1}{m}\sum_{i=1}^{m}(h_{\theta}(x^{(i)})-y^{(i)})x_{j}^{(i)}
$$
在 `costFunction.m` 中合适的位置输入：

```Octave
% 计算 h_theta
h_theta = sigmoid(X * theta);
J = (log(h_theta') * (-y) - log(1 - h_theta') * (1 - y)) / m;

% 计算 grad
grad = (X' * (h_theta - y)) / m;
```

这里有三个地方都用到了 `sigmoid(X * theta)`，所以先进行计算，可以略微提高一点效率，主要是可以让代码变得清晰一些。

### 1.4 用 fminunc 优化计算 Learning parameters using fminunc

在上周的作业中，我们用梯度下降法完成了任务，这周我们需要用到一个更复杂的算法，我们提供代价函数和梯度，算法自动帮我们完成参数的计算。作业提供的代码已经帮我们实现了，可以看一下 `ex2.m` 中的相关代码：

```Octave
%  Set options for fminunc
options = optimset('GradObj', 'on', 'MaxIter', 400);

%  Run fminunc to obtain the optimal theta
%  This function will return theta and the cost 
[theta, cost] = ...
	fminunc(@(t)(costFunction(t, X, y)), initial_theta, options);
```

### 1.5 预测并评估 Evaluating logistic regression

准备工作已经全部做完，我们现在要根据我们得到的模型来进行预测，并看一下预测的准确率如何。（这里的准确率是用实验数据作为数据集计算的，并没有准备额外的测试数据集）

预测的代码也比较简单，只需判断最终得出的结果是否大于 $ 0.5 $ 即可，大于或等于记为 $ 1 $，小于则记为 $ 0 $。这里有很多种实现方式，我自己是对每个数据进行计算之后 **加上 0.5 然后向下取整**。

在 `predict.m` 中合适的位置输入：

```Octave
p = floor(sigmoid(X * theta) + 0.5);
```

## 2 正则化逻辑回归 Regularized logistic regression

在这一部分中，我们需要用到 **正则化逻辑回归 Regularized logistic regression **，现在有一批芯片测试的结果，每一个芯片都进行了两轮的测试，并且标记出了合格与不合格，现在我们要根据这两个测试的结果判断这个芯片是否合格。

在这个任务中我们需要完成：

* 数据可视化
* 特征映射
* 计算 cost function 和 gradient
* 用 fminunc 优化计算
* 可视化决定边界
* 【可选】调整参数

### 2.1 数据可视化

这一步实际上我们在第一个任务的时候已经完成

### 2.2 特征映射

由于我们的数据中的特征非常少，只有两个，是两次测试的结果，我们需要用这两个特征进行高次的运算，来获取更多的特征，这一步在提供的程序中也已经完成，相关代码在 `mapFeature.m` 中。

### 2.3 计算代价函数和梯度 Cost function and gradient

这一步和第一个任务就略有不同了，因为我们需要正则化，所以需要在相关计算公式的基础上加以改动，并且第一项是不需要进行这个操作的。

代价函数的计算公式为：
$$
J(\theta) = \frac{1}{m}\sum_{i=1}^{m}[-y^{(i)}\log(h_{\theta}(x^{(i)})) - (1-y^{(i)})\log(1-h_{\theta}(x^{(i)}))] + \frac{\lambda}{2m}\sum_{j=1}^{n}\theta_{j}^2
$$
梯度的计算公式为：
$$
\frac{\partial J(\theta)}{\partial \theta_{j}} = \frac{1}{m}\sum_{i=1}^{m}(h_{\theta}(x^{(i)})-y^{(i)})x_{j}^{(i)} \quad for \ j=0\\
\frac{\partial J(\theta)}{\partial \theta_{j}} =\left ( \frac{1}{m}\sum_{i=1}^{m}(h_{\theta}(x^{(i)})-y^{(i)})x_{j}^{(i)} \right ) + \frac{\lambda}{m}\theta_{j} \quad for \ j \ge 1
$$
相应地，我们的代码也只要小改即可，在 `costFunctionReg.m` 中合适的位置输入：

```Octave
% 计算 h_theta
h_theta = sigmoid(X * theta);
theta_length = size(theta, 1);
J = ((log(h_theta') * (-y) - log(1 - h_theta') * (1 - y)) / m) + (lambda * theta(2:theta_length)' * theta(2:theta_length) / (2 * m));


% 计算 grad
grad = (X' * (h_theta - y)) / m + lambda * theta / m;
grad(1) = grad(1) - lambda * theta(1) / m;
```

### 2.4 用 fminunc 优化计算

这一步也不需要我们手动输入代码，在第一个任务中我们也已经看到了如何进行计算。

### 2.5 可视化决定边界

这一步也不需要我们输入代码，我们可以运行 `ex2_reg.m` 看看边界的情况，有了这个图在后面一步修改参数的时候可以很明显地看出来参数对于结果的影响。

### 2.6 【可选】调整参数

在运用正则化技术的时候，我们会设定一个 `lambda` 参数， 这个参数的大小决定了正则化影响的程度有多大，作业给的代码中默认设置为 `1`，我们可以调整其大小再观察边界的生成情况，直观地感受这个参数的影响程度。这里需要注意，修改这个参数是要修改 `ex2_reg.m` 文件中 64 行和 110 行的数值，两个都需要修改。

本次作业到此顺利完成。
