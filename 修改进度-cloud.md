# THEMIS 改稿进度（外部顾问侧记录）

**最后更新**：2026-08-16（R2 回执审计完成，R3 骨架待确认）
**论文**：THEMIS，投 USENIX Security '27 Cycle 1
**注册** 2026-08-18 AoE（已锁定）　**提交** 2026-08-25 AoE

---

## 0　新开对话时必须一起提供的文件

**读完本文件后，把下列文件一起发给新助手。缺任何一项都会导致它凭空推断。**

### 必须提供（缺一不可）

| # | 文件 | 从哪来 | 为什么必须 |
|---|---|---|---|
| 1 | `USENIX27_main.tex` | 论文目录，**最新版** | 唯一的正文修改目标。不给它就只能猜正文现在长什么样 |
| 2 | `refs.bib` | 论文目录，最新版 | 45 条 entry，其中 13 条是 R1 新增 |
| 3 | `NDSS意见.txt` | 上一轮投稿的两份审稿意见（9A / 9B） | 所有改稿动作的依据 |
| 4 | `extra_experiment.csv` | 实验目录 | E1/E2/E3/E9/E10 的全部数据，834 行 51 列 |
| 5 | `prompt.md` | 本仓库 | 新对话的开场提示词 |
| 6 | 本文件 | 本仓库 | 进度与已结案清单 |

### 强烈建议提供

| # | 文件 | 为什么 |
|---|---|---|
| 7 | `.helicon/decisions.md` | 已锁定的判定（D-1 至 D-5 等），新助手不读会重开旧争论 |
| 8 | `.helicon/project.yaml` | 章节字数指纹，判断篇幅分布用 |
| 9 | 最近一轮 Codex 回执（含 verification report + checker summary） | 没有回执就无法审计上一轮 |
| 10 | 最新编译的 PDF | 判断版面、图的实际效果 |

### 按需提供

- `Figure/` 目录下的图文件（讨论图的时候）
- HElicon 仓库地址：`https://github.com/SeehowLi/HElicon/tree/v2.0`（助手可自行 clone 读规则）
- 若讨论 softmax 深度或 BTS：`softmax_benckamrk.cpp`、`ppmm-test-211-interleaved-full-216.cpp` 的相关片段

---

## 1　协作链条与角色

```
外部旗舰模型（诊断 + 写指令）
  → 用户（判断、确认、可能提要求）
    → Codex Desktop（本地论文 + 实验数据 + HElicon v2.0）
      → 回执
        → 外部模型（审计）→ 下一轮
```

**外部模型只做两件事**：独立诊断；产出可直接粘贴给 Codex 的修改指令。

**外部模型不产出成品正文。** 真正的 LaTeX 修改由 Codex + HElicon 执行。直接写好段落等于绕过 HElicon 的 claim–evidence 一致性、不可变项保护、术语冻结、断言强度检查。

**每轮外部模型交付三块**：
1. 诊断——这一轮解决什么问题，依据是哪条审稿意见、哪个数字、哪处不一致
2. 可直接粘贴给 Codex 的指令（放代码块里，用户原样转发）
3. 验收判据——Codex 回来后看什么数字才算通过

**每轮回执开头**，Codex 须先用五段短话回答：本轮达成什么 / 暴露什么新问题 / 下一轮重点 / 补做了哪些核实 / 是让文章更好还是只是更快过关。

### 新图或新实验的规格纪律

只要某一轮需要新图或新实验，必须在该轮指令中给出可直接动手的完整规格，并标明：

- **P 类**（零机时）：纯 Python，画图 / 从已有 CSV 取数 / 整理已发表数字
- **M 类**（需 OpenFHE 重跑）：需要真实同态运算

图的规格必须含：位置 / 目的 / 数据源（精确到 CSV 筛选条件）/ 图型 / x 轴 / y 轴 / 系列 / 参考线 / 标注 / 排版 / 字号与配色 / 验收结论。实验的规格必须含"退路"一项（结果与预期相反时论文怎么写）。

---

## 2　论文当前状态

- **标题（已锁定）**：`THEMIS: Level-Budget Co-Design for Encrypted Transformer Inference`
  - 注意：`.tex` 里的 `\title` 目前**仍是旧标题**，尚未写入
- 作者、ORCID、Topic 已完成
- 主稿 `USENIX27_main.tex`，R2 后 2152 行
- 页数：**正文 15 页**、附录 3 页、参考文献 3 页，PDF 共 20 页
  - USENIX 正文限制需确认（一般为 13 页，不含参考文献与附录）
  - **当前阶段不压页**。删减只按内容必要性，不为凑页数
- `refs.bib` 45 条 entry，正文唯一 cite key 40 个，5 条未引用（含模板残留 `example2024`，应删）
- 环境计数：table/table\* 9，figure/figure\* 5，algorithm 5，proposition 2，proof 2，equation 55
  - theorem / lemma / corollary 已全部为 0（R2 删除）

### 故事主线的转换（**尚未写入正文**，是 R3 的目标）

```
现状：PCMM 优化 → CCMM 优化 → Softmax 优化 → 端到端结果（三点平铺）
目标：总分结构
      总：Level-Budget Architecture（架构 + 层级安排）
      分：基于特征差异的 co-design
          矩阵乘  —— 层级少、操作多
          非线性  —— 层级多、每层操作少
          因为特征不同，所以放在不同位置
```

---

## 3　必须保持的数字口径

