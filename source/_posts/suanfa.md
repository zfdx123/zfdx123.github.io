---
layout: post
title: 算法
date: 2025-11-13 22:10:19
tags: 
    - 编程
    - C#
categories: 
    - 编程
    - C#
thumbnail: "images/sql_csharp/fm.PNG"
cover: "images/sql_csharp/fm.PNG"
excerpt: "public virtual async Task<List<T>> Select<T>(string query, SqlParameter[] sqlParameters) where T : new()"
license: cc_by_nc_sa
---

# 🧠 一、数据定义与符号说明

| 符号                                         | 含义                                   |
| ------------------------------------------ | ------------------------------------ |
| ( X \in \mathbb{R}^{N \times T \times F} ) | EEG输入序列，包含 (N) 条样本，每条长度 (T)，特征维度 (F) |
| ( Y \in \mathbb{R}^{N \times H} )          | 目标输出序列，每条样本预测未来 (H) 步（attention值等）   |
| ( \hat{Y} \in \mathbb{R}^{N \times H} )    | 模型预测结果                               |
| ( t )                                      | 时间步                                  |
| ( f )                                      | 特征维度                                 |
| ( h_t )                                    | LSTM隐藏状态                             |
| ( a_t )                                    | Attention权重                          |
| ( \tilde{h} )                              | 注意力加权后的上下文向量                         |

---

# ⚙️ 二、模型结构数学描述

模型的主体由以下几个部分组成：

---

## (1) 输入层

EEG序列输入：
[
X = [x_1, x_2, ..., x_T], \quad x_t \in \mathbb{R}^F
]

---

## (2) LSTM 层（时间依赖建模）

LSTM单元更新公式如下：

[
\begin{aligned}
i_t &= \sigma(W_i x_t + U_i h_{t-1} + b_i) \
f_t &= \sigma(W_f x_t + U_f h_{t-1} + b_f) \
o_t &= \sigma(W_o x_t + U_o h_{t-1} + b_o) \
\tilde{c}*t &= \tanh(W_c x_t + U_c h*{t-1} + b_c) \
c_t &= f_t \odot c_{t-1} + i_t \odot \tilde{c}_t \
h_t &= o_t \odot \tanh(c_t)
\end{aligned}
]

输出为隐藏状态序列：
[
H = [h_1, h_2, ..., h_T]
]

---

## (3) 注意力机制（Attention Layer）

模型通过 Attention 聚焦于关键时间步：

计算注意力权重：
[
e_t = v^\top \tanh(W_h h_t + b_h)
]

Softmax归一化：
[
a_t = \frac{\exp(e_t)}{\sum_{k=1}^{T} \exp(e_k)}
]

上下文向量：
[
\tilde{h} = \sum_{t=1}^{T} a_t \cdot h_t
]

这一步帮助模型在 EEG 时序信号中 **自动聚焦于与预测相关的时间窗口**（例如注意力或情绪波动）。

---

## (4) 全连接层（预测层）

将上下文向量映射到未来预测：
[
\hat{Y} = W_y \tilde{h} + b_y
]
输出维度 ( H = n_{\text{output_timesteps}} )。

---

# 🎯 三、训练目标与损失函数

损失函数通常为 **加权组合的回归误差**：

[
\mathcal{L} = \alpha \cdot \frac{1}{N H} \sum_{i=1}^N \sum_{t=1}^H (\hat{y}*{i,t} - y*{i,t})^2

* (1 - \alpha) \cdot \frac{1}{N H} \sum_{i=1}^N \sum_{t=1}^H |\hat{y}*{i,t} - y*{i,t}|
  ]

即：
[
\mathcal{L} = \alpha \cdot \text{MSE} + (1-\alpha)\cdot \text{MAE}
]

在你代码中使用的是：

```python
loss = mean_absolute_error
metrics = [mean_absolute_error, mean_squared_error]
```

所以主要是 **MAE（平均绝对误差）**：
[
\mathcal{L}*{MAE} = \frac{1}{N H}\sum*{i=1}^N \sum_{t=1}^H |\hat{y}*{i,t} - y*{i,t}|
]

