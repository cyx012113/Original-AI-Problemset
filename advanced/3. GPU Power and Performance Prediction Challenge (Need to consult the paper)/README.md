# ⚡ GPU功耗预测：当 AI 开始操心电费账单 🔋

> **"给你一张 GPU 的体检报告和一份模型的简历，你能算出这台机器会烧多少电、跑多快吗？"**
> —— 欢迎来到 **GPU 功耗与性能预测挑战**，让你足不出户，体验一把 **数据中心能效架构师** 的日常。

<div align="center">

<svg width="500" height="130" xmlns="http://www.w3.org/2000/svg">
    <rect width="500" height="130" fill="#f8f9fa" rx="10"/>
    <!-- 左侧：GPU + 模型 -->
    <rect x="15" y="35" width="50" height="55" rx="4" fill="#34495e"/>
    <text x="40" y="70" font-size="14" fill="#ecf0f1" text-anchor="middle">GPU</text>
    <rect x="75" y="35" width="50" height="55" rx="4" fill="#8e44ad"/>
    <text x="100" y="70" font-size="12" fill="#ecf0f1" text-anchor="middle">LLM</text>
    <text x="155" y="68" font-size="22" fill="#7f8c8d">+</text>
    <!-- 请求箭头 -->
    <path d="M180,50 L200,50" stroke="#e74c3c" stroke-width="2"/>
    <text x="175" y="40" font-size="10" fill="#e74c3c">请求</text>
    <!-- 中间：问号 -->
    <text x="220" y="72" font-size="36" fill="#f39c12">❓</text>
    <text x="255" y="68" font-size="22" fill="#7f8c8d">→</text>
    <!-- 右侧：四个预测目标 -->
    <text x="290" y="52" font-size="13" fill="#2c3e50">⚡ 功耗(W)</text>
    <text x="290" y="72" font-size="13" fill="#2c3e50">⏱️ 延迟(s)</text>
    <text x="400" y="52" font-size="13" fill="#2c3e50">🌱 能效(tok/J)</text>
    <text x="400" y="72" font-size="13" fill="#2c3e50">🚀 吞吐(tok/s)</text>
    <!-- 底部幽默文字 -->
    <text x="140" y="118" font-size="13" fill="#95a5a6">"24个特征 → 4个数字，比老板让你预估季度KPI简单多了。"</text>
</svg>

_"从硬件参数到电费账单——AI 能效分析师正式上岗。"_

</div>

---

## 🔌 题目简介

你是一家 AI 云计算公司的能效架构师。老板甩给你一堆实验记录——记录了各种 GPU 型号跑各种 LLM 模型在不同请求负载下的实际表现。现在他要你：

> **不看实测结果，只看"配置单"，就能预测功耗、延迟、能效和吞吐量。**

- **输入**：24+ 个特征，涵盖运行负载（QPS / Token 数）、模型参数（参数量 / 层数 / 隐藏维度）、硬件规格（带宽 / 算力 / TDP）
- **输出**：**4 个连续数值**——GPU 功耗、端到端延迟、能效比、吞吐量
- **数据来源**：_Watt Counts_ 论文的实证数据集，8 种 GPU × 30+ 种 LLM × 离线/在线两种场景
- **训练集**：数千条实测记录，特征 + 4 个标签全给你
- **A 榜测试集**：只给特征，让你悄悄验证模型
- **B 榜测试集**：只给特征，最终评分全靠它

> 😅 **温馨提示**：不要试图手工推导——H200 跑 7B 模型的功耗和 T4 跑 1.5B 模型的延迟，这两者之间的物理规律比高数课本还复杂。把这事交给机器学习吧。

<p style="font-size: 8px;"><del>如果你觉得"不就是个回归吗"，恭喜你——你已经具备了轻敌的所有条件。训练集中高端显卡（H200、H100）的数据故意砍掉大半——毕竟现实中谁没事拿 H200 做实验啊？不过别担心，A/B 榜是均衡的。</del></p>

---

## 📊 评分标准：R² + WMAPE —— 四项全能才拿金牌

这是一道"偏科禁止"的题。你需要同时预测四个目标，缺一则败。

### 主指标：R²（决定系数）

$$
R^2_i = 1 - \frac{\sum_j (y_{ij} - \hat{y}_{ij})^2}{\sum_j (y_{ij} - \bar{y}_i)^2}
$$

四个目标的 R² 取**平均**作为主得分。**越高越好**，范围 0~1，1.0 = 完美（别想了）。

### 辅助指标：WMAPE（加权平均绝对百分比误差）

$$
\text{WMAPE}_i = \frac{\sum_{j} |y_{ij} - \hat{y}_{ij}|}{\sum_{j} |y_{ij}|} \times 100\%
$$

四个目标的 WMAPE 取平均作为辅助参考。**越低越好**，0% = 完美。

> 🎯 **目标**：让你的模型成为"GPU 算命师"——掐指一算，功耗延迟全知道。偏科某一项？不好意思，总分教你做人。

---

## 🚫 约束条件（"违反任意一条，罚款 100000000 度电"）

1. ❌ **不准用预训练权重**（ImageNet / 任何其他数据集上训过的模型都不行）。必须从零开始训练。
2. ❌ **不准调戏大模型 API**（GPT、Claude、文心一言……它们的物理直觉可能还不如你）。
3. ❌ **只允许 NOAI 考纲内的库**（`numpy`、`pandas`、`scikit-learn`、`torch`、`xgboost`、`lightgbm` 等，别掏出来路不明的第三方轮子）。
4. ❌ **严禁使用本数据集的任何外部副本或变体**。