```
RTE accuracy          72.56% → 71.48%      ← 71.84% 是旧材料错误，永不再用
SST-2 accuracy        92.43% → 92.32%
QNLI accuracy         91.54% → 91.36%
sequence length       128
batch size            256
ring dimension        65536（正文写 2^16）
slots                 32768
scaling factor        50 bits
secret distribution   sparse ternary，Hamming weight h = 192
end-to-end amortized  400.06 s/input
matrix kernels        9.72 s/input，2.4%
nonlinear stages      33.59 s/input，8.4%
BTS（显式三次）        356.75 s/input，89.2%
softmax comparison    1.52 s → 0.59 s（2.57×）
softmax precision     8.84 bits → 13.25 bits
softmax 倒数深度       43 → 21；softmax-path BTS 阶段 3 → 1；389.38 s → 151.59 s
```

**Table V 逐行相加 = 400.06，精确。** 内部完全自洽，可放心让人核。

三项时间分解：9.72 + 33.59 + 356.75 = 400.06 ✓
层级分解：矩阵 6 层，非线性 38 层（entry-to-exit 跨度，见 D-1）

**追加。** `evaluation set`：RTE 277 / SST-2 872 / QNLI 5463（完整 GLUE dev 集）。Table IV 每个数值除以对应规模均得整数正确数。

**冻结数字的计数口径更新。** 使用带边界正则 `(?<![0-9.])N(?![0-9.])`。R2/R3 的裸 `grep` 已失效：546 会匹配 5463、0.71 会匹配 90.71、43 会匹配 92.43。因此基线自 R4 重设，与前几轮不可比；另对 12 个核心数字做位置核对（`\label` + 上下文），不只统计总量。

---

## 4　已锁定判定（`.helicon/decisions.md` 同步）

- **D-1**　Table V 的 `17→3` 是**算子级黑盒口径**：只记入口与出口层级，**不表示内部只消耗 14 层**。不得据此做深度装载论证
- **D-2**　非 L2 倒数深度 = 16，L2 = 21，exp = 4，THOR-style = 43。**「43 > 16 装不下」永久禁用**（量纲不一致）
- **D-3**　softmax 闭合论证的基座是：倒数深度 43→21 ⟹ softmax-path BTS 阶段 3→1 ⟹ 389.38→151.59 s。三段量纲一致、逐段实测。R3 / R6 继承
- **D-4**　"只需刷新分母"这条 observation 归功 `softmax24`（Cho 等，THOR 所基于的工作），**不作为 THEMIS 贡献**；交错多批次打包可作为实现布局描述，但不再作为贡献主张
- **D-5**　`homomorphic-free`、`homomorphic evaluation-free`、`FHE-free`、`key-switch-free` 及任何自创的 `X-free` 复合词全部永久禁用
- **D-6**　全文统一术语 `coefficient-domain execution`，与实现中的 `EvalCoeffDomainLinearPCMM` 一致
- **D-7**　PCMM 的核心主张是“从系数层进入使同一批算术成为稠密 GEMM”。这一观察归功 BLAS 线；THEMIS 的增量是在槽编码 $L \ll N$ 区间使其成立
- **D-8**　C2PMM 相消**必须有证明**：正文简版（≤12 行，含两个恒等式），附录完整版（60–80 行）。9A 的槽侧直觉不足以推出 THEMIS 所需的系数侧 $2N\times2N$ 提升矩阵相消；共轭分支归零不是 FFT/IFFT 互补的直接推论。**这推翻 R2 的全删**
- **D-9**　贡献层级（H-POSITION 锁定）：顶层 level-budget co-design；A1 `coefficient-domain execution`；A2 ColPair/DiagPair（非同一 trick）；A3 denominator-conditioned softmax；S 端到端系统
- **D-10**　已废除的主张：`key-switch-free`、交错打包、相消性质本身、complex-number trick、只刷新分母的 observation
- **D-11**　PCMM 表的锚点是**一次完整 Q 投影**，不是六个投影合计。数字来自 `PCMM_benchmark.cpp` 的循环边界静态展开，不是动态执行。R3-V11 从论文抠的多投影总数（Powerformer 464、THOR 922）**不能与之直接换算**
- **D-12**　level cost 的统一规则：在线路径上串行的明密文乘法层数。加密时 3 个模数、算完剩 1 个 = 消耗 2 层；2 个模数 = 1 层。五行必须同规则推导，不得出现“作者裁定”的单格
- **D-13**　不比较 NEXUS 的 accuracy 与密文矩阵乘。理由：第一代工作，性能差两三个数量级，Euston 已与其对比。Introduction 保留引用，Evaluation 开头加一句比较范围说明
- **D-14**　对比表一律用**每输入**口径为主列，整批数进表注。整批数与单输入数不得并排出现在同一列
- **D-15**　正文页数上限 **13 页**（USENIX Security '27 Cycle 1）。R4 后正文 15 页；减页只做合并与移动，不删机制解释
- **D-16**　Contributions 用总分 + **洞察式**：厉害感来自读者没想到的观察，不来自形容词。禁止 `novel` / `significant` / `first` / `state-of-the-art` 等
- **D-17**　bootstrapping 不做优化讨论。本文做的是体系结构优化，bootstrapping 作为可替换模块，任何改进都能接入；且本文的 S2C/C2S 是复数结构、分解为多个矩阵乘，强行融合可能增加计算。降为“未来方向”，正文有空间才写，放 R7
- **D-18**　`fig:col_diag_pair` 保持跨栏；压单栏的只有 RNS 合并图
- **D-19**　参考文献检查改**单向**：正文 cite key ⊆ bib entry。未引用的 bib 条目保留，不再报告

### 禁用表述