---

# 🚀 四、优化算法（Adam 自适应优化器）

Adam 结合了动量法与RMSProp：

[
\begin{aligned}
m_t &= \beta_1 m_{t-1} + (1-\beta_1) \nabla_\theta \mathcal{L}*t \
v_t &= \beta_2 v*{t-1} + (1-\beta_2) (\nabla_\theta \mathcal{L}_t)^2 \
\hat{m}_t &= \frac{m_t}{1-\beta_1^t} \
\hat{v}*t &= \frac{v_t}{1-\beta_2^t} \
\theta_t &= \theta*{t-1} - \eta \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}
\end{aligned}
]

其中：

* (\eta)：学习率；
* (\beta_1, \beta_2)：动量参数（默认 0.9 和 0.999）。

---

# 📉 五、自适应学习率（ReduceLROnPlateau）

当验证集 loss 连续 (k) 轮未改善时，自动降低学习率：

[
\eta_{new} =
\begin{cases}
\eta_{old} \times \gamma, & \text{if } val_loss \text{ 未改善} \
\eta_{old}, & \text{otherwise}
\end{cases}
]

例如：
[
\gamma = 0.5, \quad k = 3
]
表示如果连续 3 轮 val_loss 没降低，学习率减半。

---

# 🧩 六、早停策略（EarlyStopping）

当验证集 loss 连续 (p) 轮未改善时提前停止训练：

[
\text{if } val_loss_{t} > \min(val_loss_{t-p:t-1}) \text{ for } p \text{ epochs, then stop.}
]

并自动恢复到 **最佳 epoch 的权重**。

---

# 🔮 七、预测阶段数学表达式

预测阶段输入最后一段 EEG 序列 ( X_{pred} )：

[
\hat{Y}*{future} = f*\theta(X_{pred})
]

其中 ( f_\theta ) 是训练好的神经网络。

未来时间点序列由采样率与预测步数定义：
[
t_{future,i} = t_{last} + i \cdot \frac{prediction_duration}{H}, \quad i = 1, 2, ..., H
]

最终绘制：
[
\text{Trend}(t) = { (t_{past}, attention_{past}), (t_{future}, \hat{Y}_{future}) }
]

---

# 🔥 八、误差分布热图（Error Heatmap）

预测误差矩阵：
[
E = |\hat{Y} - Y|
]

归一化后绘制为二维热图：

[
E_{norm} = \frac{E - \min(E)}{\max(E) - \min(E)}
]

可视化为时间步 × 输出步的误差强度分布图。

---

# 📊 九、Attention 趋势分析图

从 Attention 权重序列 (a_t) 可视化模型关注的时间区间：

[
\text{Attention Trend} = { (t, a_t) \mid t = 1, 2, \ldots, T }
]

并叠加预测趋势：
[
\text{Combined Trend} = [a_t]*{past} + [\hat{Y}*{future}]
]

---

# ✅ 十、总结表

| 模块        | 主要公式                                            | 说明                |
| --------- | ----------------------------------------------- | ----------------- |
| 输入序列      | ( X \in \mathbb{R}^{T\times F} )                | EEG 多通道输入         |
| LSTM      | ( h_t = f(x_t, h_{t-1}) )                       | 时间序列依赖            |
| Attention | ( a_t = \text{softmax}(v^\top \tanh(W_h h_t)) ) | 聚焦关键时间点           |
| 输出层       | ( \hat{Y} = W_y \sum_t a_t h_t + b_y )          | 未来值预测             |
| 损失函数      | ( \mathcal{L} = \text{MAE} )                    | 平均绝对误差            |
| 优化器       | Adam                                            | 自适应梯度下降           |
| 学习率调节     | ( \eta \rightarrow \eta \times \gamma )         | ReduceLROnPlateau |
| 早停策略      | stop if ( val_loss ) 未改善                        | EarlyStopping     |
| 预测趋势      | ( t_{future} = t_{last} + i \Delta t )          | 实时趋势预测            |