# 主要参考资料

最后核对：2026-07-27。

本列表用于说明研究依据与创新边界。文献中的任务、数据与接口并不都与本项目
相同，因此不能仅凭结构相似就推断本项目机制有效。

## 数据与同任务研究

1. Mingde Yao, Menglu Wang, King-Man Tam, Lingen Li, Tianfan Xue,
   Jinwei Gu. **PolarFree: Polarization-based Reflection-Free Imaging.**
   CVPR 2025.  
   [CVF论文](https://openaccess.thecvf.com/content/CVPR2025/html/Yao_PolarFree_Polarization-based_Reflection-Free_Imaging_CVPR_2025_paper.html) ·
   [代码与PolaRGB数据](https://github.com/mdyao/PolarFree)

   本项目使用PolaRGB开展开发期验证。公开论文支持偏振是反射去除的重要
   物理线索，但不直接证明本项目的融合或双向机制。

2. Chenyang Lei, Xuhua Huang, Mengdi Zhang, Qiong Yan, Wenxiu Sun,
   Qifeng Chen. **Polarized Reflection Removal with Perfect Alignment in
   the Wild.** CVPR 2020.  
   [CVF论文](https://openaccess.thecvf.com/content_CVPR_2020/html/Lei_Polarized_Reflection_Removal_With_Perfect_Alignment_in_the_Wild_CVPR_2020_paper.html)

## 偏振融合与显式物理表示

3. Kei Ikemura, Yiming Huang, Felix Heide, Zhaoxiang Zhang, Qifeng Chen,
   Chenyang Lei. **Robust Depth Enhancement via Polarization Prompt Fusion
   Tuning.** CVPR 2024.  
   [CVF论文](https://openaccess.thecvf.com/content/CVPR2024/html/Ikemura_Robust_Depth_Enhancement_via_Polarization_Prompt_Fusion_Tuning_CVPR_2024_paper.html) ·
   [代码](https://github.com/lastbasket/Polarization-Prompt-Fusion-Tuning)

   PPFT提供并行偏振分支与深层融合参考，但任务是深度增强，不直接给出反射
   去除中的偏振任务效用或可靠性定义。

4. Shuangfan Zhou, Chu Zhou, Youwei Lyu, Heng Guo, Zhanyu Ma, Boxin Shi,
   Imari Sato. **PIDSR: Complementary Polarized Image Demosaicing and
   Super-Resolution.** CVPR 2025, pp. 16081–16090.  
   [CVF论文](https://openaccess.thecvf.com/content/CVPR2025/html/Zhou_PIDSR_Complementary_Polarized_Image_Demosaicing_and_Super-Resolution_CVPR_2025_paper.html) ·
   [代码](https://github.com/PRIS-CV/PIDSR)

   PIDSR支持显式Stokes信息、互补子任务和阶段式处理具有研究先例；其任务是
   偏振去马赛克与超分辨率，不能直接证明RGB↔Polar反馈适用于反射去除。

## 循环、双流与偏振恢复的创新边界

5. Wenjiao Bian, Yusuke Monno, Masatoshi Okutomi.
   **Reflection Removal Using Recurrent Polarization-to-Polarization
   Network.** ICASSP 2024, pp. 2570–2574.  
   [DOI:10.1109/ICASSP48485.2024.10446021](https://doi.org/10.1109/ICASSP48485.2024.10446021) ·
   [arXiv](https://arxiv.org/abs/2402.18178)

   RP2PN使用偏振到偏振的反射/透射预测和循环细化，因此“循环互相促进”本身
   不能作为本项目的新颖性主张。

6. Lijun Deng, Yang Xu, Hedong Liu, Zixian Liu, Zhen Yang, Xin Zhou,
   Zhihua Xie, Haofeng Hu. **Polarization Information Restoration for
   Visual Reflection Removal via Cross Dual-Stream Network.**
   Knowledge-Based Systems, 341:115811, 2026.  
   [DOI:10.1016/j.knosys.2026.115811](https://doi.org/10.1016/j.knosys.2026.115811)

   PICDS-Net联合反射去除与偏振恢复，并使用渐进双流和跨分支交互。因此
   “双流、双向交互、偏振恢复或普通门控”均不足以单独构成本项目创新。

## 不确定性与可靠性定义

7. Alex Kendall, Yarin Gal. **What Uncertainties Do We Need in Bayesian
   Deep Learning for Computer Vision?** NeurIPS 2017.  
   [arXiv:1703.04977](https://arxiv.org/abs/1703.04977)

   该文提供异方差不确定性建模的通用依据，但不直接定义本项目的偏振测量
   可靠性。若后续使用相关损失，仍需独立验证校准、risk–coverage和任务风险。

## 本项目对文献的使用原则

- 文献支持的是机制先例、数学定义或实验设计依据；
- 本项目的正负结论来自自身受控实验；
- “偏振有用”不等于任意偏振接口都有效；
- “已有双向/循环先例”也不等于本项目必须采用双向/循环；
- 创新边界必须落到可校验的可靠性、任务对齐或错误传播控制，而不是结构名词。