```
optimal / first / practical / irreducible / lower bound / inherent（作为 THEMIS 的无限定主张）
concurrent / parallel work / contemporaneous（用于 Liu–Zhang）
building on / following / based on the algorithm of（作为 §IV-A 段落主干）
把 BTS 89.2% 写成不可约下界或固有开销
把「3 BTS → 1 BTS」写成整个 Transformer block 只有一次 BTS
把 inverse depth（43 / 21）与 available levels（16）相减或比较
71.84（正确值 71.48）
"128-bit security" 或任何等价保证性表述
把 28.4 小时写成实测（它是 400.06 × 256 的推算）
把 E10 的 9.7% 写成端到端实测（它是 model-derived counterfactual）
throughput advantage / keep the backend busy / leading dimension close to N（已被 E3 否定）
nohalf（已清零）
homomorphic-free / homomorphic evaluation-free / FHE-free 及任何自创的 X-free 复合词
key-switch-free（永久废弃）
  → V11/V14 已证 Euston / MOAI / NEXUS 的在线 PCMM 同样零 key-switch，且明密文乘法本来就不需要 key switching；这不是 THEMIS 的性质
Kernel speedup（作为表列名）
  → 用 Dense GEMM backend（事实而非结果）
novel / significant / substantially / first / for the first time / unprecedented / state-of-the-art
任何暴露审稿过程的措辞（双盲投稿）
no CKKS operation / no homomorphic operation（绝对表述）
节标题与小节标题中的冒号；"Co-design I/II" 式标题
  → 总分关系放在开篇句里，不放在标题里
```

---

## 5　**已结案，不要重开**（新助手重点读这一节）

以下问题在前几轮已经彻底讨论清楚。**不要再当成新发现提出来。**

| 议题 | 结论 |
|---|---|
| **端到端 400.06 s** | 完整同态实测，数据真实。Table V 逐行相加精确等于 400.06。**不要再质疑** |
| **softmax benchmark 里的 exp 解密重加密** | 只存在于 micro-benchmark，目的是刷新层级，**不影响精度**（exp 精度本身高于最终 softmax 精度若干比特）。端到端路径中 exp 走完整同态。Table III 两行同等排除 exp，比较公平且已在正文写明 |
| **Table V 的 `17→3`** | 算子级黑盒口径。中间可能有多次自举刷新层级，论文不展开，被问到能解释清楚。见 D-1 |
| **L2 那次额外的 bootstrap** | 在端到端路径内，成本已摊入 Table V |
| **softmax 内部的自举** | 只作用于**分母**，不作用于分数矩阵。分母每行一个值，数据量远小于分子，所以整个 Softmax 行只有 1.25 s/层/输入，而算子边界 BTS 是 15.12 s/层/输入。两者差约 12 倍，**本来就不是同一种东西** |
| **89.2% 的口径** | 356.75 = 181.40 + 90.70 + 84.65，只含**显式列出的三次** BTS，不含 softmax 内部分母刷新。所以 89.2% 是**下界**，正文已注明。这对我们有利 |
| **sparse ternary（h=192）** | 用户已确认这是实际配置且合规。**采用 A 档写法：只报参数，不援引标准表、不给比特数、不写 logQP / dnum**。详细配置进附录或只在代码中体现 |
| **Liu–Zhang 的关系** | 他们**在先**（TSC 16(3), 2023；PCMM24 §1.1 明写 "Our starting point is the algorithm from [26]"）。**不能写成 concurrent / parallel work**，查日期五秒就翻。正确处理见第 6 节 |
| **投稿去向** | 只投 USENIX Security '27 Cycle 1，不考虑 S&P 或其他退路。不要再讨论 |
| **artifact** | 代码已上传。不要再提醒 |
| **页数** | 正文上限 13 页；R4 后为 15 页。R5 只通过合并与移动浮动体减页，不删机制解释 |
| **rescale** | PCMM 每次一次，用于 scale 与噪声管理。**必须在正文主动写出**，因为 Table V 的 3→2 摆在那里。写了账就清楚，不写会被追问 |
| **E8 运行次数** | 不写。跑一轮几十小时，同领域也不写。改为写明**完整 dev 集规模**（277/872/5463），这比运行次数更有说服力，且顺带解释了 400.06×5463 的算式 |
| **PCMM 数字来源** | 一律从 `PCMM_benchmark.cpp` 实测，五个对比方案全部有复现。论文抠出的数字只用于交叉核对，冲突以代码为准 |
| **层级消耗** | 必须逐方案实测。Euston 可能是 2、Powerformer 可能不是 1；不要假设全为 1 |
| **Fig 4** | 保留四 panel，micro-benchmark 的速度对比有必要 |
| **CCMM 切法** | 结论 + observation + 思路详解 + **算法**留正文；只有逐步代数推导进附录 |
| **Powerformer = model-modified** | §3.1 用 BPMax 替换 Softmax，§3.3 蒸馏重训。accuracy 表标 ✗；中性陈述事实，不写“其结果不作数” |
| **C2PMM 证明的分层** | 正文 12 行（proposition + 简证 + 两个恒等式），附录 80 行完整推导。R2 的全删已被推翻，不要再删 |
| **§IV 的页数瓶颈在浮动体不在文字** | 四个跨栏 float + 两个算法。再砍文字无效，必须合并 float |
| **Euston vs MOAI 的差异来源** | 唯一区别是交错打包。MOAI 有，因此 batch 内旋转免费 → rotation = 0 且少一层深度；Euston 没有，必须用 mask 乘法实现 batch 内旋转，每次 mask 乘法配套一次 rotation。**Euston 的 PT×CT / rotation / 加法三项必然多于 MOAI** |
| **R4 Euston 行是错的** | 三项与 MOAI 相同却差 1 层深度，内部矛盾。根因是查代码停在 `RunStreamingCtPtMatrixMul` 封装层 |
| **代码查法** | 论文优先定基准，然后**必须拆到 `hec.rot` / `hec.mult` / `hec.add` 调用点**，不能只看表层封装 |
| **NEXUS** | 不比较。Evaluation 开头一句比较范围说明，不点名不贬低 |

