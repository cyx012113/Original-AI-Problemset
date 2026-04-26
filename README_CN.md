# Original-AI-Problemset（原创AI赛题集）

![CC BY-NC-SA 4.0](http://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-nc-sa.svg)

一套覆盖多个难度的**原创人工智能竞赛题目**合集。
每道题目均包含中英双语题面、基线代码、公开/私有数据集以及官方评分材料，可直接用于竞赛、教学或个人练习。

## 难度分类

- **beginner**（初级） – 入门级任务，对背景知识要求较低。
- **intermediate**（中级） – 典型的机器学习或简单深度学习问题。
- **advanced**（高级） – 复杂建模、多模态数据或进阶架构。
- **expert**（专家级） – 研究导向型挑战，常涉及特定领域知识（如计算生物学、蛋白质折叠等）。

## 仓库结构

每题均严格遵循以下目录布局：

```tree
{Problem-Name}/
├── LICENSE
├── Contestant Data/              # 选手材料
│   ├── {Problem-Name-EN}.md      # 英文题面（Markdown）
│   ├── {Problem-Name-EN}.pdf     # 英文题面（PDF）
│   ├── {Problem-Name-CN}.md      # 中文题面（Markdown）
│   ├── {Problem-Name-CN}.pdf     # 中文题面（PDF）
│   ├── baseline.ipynb            # 简易基线方案
│   ├── Data/
│   │   ├── train.csv
│   │   ├── A.csv                 # 公开测试集
│   │   └── B.csv                 # 私有测试集
│   ├── Paper_{Reference}.pdf     # 数据集论文
│   ├── README.md                 # 本题说明
│   └── LICENSE
└── Rating Data/                  # 评分材料
    ├── A_ground_truth.csv
    ├── B_ground_truth.csv
    ├── std.ipynb                 # 参考正解代码
    └── LICENSE
```

- **Contestant Data** – 提供给参赛者的全部内容。
- **Rating Data** – 真实标签与评分代码；在正式比赛期间通常不公开。

## 开源协议

本仓库所有题目均采用 **知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议（CC BY-NC-SA 4.0）** 发布。
详情请参阅根目录及各题目文件夹内的 `LICENSE` 文件。

## 使用方式

- **竞赛举办方**：可 Fork 相关题目，部署至自有平台，并使用 `Rating Data` 进行评分。
- **学习者与从业者**：克隆仓库，阅读题面，自由尝试解决方案。
- **贡献者**：欢迎通过 Pull Request 提交新的原创题目。请严格遵循上述目录结构，并同时提供中英文题面。

## 贡献指南

1. 在 `problems/{难度等级}/` 下新建题目文件夹。
2. 按模板填充 `Contestant Data` 与 `Rating Data` 的所有文件。
3. 检查文本格式与许可证完整性。
4. 发起 Pull Request，并简要描述题目内容。

如有疑问，欢迎提交 Issue。
