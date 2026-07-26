# 主要参考资料

## 数据与同任务基线

1. Mingde Yao, Menglu Wang, King-Man Tam, Lingen Li, Tianfan Xue,
   Jinwei Gu. **PolarFree: Polarization-based Reflection-Free Imaging.**
   CVPR 2025.  
   [CVF论文](https://openaccess.thecvf.com/content/CVPR2025/html/Yao_PolarFree_Polarization-based_Reflection-Free_Imaging_CVPR_2025_paper.html) ·
   [代码与数据](https://github.com/mdyao/PolarFree)

2. Chenyang Lei, Xuhua Huang, Mengdi Zhang, Qiong Yan, Wenxiu Sun,
   Qifeng Chen. **Polarized Reflection Removal with Perfect Alignment in
   the Wild.** CVPR 2020.  
   [CVF论文](https://openaccess.thecvf.com/content_CVPR_2020/html/Lei_Polarized_Reflection_Removal_With_Perfect_Alignment_in_the_Wild_CVPR_2020_paper.html)

## 主要架构参考

3. Kei Ikemura, Yiming Huang, Felix Heide, Zhaoxiang Zhang, Qifeng Chen,
   Chenyang Lei. **Robust Depth Enhancement via Polarization Prompt Fusion
   Tuning.** CVPR 2024.  
   [CVF论文](https://openaccess.thecvf.com/content/CVPR2024/html/Ikemura_Robust_Depth_Enhancement_via_Polarization_Prompt_Fusion_Tuning_CVPR_2024_paper.html) ·
   [代码](https://github.com/lastbasket/Polarization-Prompt-Fusion-Tuning)

   本项目主要参考其并行偏振分支和深层融合思想，但认为“是否真正利用了
   偏振、在何处注入多少偏振”仍需要显式对照验证。

4. Shuangfan Zhou, Chu Zhou, Youwei Lyu, Heng Guo, Zhanyu Ma, Boxin Shi,
   Imari Sato. **PIDSR: Complementary Polarized Image Demosaicing and
   Super-Resolution.** CVPR 2025.  
   [CVF论文](https://openaccess.thecvf.com/content/CVPR2025/html/Zhou_PIDSR_Complementary_Polarized_Image_Demosaicing_and_Super-Resolution_CVPR_2025_paper.html) ·
   [代码](https://github.com/PRIS-CV/PIDSR)

   本项目参考其显式Stokes监督、互补子任务和阶段式结构，但PIDSR处理的是
   去马赛克与超分辨率，不能直接证明RGB↔Polar反馈适用于反射去除。

## RGB到偏振的边界

5. **RGB-to-Polarization Estimation: A New Task and Benchmark Study.**
   NeurIPS 2025 Datasets and Benchmarks Track.  
   [论文入口](https://proceedings.neurips.cc/paper_files/paper/2025/hash/30697d9ef8ce55de6ccc38e043a94142-Abstract-Datasets_and_Benchmarks_Track.html)

该方向提示RGB到偏振存在物理歧义，预测偏振不能天然替代实测偏振；本项目
因此把RGB→Polar定位为“对实测偏振的锚定纠正”，并要求通过下游去反射门。

