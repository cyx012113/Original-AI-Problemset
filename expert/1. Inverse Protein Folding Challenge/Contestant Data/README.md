# 🧬 逆蛋白质折叠挑战：当 AI 学会“看图猜字” 🔮

[![Challenge](https://img.shields.io/badge/NOAI-2026-blue)](https://github.com/your-repo)
[![Protein Folding](https://img.shields.io/badge/Inverse-Folding-green)](https://arxiv.org/abs/1234.56789)

> **“给你一张蛋白质的 3D 自拍，AI 居然能猜出它原来长啥样？”**  
> —— 没错，这就是 **逆蛋白质折叠**。就像看到一栋摩天大楼的建筑图纸，反推需要多少吨钢材和玻璃。

<div align="center">

<!-- 一个简单的SVG卡通：蛋白质结构→问号→氨基酸序列 -->
<svg width="400" height="120" xmlns="http://www.w3.org/2000/svg">
    <!-- 背景 -->
    <rect width="400" height="120" fill="#f8f9fa" rx="10"/>
    <!-- 左侧：蛋白质折叠结构（螺旋 + 箭头） -->
    <path d="M30,60 Q50,30 70,60 T110,60 T150,60" stroke="#2c3e50" stroke-width="3" fill="none"/>
    <circle cx="40" cy="55" r="4" fill="#e74c3c"/>
    <circle cx="70" cy="50" r="4" fill="#3498db"/>
    <circle cx="100" cy="60" r="4" fill="#2ecc71"/>
    <circle cx="130" cy="55" r="4" fill="#f39c12"/>
    <!-- 箭头 -->
    <line x1="180" y1="60" x2="220" y2="60" stroke="#7f8c8d" stroke-width="3" marker-end="url(#arrow)"/>
    <defs>
        <marker id="arrow" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
            <path d="M0,0 L0,6 L9,3 z" fill="#7f8c8d"/>
        </marker>
    </defs>
    <!-- 右侧：氨基酸序列（字母串） -->
    <text x="240" y="55" font-family="monospace" font-size="14" fill="#2c3e50">A C D E F G H</text>
    <text x="240" y="75" font-family="monospace" font-size="14" fill="#2c3e50">I K L M N P Q</text>
    <text x="240" y="95" font-family="monospace" font-size="14" fill="#2c3e50">R S T V W Y ?</text>
    <!-- 表情 -->
    <text x="360" y="30" font-size="24">🤔</text>
    <text x="360" y="100" font-size="24">💡</text>
</svg>

_“从结构到序列，就像从脚印推测动物——但这次是 AI 上场！”_

</div>

---

## 🧪 题目简介

你拿到了一份 **蛋白质的 3D 骨架坐标**（只有 $C_\alpha$ 原子的位置），你的任务是 **猜出每个位置上原本是哪种氨基酸**。  
这就是 **逆蛋白质折叠**（Inverse Protein Folding）——蛋白质设计领域的“读心术”。

- **输入**：$L$ 个点的$(x,y,z)$坐标（$L$ 在 $50\sim 800$之间）
- **输出**：每个点对应的氨基酸编号（$0\sim 19$，共 $20$ 种）
- **数据来源**：CATH 4.4 非冗余数据集（已贴心剔除缺失坐标的残基）
- **训练集**：$30000$ 个样本，提供坐标 + 标签
- **A 榜测试集**：$2000$ 个样本，只有坐标（用来悄悄验证你的模型）
- **B 榜测试集**：$1938$ 个样本，只有坐标（最终排名靠它）

> 😅 **温馨提示**：不要试图用肉眼观察坐标猜氨基酸——那是超人的工作。让神经网络替你打工吧。

---

## 📊 评分标准：$F_2$ 分数（召回率狂喜）

我们不玩准确率那种“及格万岁”的游戏。  
**$F_2$ 分数**把召回率的权重拉满（精确率:召回率 = $1:4$），意味着：

- ✅ **多找出几个稀有的氨基酸**（比如色氨酸W、半胱氨酸C）比少犯错更重要
- ❌ 漏掉一个关键残基 → 扣大分（就像忘记放盐的菜，没法补救）

公式长这样（别怕，代码里有现成的）：

$$
F_2 = 5 \cdot \frac{\text{Precision} \cdot \text{Recall}}{4 \cdot \text{Precision} + \text{Recall}}
$$

最终成绩 = 所有样本的 $F_2$ 分数取平均（每个样本内部先对20类取宏平均）。

> 🎯 **目标**：让你的模型像“蛋白质侦探”，宁可多抓几个嫌疑人，也不能放走真凶。

---

## 🚫 约束条件（“禁止令”）

1. ❌ **不准用预训练模型**（ESM、AlphaFold、ResNet50... 通通不许）。你必须从零开始训练，但可以用它们的 **网络结构** 当参考。
2. ❌ **不准调用外部 LLM API**（GPT、Claude：谢谢，我们不约）。
3. ❌ **不准使用第三方妖艳库**（比如 `mmproteo`、`capipy`）。
4. ⏱️ **训练时间 $\le$ 10 分钟**（T4 显卡上，超过 10 分钟会被评委笑）。
5. 🧠 **参数量 $\le 1^6$**（别堆几亿参数，T4 会哭）。

> 😈 违反以上任意一条，你的模型会被罚去折叠 $100000000$ 个错误折叠的蛋白质。

---

## 📁 数据格式（CSV，简单粗暴）

### `train.csv` 长这样

| domain_id | coords                      | labels            |
| --------- | --------------------------- | ----------------- |
| `12asA00` | `12.345,23.456,34.567, ...` | `5,12,3,19,1,...` |

- `coords`：`x1,y1,z1,x2,y2,z2,...`（每个坐标保留 $3$ 位小数）
- `labels`：$0\sim 19$ 的数字串，长度 = 残基数

### `A.csv` 和 `B.csv`

只有 `domain_id` 和 `coords`，没有 `labels`（因为要你自己猜）。

> 📌 **注意**：所有样本已经移除了缺失坐标的残基，所以 `coords` 和 `labels` 组数永远一致。不用操心 `mask` 了，省心！

---

## 🚀 提交格式

你需要输出两个文件：

- `A_predict.csv`
- `B_predict.csv`

每个文件包含两列：

| domain_id | predictions       |
| --------- | ----------------- |
| `12asA00` | `5,12,3,19,1,...` |

- `predictions`：预测的类别编号（$0\sim 19$），长度必须和原文件中的残基数相等。
- **长度不对齐** → 该条数据直接0分（残酷但公平）。

> 💡 小贴士：用 `len(coords.split(','))//3` 算出残基数，然后输出同样长度的预测串。

---

## 🧠 Baseline 思路（给你打个样）

我们提供了一个 **极简基线模型**（独立残基 MLP），它把每个氨基酸当成孤立的点，只用它的 $(x,y,z)$ 坐标分类。
预期 $F_2$ 分数 **< 0.3**（相当于盲猜），你可以轻松超越它。

#### 给点提示（看看就好）

**改进方向**（由易到难）：

1. 加入 **相对位置编码**（让模型知道谁挨着谁）
2. 用 **图神经网络**（把每个残基和空间最近的 K 个邻居连起来）
3. 提取 **旋转平移不变特征**（距离、二面角、局部方向）
4. 用 **Transformer** 捕捉长程依赖（注意：T4 显存有限，别太贪心）

> 😎 如果你能想到用 **SE(3)-等变网络**，评委可能会给你鼓掌。

---

## 📚 参考资料（助你成为逆折叠大师）

- [CATH 4.4 论文](https://doi.org/10.1093/nar/gkae1087)（数据集来源，记得引用）
- PiFold、ProteinMPNN（ArXiv 上随便搜，看懂算你赢）
- PyTorch 官方教程（不会PyTorch？先去学两天）

---

## 🎉 最后的话

**逆折叠** 不仅是算法竞赛，更是通往 **AI设计新蛋白质** 的大门。
也许你的模型将来能设计出 **分解塑料的酶**、**靶向癌症的药物**，甚至 **不会让人过敏的蛋白质**。

但今天，你只需要让它学会 **从坐标猜氨基酸**。
Go, build, and let the $F_2$ be with you! 🦾

## 数据来源与许可

本题所使用的数据集基于 CATH 4.4 非冗余数据集（Waman et al., 2025）进行筛选与处理。原始 CATH 4.4 数据由 [UCL 研究团队](https://www.ucl.ac.uk/research/our-people-and-teams) 发布，采用 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/legalcode.txt)。
我们对原始数据进行了以下处理：移除缺失 Cα 坐标的残基，并将序列坐标和氨基酸标签按样本整理为 CSV 格式。

本题目的文本、题目设计及数据集衍生部分由 [cyx012113](https://github.com/cyx012113) 创作，整体采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.txt) 许可。
这意味着你可以自由分享、改编这道题目，但必须给出适当署名、不得用于商业目的，且任何改编作品必须采用相同的许可证。

- 原始 CATH 4.4 许可：<https://creativecommons.org/licenses/by/4.0>
- 本题目许可：<https://creativecommons.org/licenses/by-nc-sa/4.0>

<div align="center"> &copy cyx012113 2026. </div>
