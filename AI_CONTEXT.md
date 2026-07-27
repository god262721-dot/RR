
# AI_CONTEXT：新AI必读的项目总览

最后更新：2026-07-27。

## 1. 角色与证据纪律

你正在协助一个计算机视觉研究课题，而不是开发正式软件产品。必须遵守：

1. 区分**实验直接证据、综合解释、待验证假设**。
2. 不因为用户或导师偏好双向纠偏，就默认双向纠偏已经成立。
3. 优先提出低成本、单变量、有停止条件且能被证伪的实验。
4. 如实保留负面结果，不选择性汇报有利checkpoint或seed。
5. 不把已反复用于路线决策的开发场景称为最终独立测试集。
6. 不把普通sigmoid gate称为“校准置信度”或“任务效用”，除非有相应监督和诊断。

## 2. 当前研究问题

任务是偏振引导的RGB图像反射去除。输入包括带反射RGB和与其配对的四角
偏振观测，目标是恢复无反射RGB。

项目最初候选方向包括：

- 偏振测量可靠性；
- 显式Polar→RGB融合；
- RGB→Polar→RGB双向纠偏；
- 必要时的有限轮次迭代。

经过E0至TP18，核心问题已经收窄为：

> 正确配对的偏振具有增量信息，但如何让RGB恢复网络稳定、可解释地消费这些
> 信息，并避免把偏振准确性误当成任务效用？

暂定题目可使用“偏振引导的图像反射去除方法研究”，副标题可使用“面向偏振
可靠利用与显式跨模态交互”。暂不把“双向纠偏”写成已经成立的前提。

## 3. 数据与偏振表示

数据来自PolaRGB。本地协议使用：

- 带反射RGB；
- 0°、45°、90°、135°四角偏振图；
- 无反射RGB GT；
- 数据中对应的四角GT图暂称“候选干净偏振GT”。

在语义未进一步核验前，不应把最后一项外推为数据集官方定义的唯一偏振GT。

四角观测记为：

\[
I_0,\ I_{45},\ I_{90},\ I_{135}.
\]

当前使用：

\[
S_0=\frac{I_0+I_{45}+I_{90}+I_{135}}{2},\quad
S_1=I_0-I_{90},\quad
S_2=I_{45}-I_{135},
\]

\[
P_0=S_0/2,\quad
q=\operatorname{clamp}\left(\frac{S_1}{S_0+\epsilon},-1,1\right),\quad
u=\operatorname{clamp}\left(\frac{S_2}{S_0+\epsilon},-1,1\right).
\]

低强度区域的归一化`q/u`可能放大量化噪声；任何可靠性结论必须专门验证。

## 4. 已完成的实验链

### E0：RGB-only

- 只输入带反射RGB。
- 作用是建立训练、评估与RGB恢复基线。
- E0-large：PSNR 19.759、SSIM 0.833、LPIPS 0.154。

### E1：单向Polar→RGB

- RGB和偏振分别编码，在瓶颈处注入偏振特征。
- correct、zero、shuffled使用同一checkpoint推理。
- E1-large correct：PSNR 20.286、SSIM 0.843、LPIPS 0.149。
- zero为18.538 dB，shuffled为17.935 dB。
- 小规模E1相对E0为`+0.588 dB`，8/8场景提升。

直接证据支持：模型确实使用正确配对的偏振，单向P→R具有开发期可行性。

### E2：单轮RGB→Polar→RGB

- 在E1上增加约3.7万参数的FiLM式R2P头。
- E2-large PSNR为19.378，相对E1下降`0.908 dB`，1/8场景胜。
- 同一E2 checkpoint中R2P on相对off只有约`+0.015 dB`。

直接证据只否定当前FiLM实现；不能推广为所有双向结构无效。

### TP09：冻结E1，只训练R2P头

- 两个预注册seed，各固定2000步。
- 一个seed效应接近零，另一个best停在step 0。
- 两个seed都未通过数值价值与偏振特异性门。

### TP10：候选偏振GT监督R2P

- 冻结E1，只训练R2P和偏振decoder。
- 偏振重建MAE约由0.078降至0.061，说明辅助任务确实学到偏振。
- 两个seed的R2P-on相对off的RGB PSNR约为`−0.047 dB`和`−0.048 dB`。

直接证据支持：偏振预测更准确，不会自动转化为RGB去反射收益。

### TP11：state-only polar oracle

- 二阶段consumer参数量匹配，E1冻结。
- `state_gt−pobs`：`+0.0031 dB`，95% CI
  `[-0.0752,+0.0815]`，7/12场景胜。