---

## 6　Liu–Zhang 与 PCMM 写法（关键，容易写错）

### 6.1 归功的四层顺序（顺序本身就是内容）

段落顺序固定，**归功句的主语必须是那个算法或那条脉络，不得是 THEMIS**。THEMIS 不得作为 follow / adopt / build on / start from / inherit 的主语。归功占一句，不扩成一段，不放在段落第 1、2 句。

1. **问题**：每层六个 PCMM 位置；现有加密 Transformer 系统的在线 PCMM 依赖同态算子
2. **THEMIS 的答案**：矩阵乘本身从不以同态方式求值
3. **算法出处（一句）**：该归约的算法起点见于 Liu–Zhang（在其可验证外包方案中作为部件），由 PCMM24 / BLAS25 发展为 FHE 的 PC-MM 形式，后者在 $d \ge N$ 下建立槽编码兼容性
4. **THEMIS 的算法增量**：槽编码 + $L \ll N$ 区间的实例化；不需要 PCMM24 在 $d<N$ 时所需的 MLWE ModDecomp/ModPack 路径
5. **THEMIS 的系统增量**：将该 execution 置入端到端加密 Transformer pipeline，并与下游 CCMM 衔接；交错打包不再作为贡献主张

**不写"独立得到"脚注。** 那是防御姿态，会把读者注意力引向相似性。

### 6.2 coefficient-domain execution 的正确形态

**术语冻结。** 全文统一使用 `coefficient-domain execution`，与实现中的 `EvalCoeffDomainLinearPCMM` 一致。`homomorphic-free`、`homomorphic evaluation-free`、`FHE-free`、`key-switch-free` 及任何自创的 `X-free` 复合词均永久禁用。

**核心主张。** PCMM 从系数层进入，使同一批算术成为稠密 GEMM；这一观察归功 BLAS 线。THEMIS 的增量是在槽编码 $L \ll N$ 区间使其成立，而不是宣称某种 key-switch 性质。

**正文写一句精确陈述**：PCMM 通过 `coefficient-domain execution` 对密文相关的系数矩阵执行稠密整数线性代数；路径上每次 PCMM 有一次 rescale。rescale 是同一批系数矩阵的逐系数整数运算，用于 scale 与噪声管理，不参与矩阵乘。

**主动把 rescale 说出来。** Table V 第一行是 `Q,K PCMM 3→2`，级别掉了一层。不说，读者会问“这一层怎么掉的”；说了，账就清楚。

**用 V14 的代码实测让数字说话。** 六方案统一统计 CT add / PT×CT / rotation / 层级消耗 / 是否调 Dense GEMM backend；不要把 `Kernel speedup` 当成表列名，也不从论文抠数代替实测。

---

## 7　实验状态

### 已跑完，数据在 `extra_experiment.csv`

| ID | 内容 | 关键结论 |
|---|---|---|
| E1 | CCMM 分阶段消融（244 行） | **消融非单调**。COLPAIR 单独只降 1.5% rotation，价值在与 BSGS 组合时拓宽可分解基数（64→128，1.33×）。如实呈现非单调比每行都加速更可信 |
| E2 | Key-switching 在线代价常数（74 行） | hoisting **收益有条件**：单次 rotation 收益很小或为负，复用两次以上才 break-even。BSGS 下 baby-step 被复用 G 次，所以此处成立 |
| E3 | 批大小对 PCMM 的影响（163 行） | 整批时间约 44.5 s，B=1/64/256 波动仅 0.71%；B=1 每输入是 B=256 的 254.27×。**这是 slot amortization，不是 kernel 变快**。Codex 已代码核实：dgemm 矩阵形状由环维度决定，不随 B 变化 |
| E9 | RNS 原语层级伸缩（198 行） | 七个层级点：paper level 1/2/4/8/12/16/17，towers 2/3/5/9/13/17/18。L17/L1：ROTFAST 8.228×、Rot 12.401×、RELIN 13.672× |
| E10 | 层级放置反事实（154 行） | 矩阵成本 9.720 s（2.43%）→ 反事实高层 38.770 s（9.69%），3.989×。**是 model-derived counterfactual，不是端到端实测** |

**运行条件**（五者一致）：第二颗 CPU、NUMA node 1、104 硬件线程、1 次 warmup、4 次正式取算术平均、correctness = PASS。`run_id` 形如 `*_20260812_NODE1_4MEAN`。

**取数纪律**：正文数字一律取 `record_type = summary` 或 `derived`；`raw` 只用于复核；`assumption` 类的衍生结果必须在正文写明是 model-derived。

### E9 的两个易错点

**基准会翻转结论。** 以 Level 1（2 个模数）为分母，线性预期 = towers ÷ 2 = 9.00×，而 ROTFAST 只有 8.228×，**低于**线性预期；以 Level 2 为分母，线性预期 6.00×，四项全部超过。**同一批数据换基准得出相反结论。** 已裁定用 Level 1，caption 必须写明基准。

**正确的结论表述**：代价至少随 limb 数线性增长；**带 key-switching 的原语在 L12 以上转为超线性**（Rot 1.38×、RELIN 1.52×），**hoisted core 保持近似线性**（0.91×）。

**不依赖任何基准的白拿结论**：Rot ÷ ROTFAST 从 1.287 升到约 1.94–2.00，RELIN ÷ ROTFAST 从 1.241 升到约 2.06–2.13。即 **hoisting 可回收的余量随层级从约 1.29× 涨到约 2×**。注意中间不单调，只能写"从约 1.29 升到约 2.0"。

**低层偏快的成因**：2–3 个 limb 时 NTT 工作量喂不满 104 线程。只能写成"我们将其归因于……"，**不得写成实测结论**（没测线程利用率）。

