# RDL Function と局所 M_B

*RDL Functions / T2 設計ノート / v0.1*  
*位置づけ：RDL応用案 / 関数モジュール設計 / 暫定文書*

---

## 0. 位置づけ

本書は、RDLにおける「関数（Function）」を、単なる外部計算道具としてではなく、**有限な関係拘束を演算可能な形へ圧縮した再利用可能な局所構造**として扱うための設計原則を整理する。

本書は T0 BASE / SPEC の定義を変更しない。

T0において `M_B` は、

> 有限境界 `B` のもとで一時的に保持され、現在の解釈・予測・選択・応答・更新を拘束する自己側の有限関係拘束構造

である。

したがって本書では、

```text
Function = M_B
```

と無条件に同一視するのではなく、

```text
Function
=
現在の有限Bにおいて作用可能域を拘束する場合、
演算可能な形に圧縮された局所 M_B 片として扱える
```

と措定する。

これは T0 の追加公理ではなく、RDL_Functions 等で利用する **T2側の設計解釈**である。

---

## 1. 中核命題

本書の中心命題を次のように圧縮する。

> **Function は、関係拘束を再利用可能な演算形式へ圧縮した局所 M_B 片として扱いうる。**

一般的な関数表現

```text
y = f(x)
```

は、RDL的には少なくとも次を暗黙に持つ。

```text
Input x
↓
適用境界 B_f
↓
関係 / 拘束 / 評価規則
↓
演算
↓
Output y
```

つまり `f` は、

- 何を入力として読むか
- どの差異を保持するか
- どの関係を評価するか
- どの条件で出力を分岐するか
- 何を比較不能・未解決として残すか

を有限に固定した構造である。

その構造が現在の解釈・予測・選択・応答・更新を拘束するなら、RDL上は `M_B` の局所断片として読むことができる。

---

## 2. Function と M_B 全体を同一視しない

重要なのは、

```text
Function
≠ M_B 全体
```

である。

`M_B` は、関係・履歴・拘束・Commitment・Provenance・Authority・観測状態など、多様な有限構造を含みうる。

一方、Function は通常、その一部を演算可能な形式へ圧縮したものとして扱う。

```text
M_B
├ relation structure
├ observation
├ history
├ provenance
├ authority
├ commitment
├ active constraint
└ executable local structure
      ↑
    Function
```

したがって、本書でいう「Function は M_B である」とは、

> Function が M_B の完全な代替物である

という意味ではなく、

> Function が有限な関係拘束構造として M_B の一部を担いうる

という意味である。

---

## 3. Function の RDL 展開

### 3.1 一般形

従来の関数：

```text
y = f(x)
```

RDL展開：

```text
Result
=
f(
    input,
    B,
    Q,
    t,
    O,
    Purpose,
    evaluator identity/version
)
```

ただし、有限境界 `B` を用いる以上、

```text
ξ(B) ≠ 0
```

を継承する。

したがって、Function の出力は、世界についての終端的な確定値ではなく、

> **有限条件のもとで、その局所関係拘束構造を作用させた結果**

として扱う。

---

## 4. Function は「答え」ではなく拘束構造を持つ

Function の本体は出力値だけではない。

例えば、

```text
similarity(A, B) → 0.87
```

という結果の背後には、

```text
SimilarityFunction
├ 何を比較対象とするか
├ relation identity の扱い
├ strength の扱い
├ polarity の扱い
├ missing / unresolved の扱い
├ weighting
├ aggregation
├ comparison boundary
└ evaluator identity/version
```

が存在する。

つまり、類似比較関数を変更するということは、

```text
「似ている」の読み方を変更する
```

ことであり、RDL的には、

```text
比較に利用する局所 M_B を差し替える
```

こととして扱える。

---

## 5. 複数 Function = 複数の局所 M_B

同じ抽象操作に複数の Function が存在してよい。

例えば類似比較なら、

```text
RelationSimilarity
├ StrengthVectorComparator
├ PolarityAwareComparator
├ NeighborhoodComparator
├ TemporalComparator
├ AuthorityAwareComparator
└ ProvenanceAwareComparator
```

を持てる。

これを局所 M_B として読むと、

```text
M_B(sim-strength)
M_B(sim-polarity)
M_B(sim-neighborhood)
M_B(sim-temporal)
M_B(sim-authority)
M_B(sim-provenance)
```

という複数の有限な読み方が存在することになる。

したがって、

```text
score_A = 0.9
score_B = 0.9
```

であっても、

```text
Evaluator_A と Evaluator_B が異なる
→ 同一意味とは限らない
```

。

RDLでは結果だけでなく、

```text
どの局所 M_B / Function を使ったか
```

