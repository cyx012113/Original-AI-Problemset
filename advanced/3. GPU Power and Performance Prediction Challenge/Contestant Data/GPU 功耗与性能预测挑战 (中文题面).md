# GPU 功耗与性能预测挑战

## 1. 背景

大语言模型（LLM）的推理部署是 AI 基础设施中最核心的工程挑战之一。每一次用户请求，GPU 都在消耗电力、产生延迟——而这两者直接决定了数据中心的运营成本和用户体验。

想象你是一家 AI 云计算公司的能效架构师。你的老板甩给你一张表：上面记录了各种 GPU 型号、不同 LLM 模型、在不同请求负载下的实测功耗和延迟数据，然后问你——

> "给定硬件规格、模型参数、以及预期的请求量，我们能不能提前算出来这台机器要花多少电、有多快？"

这就是你要解决的问题。

本数据集来源于学术论文 _Watt Counts: An Empirical Study of GPU Power Consumption and Performance in LLM Inference_，包含了 8 种数据中心级 GPU、30+ 种开源 LLM、在离线批处理和在线服务两种场景下的数千次实测数据。你的任务是基于 **硬件规格 + 模型参数 + 运行负载** 这三个维度的输入，**同时预测四个核心性能指标**。

## 2. 任务定义

**输入（$X$）**：每条实验记录的以下特征——

| 维度     | 特征                     | 说明                                               |
| -------- | ------------------------ | -------------------------------------------------- |
| 运行负载 | `lambda_qps`             | 请求到达率（每秒查询数），$-1$ 表示离线批处理      |
| 运行负载 | `scenario`               | 运行场景：`offline` / `server_low` / `server_high` |
| 运行负载 | `avg_prompt_tokens`      | 平均每条请求的输入 Token 数                        |
| 运行负载 | `avg_generation_tokens`  | 平均每条请求的输出 Token 数                        |
| 运行负载 | `num_requests`           | 该实验包含的请求数量                               |
| 模型参数 | `total_b_params`         | 模型总参数量（十亿）                               |
| 模型参数 | `num_layers`             | Transformer 层数                                   |
| 模型参数 | `hidden_size`            | 隐藏层维度                                         |
| 模型参数 | `num_attention_heads`    | 注意力头数                                         |
| 模型参数 | `num_key_value_heads`    | KV 头数（GQA 相关）                                |
| 模型参数 | `model_type`             | 模型架构家族（如 `llama`、`qwen2` 等）             |
| 硬件规格 | `memory_bandwidth_gb_s`  | GPU 显存带宽（GB/s）                               |
| 硬件规格 | `tflops_16b`             | GPU FP16 算力（TFLOPS）                            |
| 硬件规格 | `thermal_design_power_w` | GPU 热设计功耗（TDP，瓦）                          |
| 硬件规格 | `memory_size_gb`         | GPU 显存容量（GB）                                 |
| 硬件规格 | `release_year`           | GPU 发布年份                                       |
| 硬件规格 | `base_clock_mhz`         | GPU 基础频率（MHz）                                |
| 硬件规格 | `boost_clock_mhz`        | GPU 加速频率（MHz）                                |
| 硬件规格 | `architecture`           | GPU 架构名称                                       |
| 硬件规格 | `memory_type`            | 显存类型（`HBM2e` / `GDDR6` 等）                   |

**输出（$y$）**：四个连续数值标签——

| 标签                                 | 含义                            | 单位      | 业务价值           |
| ------------------------------------ | ------------------------------- | --------- | ------------------ |
| `gpu_power_draw_watts`               | GPU 平均功耗                    | 瓦特（W） | 数据中心电费预估   |
| `avg_e2e_latency_seconds`            | 平均端到端延迟                  | 秒（s）   | 用户体验评估       |
| `energy_efficiency_tokens_per_joule` | 能效比（每焦耳生成 Token 数）   | Token/J   | "绿色 AI" 核心指标 |
| `throughput_tokens_per_second`       | 系统吞吐量（每秒生成 Token 数） | Token/s   | 服务容量评估       |

这是一个**多目标回归**任务。你需要构建一个模型，能够从硬件、模型和负载三个维度出发，同时预测功耗、延迟、能效和吞吐量这四个关键指标。

## 3. 数据集