- `combined−pobs`：`+0.0186 dB`，7/12场景胜。
- 两组均低于预注册的`+0.05 dB`门槛，Phase 1 STOP。
- `full_gt`有巨大收益，但它同时引入干净强度`P0_gt`，不能用于证明仅GT
  `q/u`有效。

该实验测试的是特定state-only consumer，不是所有物理一致Stokes接口。

### TP18：强P→R表示×融合pilot

固定E1输出`T1`，比较：

| Run | 表示 | 融合 | PSNR |
|---|---|---|---:|
| R0 | zero | adaptive | 21.3189 |
| R1 | `P0/q/u` | concat | 21.5557 |
| R2 | raw4+Stokes | concat | 21.5753 |
| R3 | `P0/q/u` | adaptive | **21.6314** |
| R4 | raw4+Stokes | adaptive | 21.5386 |

- R4−R0为`+0.2198 dB`、8/12场景胜，预注册pilot组合门通过；
- correct优于zero和shuffled，说明二阶段消费了配对偏振；
- 但R4−R0的95% CI跨0，且4/12场景退化；
- raw4+Stokes的平均主效应为负；
- adaptive的PSNR平均主效应为`+0.0195 dB`、5/12场景胜；
- 表示×融合交互为负；
- R3数值最好，但属于事后候选。

因此，TP18只提供“存在弱配对偏振信号”的证据，没有验证某个最终表示或gate。

## 5. 当前认识：三个概念不能混同

### 测量可靠性

回答偏振观测是否受暗区、饱和、量化或物理不可实现状态污染。它需要偏振误差、
校准、risk–coverage等专门证据。

### 预测准确性

回答预测的Stokes或四角图是否更接近候选偏振GT。TP10已表明该指标可以改善。

### 任务效用

回答该偏振信息是否能降低最终RGB恢复误差。它不能由偏振MAE或一个普通gate
的数值直接替代。

当前最有根据的综合解释是：**表示、consumer接口和下游任务之间可能存在错配。**
这仍是优先假设，不是已经定位的唯一根因。

## 6. 当前下一步：training-free full-Stokes oracle响应曲线

不要立即训练新的R2P、confidence或多轮网络。先在现有R3 consumer上检验：

\[
\mathbf S_\lambda=(1-\lambda)\mathbf S_{obs}+\lambda\mathbf S_{gt},
\quad \lambda\in\{0,0.25,0.5,0.75,1\}.
\]

执行边界：

- 不训练，只读inner-dev；
- 将`Sλ`一致转换为R3所需表示；
- `λ=0`必须与observed路径逐像素一致；
- 报告scene-macro趋势、端点差值、逐场景变化和严重退化；
- 不要求每个场景严格单调；
- 必要时用R1复核。

判定：

- R3出现稳定正响应：当前consumer可能利用理想Stokes校正，可以设计任务对齐
  的Stokes residual producer；
- R1和R3都平或负：结合TP11，暂停当前双向训练并重新设计consumer；
- producer未产生RGB净收益前，不进入learned confidence；
- recurrence最后验证，并使用等FLOPs RGB refinement控制。

## 7. 当前不应建议

在没有新证据前，不要直接建议：

- 把R3或R4冻结为最终模型；
- 增加多轮循环；
- 扩大旧FiLM式R2P头；
- 只调整`gamma/beta`幅度或损失权重；
- 把raw4+Stokes称为优于`P0/q/u`；
- 把当前adaptive gate称为可靠性或任务效用；
- 用更低偏振MAE宣称去反射机制成功；
- 重复读取开发test来选择路线；
- 把“双向、循环、Stokes或门控”本身写成创新。

## 8. 已知限制

- E0/E1/E2主要是单seed开发实验。
- 固定开发test已反复用于路线决策。
- 场景bootstrap不覆盖训练随机性。
- TP09/TP10提高了失败结论的seed覆盖，但仍不是最终论文级统计。
- TP11是单seed Phase 1筛选，且只覆盖特定state-only consumer。
- TP18是单seed inner-dev pilot，best也由同一inner-dev选择。
- TP18 R4有4/12场景退化，最差场景约`−1.16 dB`和`−1.13 dB`。
- 早期实验存在多worker随机裁剪相关性和非严格确定性风险。
- TP09/TP10报告中的LPIPS胜场方向曾写反，但正式主门未依赖该字段。

## 9. 阅读顺序

1. `README.md`
2. `docs/PROJECT_STATUS.md`
3. `docs/METHOD_SNAPSHOT.md`
4. `docs/EVIDENCE_LIMITS.md`
5. `docs/REFERENCES.md`

讨论方案时以这些文件的当前状态为准，不根据旧任务名称推断路线仍然有效。