を回収可能にする。

---

## 6. Function Activation

すべての Function を常時使用する必要はない。

現在の `B`、問い、目的、資源、関係状態に応じて、必要な Function 群のみを前景化できる。

```text
Current B
↓
Function selection
↓
Activated local M_B set
↓
Interpretation / Comparison / Selection
```

例えば、

```text
B_graph
↓
Graph関係 Function群
```

```text
B_temporal
↓
Temporal Function群
```

```text
B_authority
↓
Authority-aware Function群
```

となる。

この意味で Function Activation は、

> **現在の断面で使う局所 M_B 群の選択**

として扱える。

---

## 7. B-switch との接続

RDLAI等で `B-switch` を行う場合、単に入力表現を変えるだけでなく、利用可能な Function 群も切り替わりうる。

```text
B1
├ Graph Comparator
├ Local Constraint Evaluator
└ Short-range Predictor

↓ E / H 増大
↓ B-switch

B2
├ Temporal Comparator
├ Multi-hop Relation Evaluator
└ Historical Pattern Function
```

したがって、

```text
B-switch
=
observation boundary の変更
+
activated local M_B / Function set の変更
```

という構造を取りうる。

---

## 8. M_B → Function

経験から安定した関係構造が抽出された場合、それを再利用可能な Function へ圧縮できる。

```text
経験群
↓
Relation Observation
↓
Relation Constraint Profile
↓
Similarity / Difference Comparison
↓
Common Relation Candidate
↓
例外条件分離
↓
検査
↓
局所安定した relation structure
↓
Function 化
```

この場合、

```text
M_B → Function
```

は、

> **経験から形成された有限関係拘束構造を、再利用可能な演算形式へ圧縮する操作**

となる。

---

## 9. Function → M_B

逆に、既存の数学関数・アルゴリズム・評価関数を RDLへ翻訳し、M_B の一部として利用できる。

```text
External Function
↓
RDL Translation
↓
Input boundary
↓
relation / constraint 展開
↓
unresolved / ξ の明示
↓
local M_B fragment
↓
M_B へ接続
```

この場合、外部 Function は無条件にRDL内部の確定構造になるのではない。

最低限、

```text
適用境界
入力条件
出力意味
評価器 identity/version
既知の破断条件
Provenance
```

を回収可能にする。

したがって、

```text
Function → M_B
```

も自動的な真理導入ではなく、有限構造の取り込みである。

---

## 10. 双方向変換

以上から、Function と M_B の間には次の双方向関係を置ける。

```text
M_B
↓ structure extraction / compression
Function

Function
↓ translation / contextualization
M_B fragment
```

圧縮すると、

```text
M_B ⇄ Function
```

である。

ただし両方向とも有限条件に依存し、変換によって `ξ` が消えることはない。

---

## 11. Function Composition

複数の Function を接続する場合、それは複数の局所 M_B 片を接続することでもある。

```text
Function_A
↓
Function_B
↓
Function_C
```

RDL的には、

```text
M_B(A)
↓
M_B(B)
↓
M_B(C)
```

という拘束構造の連鎖を形成する。

ここで注意すべきなのは、

```text
各Functionが局所的に妥当
≠
合成全体が同じ意味で妥当
```

である。

合成時には少なくとも、

```text
Boundary compatibility
Input / Output semantics
Provenance continuity
Authority compatibility
unresolved propagation
evaluator version
```

を検査対象とする。

---

## 12. Function 出力型

Function の出力を一種類の「答え」へ潰さない。

例えば、

```text
similarity()
→ SimilarityObservation

classify()
→ ClassificationObservation

rank()
→ RankingObservation

cluster()
→ ClusterObservation

predict()
→ InterpretationPrediction

generalize()
→ GeneralizationCandidate

optimize()
→ OperationalSelection
```

のように、**出力の意味型**を分離する。

特に、

```text
Observation
≠ Candidate
≠ Commitment
≠ Active Constraint
```

を維持する。

Function の実行成功や高スコアも、

```text
Truth
Completeness
Absolute Correctness
```

を意味しない。

---

## 13. Function と構造抽出

Function は構造抽出の結果として生成されうる。

```text
経験
↓
比較
↓
cluster
↓
common relation extraction
↓
exception extraction
↓
GeneralizationCandidate
↓
破断検査
↓
再利用可能な relation structure
↓
Function
```

この意味で、学習は単に Node を増やすだけではない。

```text
経験を保存する
```

から、

```text
経験から再利用可能な Function を獲得する
```

へ進むことができる。

これは `M_B` の構造形成を、演算部品の獲得として表現する一つの方法である。

---

## 14. Enterprise と Game の差

