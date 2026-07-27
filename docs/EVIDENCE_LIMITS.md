# 证据、限制与禁止外推

最后更新：2026-07-27。

## 一、实验直接支持的结论

- PolaRGB本地接口已完成极小规模审计。
- E1 correct优于zero和shuffled，模型确实依赖正确配对偏振。
- E1-large是当前单向Polar→RGB开发基线。
- 当前FiLM式R2P在联合训练、head-only训练和候选偏振GT监督条件下，均未
  产生稳定RGB去反射增益。
- TP10中偏振重建误差下降，但RGB指标没有改善。
- TP11中`state_gt−pobs`与`combined−pobs`均未达到预注册的`+0.05 dB`门。
- TP18中二阶段correct偏振优于zero和shuffled，表明consumer可以利用配对
  偏振弱信号。
- TP18没有支持raw4+Stokes表示主效应、adaptive PSNR主效应或二者正协同。

## 二、综合解释

现有正负证据共同提示：

> 偏振信息的存在、偏振表示或预测的准确性、以及RGB网络能否有效消费该信息，
> 是三个需要分别验证的问题。

当前最值得检验的瓶颈假设是：偏振表示、跨模态consumer和最终RGB任务之间
存在接口错配。它是对多项证据的综合解释，不是已定位的唯一因果机制。

TP11还提示，保留污染的`P0_obs`而仅替换GT `q/u`并不构成完整的clean
Stokes；因此其STOP不能外推为所有物理一致的full-Stokes校正均无效。

## 三、仍待验证的假设

- 现有R3 consumer能否随理想full-Stokes校正产生稳定正响应；
- downstream RGB loss能否让Stokes residual producer获得任务对齐；
- 暗区、饱和或物理非法状态是否能预测有害偏振注入；
- 经过校准的可靠性是否能在保留平均增益的同时降低harm rate；
- 在单轮机制成立后，有限recurrence是否优于等FLOPs RGB refinement。

## 四、不能说

- “已经证明双向纠偏无效。”
- “偏振GT没有用。”
- “偏振预测越准，去反射一定越好。”
- “TP11证明真实`q/u`永远无助于去反射。”
- “full-GT的大幅提升证明GT偏振状态有效。”  
  full-GT同时包含clean intensity，不能归因于`q/u`。
- “raw4+Stokes比`P0/q/u`更强。”
- “当前adaptive gate稳定优于concat。”
- “当前gate就是偏振置信度或逐像素任务效用。”
- “R3或R4已经是最终模型。”
- “E1已经在独立测试集上普遍优于E0。”
- “双向、循环、Stokes或普通门控本身构成创新。”
- “PPFT、PIDSR、RP2PN或PICDS-Net已经直接证明本项目方案成立。”

## 五、统计和实验设计限制

1. E0/E1/E2主要为单seed开发实验。
2. 固定开发test已被反复用于路线选择，不能继续承担最终独立验证。
3. 场景bootstrap描述当前场景不确定性，不覆盖训练随机性。
4. TP09/TP10使用双seed，增强了特定失败结论的可信度，但仍不是论文级统计。
5. TP11只执行预注册Phase 1单seed筛选，结论限于其state-only consumer。
6. TP18是单seed inner-dev pilot，best checkpoint也由同一inner-dev选择。
7. TP18 R4−R0虽为`+0.2198 dB`、8/12场景胜，但95% CI跨0，4/12场景退化。
8. R3是看到完整2×2结果后的事后候选，后续必须重新预注册。

## 六、已知实现与报告风险

- 早期Dataset持有内部RandomState并使用多个worker，可能产生相关裁剪。
- 早期运行不是严格确定性。
- TP10在clamp后的四角输出上计算部分损失，可能存在饱和梯度。
- TP09/TP10曾把LPIPS胜场方向统一按差值大于零统计；正式主门未依赖该字段。
- TP11报告中combined residual的符号文字与代码曾不一致，但两端状态同时输入，
  信息量未改变，主门不受影响。
- 微小开发差值不能脱离上述限制被解释为稳定增益。

## 七、已作废设计

旧显式Stokes V1先把`q/u`投影到单位圆盘，再计算：

\[
L_{\mathrm{phys}}=\max(0,\hat q^2+\hat u^2-1).
\]

投影已保证平方和不超过1，因此该损失恒为零。该方案未执行，历史实验结果
不受影响，但后续不得复用这一运算顺序。

## 八、下一阶段的最低证据要求

- oracle：`λ=0`严格一致、端点正增益、scene-level趋势和严重退化报告；
- producer：相对observed产生RGB净收益，并与纯polar reconstruction对照；
- reliability：与真实误差或风险有可量化关系，不能只看PSNR；
- recurrence：相对等FLOPs控制有增益，且有害场景比例不增加；
- final：使用未参与路线选择的评估协议，并保留逐场景与负面结果。
