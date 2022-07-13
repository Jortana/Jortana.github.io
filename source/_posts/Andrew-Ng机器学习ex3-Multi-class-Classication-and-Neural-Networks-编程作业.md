---
title: Andrew Ng机器学习ex3(Multi-class Classication and Neural Networks)编程作业
date: 2022-06-28 12:13:07
categories: 机器学习
tags:
    - 机器学习
    - Machine Learning
    - Coursera
    - 编程作业
    - Octave
    - Neural Network
    - Classification
    - One vs All
mathjax: true
---

本周的课程是 Neural Network，这次的作业是一个较为简单的多类别分类算法，完整代码在 [这里](https://github.com/Jortana/coursera-machine-learning/tree/main/ex3-octave)。

<!-- more -->

## 1 多类别分类 Multi-class Classification

本次作业的任务就是实现一个分类的程序，用于识别手写的数字。做完之后对于神经网络的分类还是有一个直观和感性的理解的。

### 1.1 数据集 Dataset

这次作业帮我们把数据准备好并且导入了，也帮我们把数据的呈现做好了。数据一共有 5000 条，每一条数据都代表一个 $20 \times 20$ 的图像。也就是说每一条数据包含 400 个 feature，每一个 feature 都代表每一个像素的灰度数据。

数据的呈现其实就是随机选 100 个手写数据，并呈现在一个矩阵里。

### 1.2 逻辑回归的矢量化计算 Vectorizing Logistic Regression

看上去是新的内容，实际上我们前几次作业都是用的矢量化计算，并没有用到循环去计算代价函数以及梯度等，不过这次的作业里详细且直观地解释了矢量化计算的数学原理，我们甚至可以直接把上周的作业复制粘贴过来，效果是完全一样的，在 `lrCostFunction.m` 中合适的位置输入：

```Octave
% 计算 h_theta
h_theta = sigmoid(X * theta);
J = (log(h_theta') * (-y) - log(1 - h_theta') * (1 - y)) / m;

% 计算 grad
grad = (X' * (h_theta - y)) / m;
```

### 1.3 加上正则 Vectorizing regularized logistic regression

上面一步我们并没有进行 add regularized 的操作，也是和上次的作业一样，直接复制过来就可以了，同样在  `lrCostFunction.m` 中进行修改，把上面那一段修改为：

```Octave
% 计算 h_theta
h_theta = sigmoid(X * theta);
theta_length = size(theta, 1);
J = ((log(h_theta') * (-y) - log(1 - h_theta') * (1 - y)) / m) + (lambda * theta(2:theta_length)' * theta(2:theta_length) / (2 * m));


% 计算 grad
grad = (X' * (h_theta - y)) / m + lambda * theta / m;
grad(1) = grad(1) - lambda * theta(1) / m;
```

### 1.4 One-vs-all 策略 One-vs-all Classification

因为我们的任务是要识别手写数字，相当于一个有 10 个类别的分类任务，因此我们要用到 One-vs-all 的策略。作业里建议我们使用 `fmincg` 代替上次作业用的 `fminunc`， `fmincg` 已经帮我们准备好了，直接用就可以，用法和 `fminunc` 一样，并且在作业的代码中也给了用法示例，我们在 `oneVsAll.m` 中合适的位置输入：

```Octave
options = optimset('GradObj', 'on', 'MaxIter', 50);
for c = 1:num_labels,
  [theta] = fmincg (@(t)(lrCostFunction(t, X, (y == c), lambda)), all_theta(c, :)', options);
  all_theta(c, :) = theta';
end;
```

这里我们用到了 `for` 循环，是因为我们要计算 `num_labels` 个类别，这个循环是必要的。

#### 1.4.1 One-vs-all 预测 One-vs-all Prediction

做完了前面的准备工作后，我们要开始预测了，在我们进行计算之后，每一行都有 10 个元素，代表当前的图片对应每个数字的可能性，我们找到其中最大的值，它的 index 就是最有可能的数字**（为了方便表示，index 为 1 - 9 时代表数字 1 - 9，index 10 代表数字 0）**。

在 `predictOneVsAll.m` 中合适的位置输入：

```Octave
[value, p] = max(X * all_theta', [], 2);
```

这里可以查询一下 `max()` 函数的使用以理解这一段代码的意思。

## 2 神经网络 Neural Networks

之前我们实现了逻辑回归的预测，最终我们可以看到正确率在 $ 94.9\% $ 左右，接下来我们用神经网络实现一下看看正确率有多少（都是实验数据集，没有测试数据集）。

### 2.1 模型表示 Model representation

本次作业已经帮我们准备好了神经网络的参数，这次用到的是 3 层的神经网络，包括 1 个输入层，1 个隐藏层和 1 个输出层。

### 2.2 前馈传播和预测 Feedforward Propagation and Prediction

因为 theta 已经帮我们准备好了，我们只需要直接按步骤计算就行了，在 `predict.m` 中合适的位置输入：

```Octave
% 添加 a1 的 bias
X = [ones(m, 1), X];
% 计算 a2
a2 = sigmoid(X * Theta1');
% 添加 a2 的 bias
a2 = [ones(size(a2, 1), 1), a2];
% 计算a3
a3 = sigmoid(a2 * Theta2');
% 计算 p
[value, p] = max(a3, [], 2);
```

这时候运行 `ex3_nn.m` 可以看到神经网络预测的准确率在 $ 97.5\% $ 左右，比逻辑预测要高出大约 $ 3.5\% $ 左右，这个程序最后也帮我们做了可视化的结果呈现，可以把手写图片和预测结果进行对比。

本次作业到此顺利完成。