Function 自体を共通化しても、採用・更新・昇格方針は用途ごとに変えられる。

### Enterprise

```text
Function Candidate
↓
Boundary inspection
↓
Rupture / Durability Test
↓
Authority / Policy
↓
Shadow / Canary
↓
Promotion
```

一般化Functionの生成を許しても、CommitmentやActive Constraintへの昇格を厚く制御する。

### Game

```text
Experience
↓
Function Candidate
↓
軽量検査
↓
NPC local M_B
```

誤一般化や偏ったFunction形成を、NPCの思い込み・人格形成として利用できる。

つまり、

```text
Function generation policy
Validation policy
Promotion policy
```

は交換可能なパーツとして分離できる。

---

## 15. Function の自己例外化禁止

Function 自身も RDL の外部に特権化しない。

```text
Function
↓
有限な設計境界
↓
有限な入力・出力意味
↓
有限な評価器
↓
ξ remains
```

したがって、

```text
Function
→ inspection target
→ comparison target
→ replacement target
→ reconstruction target
```

となる。

Function の成功履歴が多いことや、数学的に確定的な実装であることも、その Function の適用意味が世界レベルで終端的に正しいことを保証しない。

実装上の exact equality や deterministic execution は保持してよいが、その意味解釈は現在の有限境界に束縛される。

---

## 16. RDL_Functions の再定義案

RDL_Functions は、

> 外部理論を RDL に翻訳した関数庫

という既存の役割に加え、より一般に、

> **再利用可能な局所 M_B を、演算可能な形で蓄積・比較・交換する T2 Function Library**

として位置づけられる。

概念的には、

```text
RDL_Core
=
M_B が成立・作用・更新するための基本力学

RDL_Functions
=
再利用可能な局所 M_B / operatorized M_B の関数庫

RDL_Durability_Modules
=
Function / M_B candidate の破断・耐久検査群

Application
=
Function群を選択・接続し、
具体的な有限 M_B として運用する系
```

と整理できる。

---

## 17. Function 共通記述形式案

各 Function 文書は、最低限次を記述する。

```text
RDLFunction
├ Function ID
├ Purpose
├ Input Type
├ Output Type
├ Required Boundary
├ Relation / Constraint Semantics
├ Evaluator Identity
├ Evaluator Version
├ Provenance Requirement
├ Coverage Semantics
├ Unresolved / ξ Handling
├ Composition Conditions
├ Known Rupture Conditions
├ Validation Policy
└ Promotion Semantics
```

これにより、Functionを単なるコード片ではなく、

> **適用条件と意味を回収可能な有限関係拘束モジュール**

として蓄積できる。

---

## 18. 最小受入条件

RDL Function を局所 M_B 片として扱う場合、少なくとも以下を満たす。

1. Function が適用される有限境界を明示または回収可能にする。
2. Function の出力意味型を明示する。
3. evaluator identity/version を回収可能にする。
4. `UNRESOLVED` や比較不能を、失敗・0・falseへ暗黙変換しない。
5. Function の実行成功を Truth や Completeness へ昇格しない。
6. Function Candidate を Commitment と同一視しない。
7. Function composition 時の境界・意味互換性を検査可能にする。
8. Function の Provenance を必要に応じて保持する。
9. Function 自体を再検査・比較・差替え可能にする。
10. Function の適用によって `ξ` が消失したと扱わない。

---

## 19. 圧縮表現

```text
関係拘束構造
↓ 圧縮・演算化
Function
↓ 実行
Observation / Candidate / Selection
↓
M_B の判断・更新へ接続
```

逆方向：

```text
External Function
↓ RDL翻訳
Boundary + Relation + Constraint
↓
local M_B fragment
```

統合すると、

```text
M_B ⇄ Function
```

ただし、

```text
Function ≠ M_B全体
Function ≠ Truth
Function ≠ Complete Model
∀B_finite: ξ(B) ≠ 0
```

---

## 20. 要約

RDLにおいてFunctionは、単なる計算道具としてだけでなく、

> **有限な関係拘束を、再利用可能かつ演算可能な形へ圧縮した局所 M_B 片**

として扱いうる。

この見方を採ると、

- Function の選択は局所 M_B の選択
- Function の差替えは読み方の差替え
- Function の合成は局所拘束構造の接続
- 構造抽出は新しい Function の獲得
- 外部アルゴリズムの翻訳は Function → M_B
- 学習した関係構造の演算化は M_B → Function

として統一的に記述できる。

RDL_Functions はこの双方向性を扱う T2 層として、

> **再利用可能な局所 M_B / Function の蓄積・翻訳・比較・交換基盤**

へ発展させることができる。