**PT×CT 在最后两点非单调**（L16 归一 15.899 → L17 12.990），原因经核实**查不出**，已排除 dnum 分组边界与测量伪影两种解释。**不得为它编造机制**，不入图。

### R4 实测结果

**V14**（`PCMM_benchmark.cpp` 静态计数，单 Q 投影，整批；Euston 行为错误的 R4 旧值，R5 的 V16 重查）：

| 系统 | b | CT add | PT×CT | Rot | Level | Dense GEMM backend | 状态 |
|---|---:|---:|---:|---:|---:|---|---|
| Powerformer | 1 | 1,535 | 1,536 | 85 | 1 | — | 可用 |
| THOR | 1 | 1,556 | 1,580 | 40 | 2 | — | R5 V16 重查 |
| Euston | 256 | 589,056（R4 旧值） | 589,824（R4 旧值） | 0（R4 旧值） | 2 | — | **错误；R5 V16 重查** |
| MOAI | 256 | 589,056 | 589,824 | 0 | 1 | — | 可用 |
| THEMIS | 256 | 0 | 0 | 0 | 1 | ✓ | 可用 |

**根因与 R5 待决项**：Euston 与 MOAI 共用 `RunStreamingCtPtMatrixMul`；文件里定义的 `RunMoaiCtPtMatrixMul` 未被调用；Euston 的 Q 按论文应走 masked IWMM，而非列打包。R5 必须拆到 `hec.rot` / `hec.mult` / `hec.add` 调用点，不能停在封装层。

**V15**：`论文MD/` 已建库 106 页、五篇；R3-V12 的 accuracy 数字复核零差异。

### 待办实验（判定）

| ID | 内容 | 类别 | 判定 |
|---|---|---|---|
| **E6** | 跨系统 end-task accuracy：各系统各自的 plaintext baseline 与 Δ（MOAI / THOR / Euston） | P，零机时 | **必做**。不能只比加密后的绝对值；不同系统微调起点不同。9B #4 点名 |
| **E8** | 完整 dev 集规模说明（RTE 277 / SST-2 872 / QNLI 5463） | P，零机时 | **不做**。accuracy 小节写明完整 dev 集规模；不写运行次数（单轮几十小时，同领域也不写） |
| **V11** | 五系统在线 PCMM 的 rotation / key-switch / pt-ct / ct-add 计数 | P，零机时（与 E6 同批文献） | **必做**。只报告可核的计数，不把零 key-switch 当作 THEMIS 性质 |
| **V14** | PCMM 五方案五项代码实测（CT add / PT×CT / rotation / 层级消耗 / 是否调 Dense GEMM backend） | P，零机时 | R4 完成；Euston 行为错误旧值，R5 V16 重查；其余四行可用 |
| **V15** | 五篇论文入 `论文MD/` + Powerformer `model-modified` 核实 | P，零机时 | ✅ **R4 完成**。106 页；R3-V12 accuracy 数字复核零差异 |
| **V16** | Euston/THOR PCMM 底层调用重查 | P，零机时 | **R5 必做**。论文优先定基准，再拆到 `hec.rot` / `hec.mult` / `hec.add` 调用点 |
| **E5** | 标定成本（样本数、耗时、是否明文、微调后是否重做） | **P，零机时**（原列 M） | **做**。9B #2 点名。不是新实验，是把现有标定脚本跑一次记时 |
| E13 | $\alpha(D_t)$ 经验范围验证 | P | **待查**。只在附录确实有未支撑的经验断言时做 |
| E4 | softmax 消融 | M | **不做**。Table III 已是五维两点对比，占机器换边际收益 |
| E11 | softmax 深度账表 | P | **不做，且有害**。会重新打开 21/16/14 的混淆，闭合链已在 Table III |
| E7 | 部署侧指标 | M | 暂缓 |

---

## 8　审稿意见与闭合状态

```
9A:  Reject       expertise = Knowledgeable   confidence = High    ← 权重远高于 9B
9B:  Weak reject  expertise = No familiarity  confidence = Medium
```

### 9A 六条

| # | 意见 | 动作 | 状态 |
|---|---|---|---|
| 1 | novelty 不清楚 | 主线换成 level-budget 架构；精确划分继承与新增 | ✅ R4 |
| 2 | PCMM/CCMM prior work 讨论不足 | 补 13 篇 + Liu–Zhang 归功 + Related Work 谱系段 | ✅ R1–R2 |
| 3 | slot-encode C2PMM 论证过长，cancellation 可直接解释 | 恢复分层证明：正文≤12行（两个恒等式）+ 附录60–80行 | ✅ R4（D-8 推翻 R2 全删） |
| 4 | bootstrapping 当黑盒 | 不做优化讨论；如正文有空间，仅作为可替换模块的未来方向，说明强行融合在复数 S2C/C2S 结构中可能增加计算 | 待做，R7 |
| 5 | artifact 只有 README | 上传可运行代码 | ✅ 已上传 |
| 6 | speedup 来源不清楚，缺逐组件 ablation | E1 非单调消融 + E2 代价常数 | 待做，R6 |

**9A 明确认可的三点，改稿中不要削弱**：性能数字较强 · final-only high-precision reciprocal 的 softmax 设计有意思 · 已有 microbenchmark 与 end-to-end evaluation。

### 9B 四条

| # | 意见 | 动作 | 状态 |
|---|---|---|---|
| 1 | 只评估单一配置 | E3 就是配置扫描，成本模型可预测；BERT-large / decoder-only 写入 Limitations | 部分，R6 |
| 2 | calibration 的数据需求、时间、稳定性不清楚 | E5（P 类）+ 正文 3 句 + 附录 | 待做，R8 |
| 3 | accuracy 运行次数不清楚 | 写明完整 dev 集规模（277 / 872 / 5463），不写运行次数 | ✅ R4 |
| 4 | 缺与 prior systems 的 end-task accuracy 对比 | E6 | 待做，R6 |

