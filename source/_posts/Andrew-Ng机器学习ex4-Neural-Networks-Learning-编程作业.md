---
title: Andrew Ng机器学习ex4(Neural Networks Learning)编程作业
date: 2022-07-05 11:20:52
tags:
    - 机器学习
    - Machine Learning
    - Coursera
    - 编程作业
    - Octave
    - Neural Network
    - Classification
    - Backpropagation
mathjax: true 
---

这周的作业是在上周的基础上进行了难度的提升，需要我们自己计算参数并加上反向传播（Backpropagation），完整代码在 [这里](https://github.com/Jortana/coursera-machine-learning/tree/main/ex4-octave)。

<!-- more -->

## 1 神经网络 Neural Networks

在进行神经网络搭建之前，我们可以先进行数据可视化的工作，这一步作业已经做好了。然后我们要构思一下我们的神经网络是一个什么结构的，这里作业给了我们一个三层的模型，包含一个输入层、一个隐藏层和一个输出层，输入层一共 401 个节点，对应每个图像的 400 个像素和一个 bias，隐藏层为包含一个 bias 的 26 个节点，输出层为 10 个节点，对应 10 个数字，这里也是将 数字 0 映射到了 10，方便 Octave 的下标表示。

### 1.1 前向传播和代价函数 Feedforward and cost function

这一步可以参考上周的作业，只是公式中增加了了一个求和的操作，并且我们需要在这一步直接进行 One-vs-all 的操作。也就是说，我们在计算每一条数据的时候，需要转换一下 `y` 的形式，这点在上周和这周的课程和作业中都讲的比较详细了。

需要注意的是，虽然这里我们看上去是要对每一条数据单独进行操作，但实际上我们只需要对 `y` 这个变量进行一次遍历，然后直接用矩阵进行计算即可。

在  `nnCostFunction.m` 中合适的位置输入：

```Octave
% 添加 a1 的 bias
X = [ones(m, 1), X];
% 计算 a2
a2 = sigmoid(X * Theta1');
% 添加 a2 的 bias
a2 = [ones(m, 1), a2];
% 计算a3，a3 就是 h_theta
a3 = sigmoid(a2 * Theta2');

% 修改 y 的值
yk = zeros(m, num_labels); 
for i = 1:m,
  yk(i, y(i)) = 1;
end

J = sum(sum(-yk .* log(a3) - (1 - yk) .* log(1 - a3))) / m;
% 上面这一行也可以写成
% J = sum(sum(-yk .* log(a3)) - sum((1 - yk) .* log(1 - a3))) / m;
```

### 1.2 正则化代价函数  Regularized cost function

我们在上一步进行计算的时候并没有进行正则化，根据作业中给出的公式可以很容易地写出正则化的代码，实际上只需要在刚刚的最后一行上再增加一步 **加** 的操作即可。

将刚刚的最后一行修改为：

```Octave
J = sum(sum(-yk .* log(a3) - (1 - yk) .* log(1 - a3))) / m + ...
  lambda * (sum(sum(Theta1(:, 2:end) .^ 2)) + sum(sum(Theta2(:, 2:end) .^ 2))) / (2 * m);
```

这里要注意 bias 的项是不需要进行正则化的。

## 2 反向传播 Backpropagation

这一部分也是这次作业比较重要的部分，反向传播是这周上课的重点。

### 2.1 S 函数求导 Sigmoid gradient

我们需要做一点准备工作，首先就是计算 Sigmoid 函数的导数，计算公式在作业中也已经给出，我们只需要按照公式来即可，在 `sigmoidGradient.m` 中合适的地方输入：

```Octave
gz = 1.0 ./ (1.0 + exp(-z));
g = gz .* (1 - gz);
```

这里我们依旧是需要支持输入数字和矩阵的，所以使用了 `.` 进行运算。

### 2.2 随机初始化参数 Random initialization

这一步在课上也讲过，如果不随机初始参数，那么后面的节点总是会得到相同的权重（这里我表达地不是非常贴切，可以看看原视频和笔记）。这一步地代码也给了我们，也是非常简单地代码，在 `randInitializeWeights.m` 中合适的位置输入：

```Octave
epsilon_init = 0.12;
W = rand(L_out, 1 + L_in) * 2 * epsilon_init - epsilon_init;
```

### 2.3 反向传播 Backpropagation

反向传播需要我们对每一条数据都进行 5 步操作：

1. 根据 $$ a^{(1)} $$ 正向计算 $$ z^{(2)}, a^{(2)}, z^{(3)}, a^{(3)} $$；
2. 对第 3 层（输出层）计算 $$ \delta^{(3)} $$；
3. 对第 2 层（隐藏层）计算 $$ \delta^{(2)} $$；
4. 计算 $$ \Delta $$（这里的 $$ \Delta $$ 是为了计算 `Theta1` 和 `Theta2` 的，所以我们在代码中也可以直接在这一步计算 `Theta1` 和 `Theta2` ，循环结束之后再对这两个变量进一步处理，这样会更高效也更方便）
5. 计算 `Theta1` 和 `Theta2`

其中前 4 步要在循环中完成，第 5 步在循环结束后完成（实际上在循环中也完成了第 5 步的一半）。

在 `nnCostFunction.m` 中前面完成的部分之后输入：

```Octave
% Backpropagation
for t = 1:m,
  % Step 1
  a1 = X(t, :)';
  z2 = Theta1 * a1;
  a2 = [1; sigmoid(z2)];
  z3 = Theta2 * a2;
  a3 = sigmoid(z3);

  % Step 2
  delta3 = (a3 - yk(t, :)');

  % Step 3
  delta2 = Theta2(:, 2:end)' * delta3 .* sigmoidGradient(z2);

  % Step 4
  Theta1_grad = Theta1_grad + delta2 * a1';
  Theta2_grad = Theta2_grad + delta3 * a2';
end

% Step 5
Theta1(:, 1) = 0;
Theta2(:, 1) = 0;
Theta1_grad = Theta1_grad ./ m
Theta2_grad = Theta2_grad ./ m
```

### 2.4 梯度检验 Gradient checking

这里作业中是搭建了一个小型的神经网络，用的数据量很小，这样也不会算很久，这是一种很方便的策略，因为梯度检验是比较耗费时间和性能的（是真的一个一个再算）。代码也在作业中给出了，并不难，如果前面的程序没有问题，可以看到我们得到的误差是远小于他给出的 1e-9 的误差值的，我自己做出来结果是在 1e-11 这个数量级水平的。

### 2.5 正则化神经网络

同样地，我们在计算 `Theta1` 和 `Theta2` 时，也需要对它们进行正则化，只需要修改上面我们完成的代码的最后两行，修改为：

```Octave
Theta1_grad = Theta1_grad ./ m + lambda * Theta1 / m;
Theta2_grad = Theta2_grad ./ m + lambda * Theta2 / m;
```

### 2.6 Learning parameters using fmincg

进行到上一步之后，作业提供的代码就会帮我们使用 `fmincg` 进行计算，本次作业到此顺利完成。

作业的后面还有一些也比较重要的工作，不过不涉及代码，大家可以自行调节一下各个参数，根据结果可以对每个参数有一个感性的认识。
