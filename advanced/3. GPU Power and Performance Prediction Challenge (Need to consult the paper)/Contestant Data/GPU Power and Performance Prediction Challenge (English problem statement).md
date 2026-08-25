# GPU Power and Performance Prediction Challenge

## Background

LLM inference deployment is one of the most critical engineering challenges in AI infrastructure. Every user request consumes GPU power and introduces latency — two factors that directly determine data center operating costs and user experience.

Imagine you are an energy efficiency architect at an AI cloud computing company. Your boss hands you a spreadsheet recording measured power draw and latency across various GPU models, LLM architectures, and request loads, then asks:

> "Given hardware specs, model parameters, and expected traffic — can we predict how much power this machine will consume and how fast it will be?"

That is the problem you are here to solve.

This dataset originates from the academic paper _Watt Counts: An Empirical Study of GPU Power Consumption and Performance in LLM Inference_. It contains thousands of real measurements across 8 data-center-class GPUs, 30+ open-source LLMs, in both offline batch and online serving scenarios. Your task is to **simultaneously predict four core performance metrics** from three input dimensions: **hardware specs + model parameters + workload**.

## Task Definition

**Input (X)** — the following features for each experimental run:

| Dimension | Feature                 | Description                                            |
| --------- | ----------------------- | ------------------------------------------------------ |
| Workload  | `lambda_qps`            | Request arrival rate (queries/sec), -1 = offline batch |
| Workload  | `scenario`              | Operating scenario: offline / server_low / server_high |
| Workload  | `avg_prompt_tokens`     | Average input tokens per request                       |
| Workload  | `avg_generation_tokens` | Average output tokens per request                      |
| Workload  | `num_requests`          | Number of requests in the experiment                   |
| Model     | `total_b_params`        | Total model parameters (billions)                      |
| Model     | `num_layers`            | Number of transformer layers                           |
| Model     | `hidden_size`           | Hidden layer dimension                                 |
| Model     | `num_attention_heads`   | Number of attention heads                              |
| Model     | `num_key_value_heads`   | Number of KV heads (GQA)                               |
| Model     | `model_type`            | Model architecture family (e.g., llama, qwen2)         |
| Hardware  | `memory_size_gb`        | GPU memory capacity (GB)                               |
| Hardware  | `release_year`          | GPU release year                                       |
| Hardware  | `base_clock_mhz`        | GPU base clock (MHz)                                   |
| Hardware  | `boost_clock_mhz`       | GPU boost clock (MHz)                                  |
| Hardware  | `architecture`          | GPU architecture name                                  |
| Hardware  | `memory_type`           | Memory type (HBM2e/GDDR6/etc.)                         |

**Output (y)** — four continuous target variables:

| Target                               | Meaning                          | Unit        | Business Value              |
| ------------------------------------ | -------------------------------- | ----------- | --------------------------- |
| `gpu_power_draw_watts`               | Average GPU power draw           | Watts (W)   | Electricity cost estimation |
| `avg_e2e_latency_seconds`            | Average end-to-end latency       | Seconds (s) | User experience             |
| `energy_efficiency_tokens_per_joule` | Energy efficiency (tokens/joule) | Token/J     | "Green AI" metric           |
| `throughput_tokens_per_second`       | System throughput (tokens/sec)   | Token/s     | Service capacity            |

This is a **multi-target regression** task. Build a model that predicts all four metrics simultaneously from hardware, model, and workload features.

## Dataset

### Data Source

The data comes from the Watt Counts Subset in the _Watt Counts_ paper, measuring real inference performance of 30+ open-source LLMs (from GPT-2 124M to Gemma-3 27B) across 8 GPUs (from Tesla T4 to H200 NVL) in both offline batch and online serving scenarios.

### File Description

| File                    | Description                                           |
| ----------------------- | ----------------------------------------------------- |
| `train.csv`             | Training set with feature columns + 4 target columns  |
| `A.csv`                 | A-leaderboard test set, **features only, NO targets** |
| `B.csv`                 | B-leaderboard test set, **features only, NO targets** |
| `baseline.ipynb`        | Contestant baseline notebook for getting started      |
| `Paper_Watt_Counts.pdf` | The _Watt Counts_ original paper                      |

### Data Characteristics

- Train, A-test, and B-test sets are randomly split at a 70:15:15 ratio.
- **Training set has GPU imbalance**: rare high-end GPUs (H200, H100, A100) are intentionally downsampled, simulating real-world data scarcity for expensive hardware. Common GPUs (T4, L4, L40S) have full representation.
- A and B test sets remain fully balanced across all GPU types.
- Values span extreme ranges — from a 0.5B model on a low-end GPU to a 27B model on an H200 at full throttle.
- Offline mode entries have `lambda_qps = -1`, representing continuous batch processing without request gaps.

## Evaluation Metrics

This task uses a **composite score** combining the following metrics:

### Primary Metric: $R^2$ (Coefficient of Determination)

For each target variable $i$:

$$ R^2*i = 1 - \frac{\sum*{j} (y*{ij} - \hat{y}*{ij})^2}{\sum*{j} (y*{ij} - \bar{y}\_i)^2} $$

The average $R^2$ across all four targets forms the primary score (range $0 \sim 1$):

$$ \text{Primary Score} = \frac{1}{4} \sum\_{i=1}^{4} R^2_i $$

**Higher is better** (1.0 = perfect prediction).

### Auxiliary Metric: WMAPE (Weighted Mean Absolute Percentage Error)

$$ \text{WMAPE}_i = \frac{\sum_{j} |y*{ij} - \hat{y}*{ij}|}{\sum*{j} |y*{ij}|} \times 100\% $$

The average WMAPE across all four targets serves as auxiliary reference (lower is better).

## Submission Format

Submit a CSV file named `A_predict.csv` (for A-leaderboard) or `B_predict.csv` (for B-leaderboard) with the following format:

```csv
gpu_power_draw_watts,avg_e2e_latency_seconds,energy_efficiency_tokens_per_joule,throughput_tokens_per_second
163.42,0.0487,27.68,4567.2
45.12,0.0125,18.34,892.1
...
```

- **Order must match the test set exactly** (first prediction row = first test set row)
- **Column names must match** the four target names above exactly
- **Do NOT include row indices or ID columns**

## Constraints

To ensure fair competition:

| Constraint         | Limit                                                                                                            |
| ------------------ | ---------------------------------------------------------------------------------------------------------------- |
| Pre-trained models | No pre-trained weights allowed (including those trained on non-public data)                                      |
| LLM APIs           | No LLM API calls (GPT-4, Claude, etc.)                                                                           |
| Libraries          | Only standard NOAI libraries allowed (`numpy`, `pandas`, `scikit-learn`, `torch`, `xgboost`, `lightgbm`, etc.)   |
| External data      | Public GPU/LLM specs are allowed, but **any external copies or variants of this dataset are strictly forbidden** |

## References

1. [_Watt Counts: An Empirical Study of GPU Power Consumption and Performance in LLM Inference_](https://arxiv.org/pdf/2604.09048) — original paper for this dataset
2. _LLM Inference Performance Engineering_ — NVIDIA Technical Blog series
3. _Roofline Model_ — classic performance analysis framework

## License

This dataset and problem statement are licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