---

## 9　轮次进度

| 轮 | 内容 | 状态 |
|---|---|---|
| **R1** | 口径裁决写入 + 归功与文献修复 + 安全参数核实 + 图件 | ✅ 完成（E11 图件被门禁阻塞） |
| **R2** | 删定理体系 + PCMM 按 E3 真实结论改写 + 两类 BTS 口径分开 + 死引用安置 | ✅ 完成（C2PMM 证明的全删由 D-8 推翻，R4 分层恢复；T7 停做、T9 失败） |
| **R3** | **§III 改名 Level-Budget Architecture + 两轴刻画（矩阵 6 层浅而密 / 非线性 38 层深而疏）+ 放置规则 + F1-a/F1-b + E10 + PCMM 表框架 + coefficient-domain execution 改写 + 安全参数 A 档 + dev 集规模 + E6 + Fig 1/Fig 3 overlay** | 定稿 |
| **R4** | **PCMM 表实测化 + coefficient-domain execution + C2PMM 两层证明 + CCMM 下沉（ss:ccmm −41.9%）+ F1 合图 + accuracy 表 Original model 列 + 论文MD 基建 + H-POSITION** | ✅ 完成 |
| **R5** | 标题写入 + Contributions 洞察式重写 + Introduction origin story + V16 Euston/THOR 底层重查 + PCMM/CCMM 两表合并 + 减页主刀 + accuracy 表删 MRPC + NEXUS 比较范围句 | 待执行 |
| **R6** | Evaluation 消融（9A #6：E1 非单调 + E2 代价常数） | 待 R5 |
| **R7** | Discussion & Limitations（+ bootstrapping 未来方向，有空间才写） | 待 R6 |
| **R8** | Softmax 章节（D-3 闭合链 + E5 标定成本，9B #2） | 待 R7 |
| **R9** | Abstract | 待 R8 |
| **R10** | 压页收尾 + 数字对账 + 匿名化 + 灰度检查 | 待 R9 |

时间不够时的牺牲顺序：R8 扩写 → R7 的 Limitations → R5 的 Introduction。**9A #4 与 #6 一条都不能砍。**

### R3 章节骨架

**标题一律短名词短语，无冒号。** 总分关系放在开篇句里，不放在标题里。

- III Level-Budget Architecture
  - A Threat Model
  - B Block-Level Execution Pipeline
  - C Level Cost Model
  - D Operator Placement
- IV Matrix Multiplication（标题不动）
- V Denominator-Conditioned Softmax（标题不动）

### R1 完成明细

Threat Model 绝对化表述收窄；四条出处改为正式出版信息（PCMM24 → CRYPTO 2024 LNCS 14922 pp.387–421；BLAS25 → Journal of Cryptology 2026 DOI 10.1007/s00145-026-09580-x；Euston → IEEE S&P 2026 DOI 10.1109/SP63933.2026.00048；CCMM25 → EUROCRYPT 2025 LNCS 15608）；新增 13 条引用；Related Work 补矩阵乘谱系（三条支线）；Liu–Zhang 归功写入；complex-number trick 降级为工程优化并归功 Sav 等 Patterns 2022 / slytHErin / nGraph-HE2；Euston 继承关系精确划分；`nohalf` 清零；Table II/III/V 三处口径修补；Fig 5 的 546× 降级为 up to；建立 `.helicon` project pack。

### R2 完成明细

删 theorem 1 + lemma 2 + Proof sketch（正文 93→10 行）+ 附录 `app:pcmm-slot-proof` 整节 194 行 + `lem:pcmm-triv` + `cor:pcmm-compose`，三个 `\newtheorem` 声明删除，六个 label 的 `\ref` 残留全为 0；PCMM 定位重写；"前导维度/后端欠载"三处全改（`leading dimension` / `backend busy` / `recovers the throughput` / `throughput advantage` / `feed the backend` 结束稿计数全 0）；七条死引用安置，cite key 33→40；Table III 的 #BTS 语义表注；89.2% 口径注明为下界；D-1 至 D-5 写入 decisions ledger；F1-a / F1-b / T-E3 生成未插入。

### 已闭合的核实（不要重跑）

- **V8**：logQ = 1559，logP = 180，logQP = 1739，Q 31 limb（59,50,51,…），P 3 limb（60,60,60）。`stdlatticeparms.cpp:187` 的 `HEStd_ternary / N=65536 / 128-classic` 上界为 1747。OpenFHE 在 HYBRID 下比较的是 log(Q·P)，见 `ckksrns-parametergeneration.cpp:221–230`。**但按 A 档写法，这些数字都不进正文**
- **V9 组三**：THOR / Euston / MOAI / NEXUS / Powerformer 五个系统的在线 PCMM **全部**调用密文算子。**待补**：其中哪几个用了 rotation / key switching（Euston 目前只查到一次同态加法）
- **V10**：Table III 的 `#BTS` 统计的是 softmax 路径上的**分母刷新阶段数**（THOR 3、THEMIS 1），全尺寸算子边界 BTS 在该 benchmark 中为 0，不计入该列
- **T3 前置**：dgemm 矩阵形状不随 B 变化，`cipherRows` 由环维度导出（`homoencrypt-compute-matmul.cpp:3547-3548,3616,3672-3721`）

---

## 10　HElicon v2.0 使用要点

仓库：`https://github.com/SeehowLi/HElicon/tree/v2.0`　本地已安装并建好 paper-local `.helicon` pack。

