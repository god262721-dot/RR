# 方法快照

## E0：RGB-only

```text
带反射RGB → RGB恢复主干 → 无反射RGB
```

作用：建立训练和评估基线，不使用偏振。

## E1：Polar→RGB

```text
带反射RGB ─→ RGB编码器 ───────────┐
四角偏振 ─→ Stokes表示 → 偏振编码器 ├→ 瓶颈融合 → RGB恢复
                                  ┘
```

作用：回答“偏振有没有增量信息”。同checkpoint的correct、zero、shuffled
对照表明网络确实依赖正确配对的偏振输入。

E1只是证明“用偏振”有效，还没有回答“怎样使用偏振最好”。

## E2：已失败的双向FiLM实现

```text
RGB瓶颈 ─→ R2P头 ─→ gamma/beta
                         ↓
偏振潜特征 ─────────→ FiLM调制 ─→ 原P2R投影 ─→ RGB恢复
```

问题不是路径没有被调用，而是该路径没有带来稳定任务收益。E2、TP09和TP10
共同否定了当前实现，不是否定所有双向结构。

## TP10：有监督R2P

```text
RGB特征 → R2P → 纠正后偏振潜变量
                    ├→ 偏振decoder → 四角候选GT监督
                    └→ 冻结P2R接口 → RGB恢复
```

偏振decoder能够学习，但同一潜变量同时服务偏振重建和冻结P2R接口，可能
存在目标冲突或分布漂移。

## 当前oracle

当前不预测偏振，只比较偏振信息状态：

```text
固定E1输出T1
   │
   └→ 同结构二阶段任务适配器
          ├→ zero polar
          ├→ observed P0/q/u
          ├→ observed P0 + candidate-GT q/u
          ├→ observed + GT + difference
          └→ full candidate-GT（仅描述）
```

设计重点：

- 同seed不同模式初始化完全一致；
- 参数量一致；
- E1冻结；
- E1始终使用正确观测偏振；
- 干预只发生在新增二阶段接口；
- 主结论不能依赖含干净强度的full-GT组。

## oracle通过后的候选

```text
Pobs → E1 → T1
             ↓
      RGB guide encoder
             ↓
Pobs锚定的显式 Δq/Δu 纠正
             ├→ 偏振重建监督
             └→ 新训练的任务适配器 → 第二阶段RGB恢复
```

这里必须把偏振重建接口和下游任务接口分开，避免再次用一个潜变量承担两个
不同目标。

