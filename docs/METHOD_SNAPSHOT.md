# 方法快照

最后更新：2026-07-27。

本文件只描述已经测试的接口和当前条件路线。它不是最终网络设计。

## E0：RGB-only

```text
带反射RGB → RGB恢复主干 → 无反射RGB
```

作用：建立不使用偏振的训练与评估基线。

## E1：单向Polar→RGB

```text
带反射RGB ─→ RGB编码器 ───────────┐
四角偏振 ─→ Stokes表示 → 偏振编码器 ├→ 瓶颈注入 → RGB恢复
                                  ┘
```

同checkpoint下的correct、zero、shuffled对照表明，E1确实依赖与当前RGB
正确配对的偏振。E1回答“偏振有没有增量信息”，没有回答“如何利用最好”。

## E2：已失败的FiLM式双向实现

```text
RGB瓶颈 ─→ R2P头 ─→ gamma/beta
                         ↓
偏振潜特征 ─────────→ FiLM调制 ─→ 原P2R投影 ─→ RGB恢复
```

E2、TP09和TP10共同表明，当前FiLM潜特征反馈在联合训练、head-only训练和
候选偏振GT监督下都没有产生稳定RGB收益。

该结论针对特定实现，不是对所有双向交互的普遍否定。

## TP10：偏振监督生效但任务接口失败

```text
RGB特征 → R2P → 纠正后偏振潜变量
                    ├→ 偏振decoder → 候选四角GT监督
                    └→ 冻结P2R接口 → RGB恢复
```

偏振decoder的重建误差下降，但RGB输出没有改善。一个可能解释是同一潜变量
同时承担偏振重建与冻结P2R输入，存在目标或分布错配；这仍是机制推断。

## TP11：state-only oracle consumer

```text
固定E1输出 T1
     │
     └→ 参数匹配的轻量二阶段consumer
            ├→ zero polar
            ├→ observed P0/q/u
            ├→ observed P0 + candidate-GT q/u
            ├→ observed + GT + normalized-state difference
            └→ full candidate-GT（仅描述）
```

该consumer只接收`T1`与偏振状态，不接收原始混合RGB、E1中间特征或完整
物理一致的clean Stokes。GT state组未通过增益门；full-GT组的大幅增益主要
包含clean intensity信息，不能归因为`q/u`。

## TP18：强P→R表示×融合pilot

固定E1输出`T1`，二阶段比较：

```text
偏振表示：P0/q/u        vs raw4 + Stokes
融合方式：concat         vs adaptive
控制组：  同容量zero polar
```

矩阵结果：

```text
                    concat        adaptive
P0/q/u              R1 21.5557    R3 21.6314
raw4 + Stokes       R2 21.5753    R4 21.5386

zero + adaptive                     R0 21.3189
```

解释：

- correct对zero/shuffled的优势证明consumer消费了配对偏振；
- R3仅是事后数值候选；
- raw4+Stokes、adaptive及其交互没有获得独立主效应支持；
- 当前adaptive模块只是任务驱动gate，不具备已验证的可靠性含义。

## 当前最小诊断：full-Stokes oracle响应曲线

先不训练producer。对观测与候选GT Stokes插值：

\[
\mathbf S_\lambda
=(1-\lambda)\mathbf S_{obs}+\lambda\mathbf S_{gt},
\quad
\lambda\in\{0,0.25,0.5,0.75,1\}.
\]

```text
Sobs ──────────────┐
                   ├→ Sλ → 转换为consumer输入 → 固定R3/R1 → RGB输出
Sgt  ──────────────┘
```

最低不变量和判据：

- `λ=0`与observed路径严格一致；
- 不训练，只评估inner-dev；
- `λ=1`相对`λ=0`应出现正向scene-macro响应；
- 报告逐场景趋势和严重退化，不要求每个场景严格单调；
- polar distance按构造下降只验证插值正确，不代表RGB任务成功。

## oracle通过后的条件路线

### 1. 任务对齐的Stokes residual producer

```text
Sobs → 预测有界 ΔS0/ΔS1/ΔS2 → 物理有效性处理 → 固定consumer → RGB输出
```

第一轮只比较：

- polar reconstruction；
- polar reconstruction + downstream RGB loss。

这里的RGB loss是任务对齐目标，不等于逐像素任务效用图。

### 2. 可靠性

仅当producer平均有增益但部分场景或区域退化时，定义并验证`C_rel`：

```text
Sused = Sobs + Crel ⊙ ΔS
```

需要比较learned、全1、同均值uniform和shuffled confidence，并报告校准、
最差场景、harm rate及risk–coverage。没有这些证据时不能称为可靠性。

### 3. 完整单轮与recurrence

只有oracle、producer和可靠性依次通过后，才进行完整条件消融。最后再比较
单轮、两轮和等FLOPs RGB refinement控制；循环不是默认有效前提。
