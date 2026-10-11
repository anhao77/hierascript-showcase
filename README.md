# HieraScript：基于参数高效视觉-语言学习的分层场景文字语系识别

[English](README.en.md) | 简体中文

[![Paper](https://img.shields.io/badge/Paper-PRCV%202026%20(camera--ready)-2f6fb0)](#引用)
[![Code](https://img.shields.io/badge/Code-coming%20soon-e8a33d)](#发布状态)
[![Dataset](https://img.shields.io/badge/Dataset-HSI--3L%20coming%20soon-e8a33d)](#发布状态)
[![Affiliation](https://img.shields.io/badge/NJUST-School%20of%20Cyber%20Science%20%26%20Engineering-6b7280)](#作者与单位)
[![Maintainer](https://img.shields.io/badge/Maintainer-anhao77-181717?logo=github)](https://github.com/anhao77)

> **发布状态**：论文已 camera-ready，官方**代码与数据集正在准备开放**，正式仓库为
> **[billqxg/HieraScript](https://github.com/billqxg/HieraScript)**。
> **本页面仅为项目展示页，不提供任何源码、数据集或模型权重。**

场景文字语系识别（Scene Text Script Identification）用于判断自然场景图像中的文字属于哪一种书写系统，
是多语言场景文字理解的基础环节，为文字识别、信息检索等下游任务提供重要先验。现有方法通常把它当作
**扁平分类**问题，并依赖视觉-语言模型的**全参数微调**，忽略了语系之间天然存在的**层级关系**。
本文提出 **HieraScript**：在预训练视觉-语言模型之上，将**参数高效迁移学习**与**层级语义建模**结合，
实现层级语系识别。

![Teaser](assets/teaser.png)

*(a) 传统扁平语系识别：全量微调 + 单层分类头；(b) 本文的分层范式：冻结视觉-语言骨干，
按层级划分的专属分类器，并引入置信度校准。*

## 主要贡献

- **HSI-3L 基准数据集**：带**显式三级层级标注**的多语言场景文字语系识别数据集，覆盖 **33 种语言**、
  **5 个 Macro 类 / 19 个 Script 类 / 33 个 Language 类**，共 **472,087 张**图像。
- **HieraScript 框架**：基于冻结的 CLIP 骨干，提出**半解耦层级表示**（Semi-Decoupled Hierarchical Representation），同时建模细粒度字形特征与层级语义，且只训练极少参数。
- **层级一致性约束与置信度校准**：通过一致性蒸馏与层级违背惩罚（Hierarchical Consistency Constraints）保证跨层级预测一致；推理阶段用**基于距离的校准器**（Distance-based Calibrator）估计预测可靠性，提升歧义与退化场景下的鲁棒性。
- **效果与泛化性**：在 HSI-3L 与 CVSI、MLe2e、SIW-13、MLT 等公开基准上优于扁平 CLIP 基线（zero-shot、全量微调、CoOp、PromptSRC），在跨域设定下亦与专用语系识别方法相当或更优。

## 方法概览

![Framework](assets/framework.png)

HieraScript 由四个部分构成：

1. **冻结的 CLIP 视觉编码器**：预训练骨干（默认 ResNet-50 变体）全程冻结，实现参数高效的特征提取。
2. **半解耦层级表示**：三条层级专属卷积分支产生各层级特征，子层级表示通过嵌套特征融合继承父层级语义信息。
3. **层级一致性约束**：将 Language 级预测经层级映射矩阵投影为 Script / Macro 级的软标签 （一致性蒸馏），并施加层级违背惩罚，避免子节点置信度高于父节点；配合动态损失平衡与基于不确定性的加权，稳定多层级优化。
4. **基于距离的置信度校准**：由分类正确的训练样本估计原型向量，在推理阶段校准歧义或退化样本的置信度。

## HSI-3L 数据集

![Hierarchy](assets/hierarchy.png)

| 属性 | 数值 |
| --- | --- |
| 语言数 | 33 |
| Macro / Script / Language 类别数 | 5 / 19 / 33 |
| 图像总数 | 472,087 |
| 训练 / 测试划分 | 350,205 / 121,882 |
| 真实场景 / 合成 | 30 种语言（真实）+ 3 种语言（合成：保加利亚语、波斯语、乌克兰语） |

真实图像来自多个公开多语言文字数据集：CVSI、SIW-13、MLe2e、MLT2017、KhmerST、
Indic Scene Text on the Roadside、Bharat Scene Text Dataset、TC-STR 7k-word Dataset；
低资源语言使用开源文本生成器合成补充。所有样本统一映射到三级层级标签空间，
评测指标为 Macro（M-Acc）、Script（S-Acc）、Language（L-Acc）三级 Top-1 准确率以及层级 F1（H-F1）。

![Dataset samples](assets/dataset-samples.png)

## 实验结果

### 与扁平 CLIP 方法及不同骨干的对比（%）

| 方法 | 骨干 | 总参数 (M) | 可训练参数 (M) | CVSI | MLe2e | SIW-13 | MLT | HSI-3L (L-Acc) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| CLIP Zero-shot | RN50 | 102.01 | 0 | 46.76 | 70.02 | 54.50 | 39.87 | 20.71 |
| CLIP 全量微调 | RN50 | 102.01 | 102.01 | 69.70 | 95.63 | 89.31 | 71.90 | 88.20 |
| CoOp | RN50 | 76.72 | 0.01 | 57.90 | 95.20 | 69.00 | 62.10 | 58.60 |
| PromptSRC | RN50 | 115.05 | 0.02 | 50.20 | 75.10 | 49.60 | 47.10 | 44.10 |
| **HieraScript（本文）** | RN50 | 72.19 | 33.87 | 98.19 | 95.78 | 97.64 | 72.08 | **91.03** |
| **HieraScript（本文）** | RN50x4 | 128.29 | 41.15 | 98.30 | 95.78 | 97.69 | 73.22 | 91.61 |
| **HieraScript（本文）** | RN50x16 | 216.54 | 49.21 | 98.93 | 94.51 | 96.62 | 77.16 | 92.11 |
| **HieraScript（本文）** | ViT-B/16 | 186.98 | 17.75 | 98.09 | 98.52 | 96.45 | 72.69 | 90.71 |
| **HieraScript（本文）** | ViT-B/32 | 188.52 | 15.64 | **99.28** | 98.84 | 97.95 | 73.04 | 89.68 |

### 与典型语系识别方法的对比（%）

| 方法 | CVSI | MLe2e | SIW-13 |
| --- | --- | --- | --- |
| Shi et al. | 94.30 | - | 89.40 |
| Gomez et al. | 97.20 | 94.40 | 94.80 |
| Cheng et al. | 98.60 | - | 96.50 |
| Ma et al. | 98.78 | 97.20 | 97.30 |
| Dutta et al. | 98.97 | 95.01 | 95.70 |
| Mahajan et al. | 97.40 | - | - |
| Guo et al. | 93.50 | 98.74 | - |
| FAS-Res2Net | 96.00 | - | 94.70 |
| SANet | 99.03 | - | 96.18 |
| Split-Net | 99.05 | 98.85 | 97.91 |
| BLCT-sum | 97.03 | 96.10 | 92.83 |
| **HieraScript（本文）** | **99.28** | 98.84 | **97.95** |

### 消融实验（HSI-3L，%）

| 变体 | H-F1 | M-Acc | S-Acc | L-Acc |
| --- | --- | --- | --- | --- |
| 基线 | 96.31 | 98.72 | 96.23 | 90.25 |
| + 层级一致性损失 | 96.46 | 98.73 | 96.28 | 90.41 |
| + 自适应损失平衡 | 96.55 | 98.73 | 96.53 | 90.92 |
| + 基于距离的置信度校准 | 96.39 | 98.72 | 96.37 | 90.45 |
| + 文本编码器 | 96.04 | 98.47 | 95.88 | 89.78 |
| 完整模型（去掉特征融合） | 96.51 | 98.88 | 96.60 | 90.53 |
| **完整模型** | **96.66** | **98.92** | **96.68** | **91.03** |



## 引用

如果本工作对你的研究有帮助，欢迎引用：

```bibtex
@inproceedings{qin2026hierascript,
  title     = {HieraScript: Hierarchical Scene Text Script Identification via Parameter-Efficient Vision-Language Learning},
  author    = {Xugong Qin, Li Ling, Xulin Zhu, Zhicong Yu, Junhao Chen, Xinran Zhang, Yuanyuan Wang, and Gangyan Zeng},
  booktitle = {Pattern Recognition and Computer Vision (PRCV)},
  year      = {2026},
  note      = {To appear}
}
```

## 作者与单位

Xugong Qin, Li Ling, Xulin Zhu, Zhicong Yu, Junhao Chen, Xinran Zhang, Yuanyuan Wang, and Gangyan Zeng

南京理工大学网络空间安全学院（School of Cyber Science and Engineering, Nanjing University of Science and Technology, Nanjing, China）


## 致谢

本工作受江苏省基础研究计划（BK20251441、BK20251414）与江苏省大学生创新训练计划（S202510288013）资助。

## 联系方式

论文相关问题请联系作者Xugong Qin （[@billqxg](https://github.com/billqxg)）。本展示页由共同作者 Zhicong Yu（[@anhao77](https://github.com/anhao77)）维护，欢迎到 [我的 GitHub 主页](https://github.com/anhao77) 交流。