### 最重要的一条：P1/P2 不适用 P3–P7 的机械闸门

`scripts/check_pass_scope.py` 的 `PASSES = ("P3","P4","P5","P6","P7")`，`pass_pipeline.md` 的机械核验契约开头就是 "For every rewriting pass **from P3 through P7**"。**P1（claim scope）和 P2（structure）不在这个契约里，脚本连参数都传不进去。**

R1 和 R2 做的全是 P1/P2 的活，却被拿 P3–P7 的闸门去卡，于是出现 `check_immutable_set = 37`（R1）和 `213`（R2）的"失败"。**这是流程用错，不是编辑出错。**

### P1/P2 的正确机制：事前授权账

Iron Rule 1 原文写着 "outside an explicit **P1 request**"；`pass_pipeline.md` 写着 "Any **unapproved** immutable-set difference is a failed pass"。措辞都是"未授权"，不是"任何"。所以：

1. 开工前在 `decisions.md` 写授权记录：本轮 pass 类型、授权改动的类别、每个类别的具体 `\label`、理由
2. `check_immutable_set` 全量照跑，角色是**证据**不是闸门；每一项差异必须映射到授权账
3. 映射不上的才是违规，回滚
4. 硬闸门收窄到本轮**没有**授权的类别

`check_claim_strength` 有对应条款：*During P1 only, an author-approved claim-scope adjustment may exempt an upward move from rollback, but every exempted move must be listed in the trailer.*

### 必须做的 baseline 管理

`check_immutable_set(before, after)` 没有 baseline 管理，BEFORE 传什么就跟什么比。**每一轮授权结构性删除之后必须重设基线**，否则删除会被反复计入，213 会滚成 300，闸门失效，真正的违规（改了带单位的数字）反而看不见。

### 尚未启用但应该用的入口

| 入口 | 用途 |
|---|---|
| `H-INTAKE` → `draft_map.md` | 给每节打 STABLE / REWORK / DELETE-CANDIDATE。重构轮开工前必跑，避免润色即将被移动的段落 |
| `H-POSITION` | 锁 story / venue / contribution hierarchy。总分结构是 positioning 决策，应在写作 pass 之前 |
| `templates/reviewer_risk_log.md` | 9A 六条 + 9B 四条各指定 owner（哪一轮、哪个 label 闭合） |
| `templates/claim_ledger.md` + `evidence_matrix.csv` | claim–evidence 一致性，目前是手工维护 |
| `scripts/build_target_profile.py` + `H-STYLE` | **P4 节奏 / P6 风格从未真正执行**，因为缺 style profile。一次性零成本，解锁 R8/R9 |
| `H-EXPORT mode=advice` / `H-INGEST` | `external_advisor_protocol.md` 就是为外部顾问角色设计的通道 |
| `H-GATE` + `submission_gate_checklist.md` | R9 的投稿前审计 |

### 已知能力边界

| | |
|---|---|
| 不破坏不可变项 | ✅ |
| 术语一致性归一 | ✅ |
| 去 AI 痕迹 | ✅ report-only |
| 对本已合格文本做改进 | ❌ 正确行为是拒绝改动 |
| P4 节奏 / P6 风格 | ❌ 需 style profile |
| **数学区外的裸数字** | ❌ **不受保护** |

**最后一条对本论文影响很大**——关键数字大量是裸数字。**每轮指令中必须显式列出该轮不得变动的数字，并要求 Codex 逐个 grep 核对。**

**注意计数口径**：R1 用带 lookaround 的正则，R2 用裸字符串 grep，两者结果不同（如 `43` 一个是 2 一个是 9），**跨轮不可比**。应固定一种口径。

### 词表硬规则

`Term` 列填完整规范形式（规范形式是 `THOR-style` 就写 `THOR-style`，写 `THOR` 会导致 P3 删掉修饰词）；第 2 列留空；第 3 列用分号分隔；Notes 列不用反引号；纯大小写变体不进 Avoid；构建期带 `--fail-on-deletion-risk`。

---

## 11　图的规范

**两条硬要求**：

1. **元素零重叠**——数字标注不得与曲线、图例重叠。线是线、字是字、图例是图例，清晰可辨
2. **风格全文一致**——新图的字体族、字号、线宽、配色、网格样式、图例位置必须与既有图对齐。做法是先从 `Figure/themis_four_panel_summary.pdf` 提取样式参数再套用，不要各画各的

**灰度可分**：USENIX 常被灰度打印。不同线型（实线/虚线/点划线）+ 不同标记形状，不得只靠颜色。

### 当前图状态

| 图 | 文件 | 状态 |
|---|---|---|
| Fig 1 | `Figure/themis_architecture.pdf` | `FNN`→`FFN` 的 TikZ overlay 已完成；像素差分零外溢。原 PDF text layer 的旧字不作为验收依据 |
| Fig 2 | `Figure/pcmm2ppmm_embedded.pdf` | 正常 |
| Fig 3 | `Figure/CCMM.pdf` | $p_{33}$→$s_{33}$ 的 TikZ overlay 已完成；像素差分零外溢。原 PDF text layer 的旧字不作为验收依据 |
| Fig 4 | `Figure/themis_four_panel_summary.pdf` | 保留四 panel；micro-benchmark 的速度对比有必要。**样式基准图** |
| Fig 5 | `Figure/offset_d0_range_reduction_by_layer.pdf` | 二 panel，后期可压一 panel |
| F1-a / F1-b（RNS 合并图） | 待入正文 | 改单栏、约 2:1 压扁、两 subfigure 左右并排；真实 RGB 配色 + 灰度可分；使用 subcaption 真子标签 |

**注意**：`.tex` 中 `FNN` 计数已为 0，FFN 问题**只存在于图内文字**。