### 3.1 数据来源

数据来源于 _Watt Counts_ 论文中的 Watt Counts Subset 实验集，在多种 GPU（Tesla T4 到 H200 NVL）上测试了 30+ 种开源 LLM（从 GPT-2 124M 到 Gemma-3 27B）在离线批处理和在线服务场景下的真实推理性能。

### 3.2 文件说明

| 文件                    | 说明                                 |
| ----------------------- | ------------------------------------ |
| `train.csv`             | 训练集，包含特征列 + 4 个标签列      |
| `A.csv`                 | A 榜测试集，**仅包含特征列，无标签** |
| `B.csv`                 | B 榜测试集，**仅包含特征列，无标签** |
| `baseline.ipynb`        | 参赛者基线 Notebook，提供入门参考    |
| `Paper_Watt_Counts.pdf` | _Watt Counts_ 原始论文               |

### 3.3 数据特征

- 训练集、A 榜、B 榜按 70:15:15 随机分割。
- **训练集存在 GPU 不平衡**：高端稀缺显卡（如 H200、H100、A100）在训练集中的样本被有意削减，模拟真实场景中昂贵硬件数据稀缺的情况。常见显卡（T4、L4、L40S）数据充足。
- A 榜和 B 榜测试集保持均衡，所有 GPU 类型均有充分代表。
- 数据跨度极大——从 0.5B 参数的小模型在低端 GPU 上的低功耗场景，到 27B 大模型在 H200 上的高吞吐场景。
- 部分数据存在离线模式（$\text{lambda\_qps} = -1$），表示连续批处理，无请求间隔。

## 4. 评价指标

本任务采用**综合评分**，由以下两个指标加权得到：

### 4.1 主要指标：$R^2$（决定系数）

对于每个目标变量 $i$，计算：

$$ R^2*i = 1 - \frac{\sum*{j} (y*{ij} - \hat{y}*{ij})^2}{\sum*{j} (y*{ij} - \bar{y}\_i)^2} $$

四个目标的 $R^2$ 取平均作为主要得分（范围 0~1）：

$$ \text{Primary Score} = \frac{1}{4} \sum\_{i=1}^{4} R^2_i $$

得分**越高越好**（$1.0$ 为完美预测）。

### 4.2 辅助指标：WMAPE（加权平均绝对百分比误差）

$$ \text{WMAPE}_i = \frac{\sum_{j} |y*{ij} - \hat{y}*{ij}|}{\sum*{j} |y*{ij}|} \times 100\% $$

四个目标的 WMAPE 取平均作为辅助参考（越低越好）。

## 5. 提交格式

你需要提交一个 CSV 文件，命名为 `A_predict.csv`（A 榜）或 `B_predict.csv`（B 榜），格式如下：

```csv
gpu_power_draw_watts,avg_e2e_latency_seconds,energy_efficiency_tokens_per_joule,throughput_tokens_per_second
163.42,0.0487,27.68,4567.2
45.12,0.0125,18.34,892.1
...
```

- **顺序必须与测试集完全一致**（第一行预测对应 `A.csv` / `B.csv` 的第一行）
- **列名必须完全匹配**上述四个标签名
- **不需要包含行索引或 ID 列**

## 6. 约束条件

为了确保比赛的公平性，请遵守以下限制：

| 约束项         | 限制                                                                                       |
| -------------- | ------------------------------------------------------------------------------------------ |
| 预训练模型     | 禁止使用任何预训练权重（包括在非公开数据上训练的）                                         |
| 大语言模型 API | 禁止调用任何 LLM API（如 GPT-4、Claude 等）                                                |
| 第三方库       | 仅限 NOAI 考纲内库（`numpy`、`pandas`、`scikit-learn`、`torch`、`xgboost`、`lightgbm` 等） |
| 外部数据       | 可以使用公开的 GPU / LLM 规格数据，但**严禁使用本数据集的任何外部副本或变体**              |

## 7. 参考资料

1. _Watt Counts: An Empirical Study of GPU Power Consumption and Performance in LLM Inference_ — 本数据集的原始论文
2. _LLM Inference Performance Engineering_ — NVIDIA 技术博客系列
3. _Roofline Model_ — 经典性能分析模型

## 8. 许可

本数据集和题目采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。