> 😈 违反任何一条，你的模型将被罚去手动给 100000000 条实验数据抄电表。

---

## 📁 数据格式（CSV，看一眼就懂）

### `train.csv` 列说明

| 维度    | 列名                                 | 类型  | 含义                                           |
| :------ | :----------------------------------- | :---- | :--------------------------------------------- |
| 🏃 负载 | `lambda_qps`                         | float | 请求到达率，-1 表示离线批处理                  |
| 🏃 负载 | `scenario`                           | str   | 场景：`offline` / `server_low` / `server_high` |
| 🏃 负载 | `avg_prompt_tokens`                  | float | 平均每条请求的输入 Token 数                    |
| 🏃 负载 | `avg_generation_tokens`              | float | 平均每条请求的输出 Token 数                    |
| 🧠 模型 | `total_b_params`                     | float | 总参数量（十亿）                               |
| 🧠 模型 | `num_layers`                         | int   | Transformer 层数                               |
| 🧠 模型 | `hidden_size`                        | int   | 隐藏层维度                                     |
| 🧠 模型 | `model_type`                         | str   | 架构家族（llama / qwen2 / mistral 等）         |
| 💻 硬件 | `memory_size_gb`                     | float | GPU 显存容量 (GB)                              |
| 🎯 标签 | `gpu_power_draw_watts`               | float | **预测目标1**：平均功耗 (W)                    |
| 🎯 标签 | `avg_e2e_latency_seconds`            | float | **预测目标2**：端到端延迟 (s)                  |
| 🎯 标签 | `energy_efficiency_tokens_per_joule` | float | **预测目标3**：能效比 (Token/J)                |
| 🎯 标签 | `throughput_tokens_per_second`       | float | **预测目标4**：吞吐量 (Token/s)                |

### `A.csv` 和 `B.csv`

跟上面一样，**唯独少了最后 4 列标签**。因为那是你要预测的。

> 📌 **注意**：训练集、A 榜、B 榜按 70:15:15 随机分割。**训练集含 GPU 不平衡**——稀缺的高端显卡（H200、H100等）样本较少，模拟真实场景中"买不起的卡数据也不多"。A/B 榜则完全均衡。

| GPU                  |    训练集保留 | 理由                       |
| :------------------- | ------------: | :------------------------- |
| Tesla T4 / L4 / L40S |      **100%** | 遍地都是，随便跑           |
| RTX 3090 / 4090      | **70% / 60%** | 消费卡，顺便测测           |
| Tesla V100           |       **50%** | 老前辈了，且用且珍惜       |
| A100 / A30           |       **40%** | 贵，测一次心疼一次         |
| H100 系列            |       **25%** | 数据中心新贵，排队都排不上 |
| H200 NVL             |       **20%** | 顶级货，电费单比房租贵     |

> 😅 毕竟现实中谁能拿一打 H200 随便做实验啊？训练时吃不饱，考试还得全考——这才是真实世界的残酷。

---

## 🚀 提交格式（列名写错直接 0 分）

你需要提交两个预测文件：

- `A_predict.csv`
- `B_predict.csv`

每个文件包含 4 列，**顺序必须与测试集完全一致**：

```csv
gpu_power_draw_watts,avg_e2e_latency_seconds,energy_efficiency_tokens_per_joule,throughput_tokens_per_second
163.42,0.0487,27.68,4567.2
45.12,0.0125,18.34,892.1
```

> ⚠️ **不要加行索引，不要加 ID 列。** 评测脚本会按行号逐行比对，多一列 → 直接 0 分（冷酷但合理）。

---

## 🧠 Baseline 思路（超越它不算本事）

官方提供了一个 **Ridge 回归的极简基线**，预期 R² 在 0.6~0.8 左右。
说实话，线性模型能有什么出息？你应该轻松吊打它。

---

## 📚 参考资料（助你成为 GPU 算命大师）

- **Watt Counts 论文**：_Watt Counts: An Empirical Study of GPU Power Consumption and Performance in LLM Inference_。（PDF 已打包进赛题压缩包，不用翻墙）
- **NVIDIA LLM Inference Performance Engineering**：官方技术博客系列，讲透了 GPU 推理的性能瓶颈。
- **Roofline Model**：经典性能分析框架，告诉你程序是卡在带宽还是算力上。
- **数据处理脚本**：赛题压缩包 `dataset/` 目录下的 `dataset-process.py` 和 `dataset-split.py`，展示了完整的特征工程和数据处理流程。

---

## 🎉 最后的话

你手中的数据来自 **真实的 GPU 集群**——从 Tesla T4 到 H200 NVL，从 GPT-2 124M 到 Gemma-3 27B，每一行都是真金白银的电费和机时烧出来的。

数据中心一年耗电量相当于一个小国家。如果你的模型能让功耗预测误差降低 5%，省下的电费够你买一打 H100。

但今天，你只需要让它学会：**看一份配置单，猜四个数字**。

Go, train, and may the R² be high! ⚡🔋

## 数据来源与许可

本题所使用的数据集基于 _Watt Counts_ 论文（Watt Counts Subset）进行筛选与处理。我们对原始数据进行了特征工程处理，包括列表列解析聚合、GPU/模型元数据合并，以及目标变量计算。

<div align="center"> &copy cyx012113 2026. </div>