### Fig 1 与 Fig 3 overlay 验收

Fig 1 / Fig 3 的 **TikZ overlay 已完成**：在 `\includegraphics` 之上叠白色矩形盖掉错字，再放正确文字；像素差分零外溢，原图其余部分逐像素不变。原 PDF 的 text layer 仍含旧字，验收依据是最终编译页面。

RNS 合并图改为**单栏**、约 2:1 压扁、两 subfigure 左右并排；**必须使用与正文一致的真实 RGB 配色**（不接受灰阶近似），同时保持灰度可分（线型 + marker）。

`fig:pcmm2ppmm` 从三 panel 改为两 panel（(a)/(b) 合并）；`fig:col_diag_pair` 保持跨栏不动。

### 合并图标签

合并图必须改用 subcaption 真子标签，使 (a)/(b) 可点对点引用。正文画了 panel，就必须有点对点的正确引用；不得再让多个 label 仅指向同一个 float。

---

## 12　用户沟通偏好

- **中文回复，技术术语保留英文原文**
- **冷静的因果推理，不要铺垫、不要客套、不要 hedging**，先给结论再给依据
- **自主执行，不要反复请示**，能自己查的自己查
- **主动找盲点**——价值最高的输出都来自主动审计
- **敢于纠正自己和对方**
- **禁用词**：语境、范畴、范式、落地、沉淀、抓手、本质上、某种程度上、在…层面、本质
- 说「大白话」时切换到零术语表达
- 用户会推「尽快」，可以加速，但**不能靠省掉验证来加速**
- **不要增加太复杂的实验**；列实验时必须写清必要性与在文中的作用，由用户判断做不做
- 每轮结束报一次离 8/25 的剩余天数

### 判断某个问题该不该提的分类

论文本来就不必解释每个细节。有些东西是刻意规避的，这是正当的。判据是**审稿人能不能自己验出来**：

| 类 | 判据 | 动作 |
|---|---|---|
| **A** | 规避了也验不出来，或验出来也不构成问题 | **不提** |
| **B** | 规避没问题，但被问到需要一句话答复 | 只提"补一句防线"，不改结构、不改数字 |
| **C** | artifact 里能 grep 到，或两步算术能推翻 | 必须提，且坚持 |

**分界线是可验证性。** artifact 已上传，C 类的范围比投稿前大了——这是唯一改变了性质的变量。

---

## 13　纪律

- 论文源码、系统名、实验数据**绝不进入 HElicon 公开仓库**（铁律 7，有污染门禁）
- 论文目录建议本地 `git init` 但**绝不 `git remote add`**
- 每轮指令中：源文件保护（开始与结束各算一次 SHA-256）、输出只给计数与布尔值、新增/重写才给正文
- 本进度仓库是**公开**的：只记录方法、判定、口径与流程，**不要贴论文正文段落、不要贴实验原始数据、不要贴代码路径的完整绝对路径**

---

## 14　一句话状态

标题、作者、topic 已锁定；R1（口径与归功）、R2（PCMM 真实机制 + BTS 口径）、R4（PCMM 表实测化、`coefficient-domain execution`、C2PMM 分层证明、CCMM 下沉与论文MD 基建）已完成。R5 的主刀是 §IV 浮动体合并与 V16 底层重查；E4/E8/E11 判定为不做，E5 降为零机时，V14 的 Euston 行已作废待 V16、V15 已完成，E6 必做；`key-switch-free` 与其他 `X-free` 造词均永久禁用；安全参数按 A 档只报基本配置。

---

## 15　Codex 执行效率

每轮指令必须包含：

1. **复用清单**：上轮已产出、禁止重新生成的文件，逐个给出路径。
2. **不要做清单**：不重查已结案事项、不做全库模糊搜索、不重建词表、不重跑 P3–P7 闸门（P1/P2 不适用）。
3. **定位表**：每个任务给 `\label` + 当前行区间，不让它搜。
4. **批量化**：读取同一批外部文献的任务合并成一次取回，PDF 保留不删。
5. **计数口径固定**：沿用同一脚本，上轮的 after 快照直接作为本轮 before。

---

## 16　多智能体与 PDF 处理

### 多智能体拓扑（每轮指令必须给）

- 主 agent 编排 + 汇总，**只有它写回执**。
- 阶段 1 并行：A（文献转换取数，可再分子 agent）／B（代码统计）。
- 阶段 2：C（HElicon P1/P2 编辑；不依赖 A/B 的任务可提前开始）。
- 全程：D（核验；C 每交付一项立刻核一项，不攒到最后）。
- 写权互斥：只有 C 能写主稿。

### PDF 硬规则（永久）

1. 任何 PDF 读取前必须先过 `pdf2md`，之后只读 MD；原 PDF 只留档。
2. 统一存 `<论文目录>/论文MD/`，命名为 `<系统名>_<年份>.md`。
3. 配 manifest：来源 URL / 原 PDF SHA-256 / 转换日期 / 覆盖小节 / 是否完整。
4. 开工先查目录；已有不重转。
---

## 17　§IV 减页方案

瓶颈是六个浮动体，不是文字。R4 砍了 41.9% 词数但页数零变化。

- **主刀**：`tab:pcmm-online-cost` 与 `tab:ccmm-summary` 合并为一张三段式表（PCMM(Q) / QK^T / S·V，统一列结构），省约 0.4 页。
- **其余**：RNS 合并图改为单栏、约 2:1、左右两个 subfigure；`fig:pcmm2ppmm` 三 panel → 两 panel；`fig:col_diag_pair` 保持跨栏；`ss:pcmm-algo` 的 Eq.4/5/6 下沉附录，只留结论。

全部是合并与移动，不删任何机制解释。