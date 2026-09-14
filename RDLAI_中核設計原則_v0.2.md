# RDLAI 中核設計原則 v0.2

*RDL応用案 / RDLAI / 近接実装層・暫定文書*  
*依存：RDL_Core BASE / SPEC v2.3、RDL_Functions、RDL_Durability_Modules*

## 0. 位置づけ

本書は、RDLを基盤としてAIを設計する場合の、比較的近い実装範囲にある中核設計原則を整理する。

対象は、まず一つのRDLAI、または限定された少数の補助系からなる認知・推論システムである。多数の自律AIからなる政治制度・経済・人工社会までは扱わない。

RDLAIはRDL_Coreそのものではない。本書で導入するRouter、キャッシュ、計算深度、睡眠相、監視系等は、**Coreを実装へ接続するための候補機構**であり、T0 Primitiveではない。

### 0.1 Core v2.3との基本接続

現行RDLでは、AIへの入力を旧 `EFP` として扱わない。

```text
RDLAI as a SILN candidate
        ↕
 environment / user / tools / memory / other systems の {RIB_i}
        ↓ Purpose / B
      RIB_B
        ↓ M_B
F = interp(M_B, RIB_B)
        ↓
Functions / reasoning / action
        ↓
後続する相互作用条件が変化
```

ここで、

```text
RDLAI全体 ≠ M_B
raw input ≠ RIB_B
RIB_B ≠ F
Function ≠ M_B
noise / uncertainty / unknown ≠ ξ
```

とする。

`M_B` は、RDLAIというSILNを現在のPurpose / Bのもとで自己側・解釈側として保持した有限構造断面である。`RIB_B` は、関与する複数のRIBから現在の問いに必要なものを取得・選択した未解釈の有限作用断面である。

`ξ` は「知らない情報のリスト」「探索すべき未知量」「ランダムノイズ」「未処理キュー」ではない。有限Bで関係を完全に回収し切れないというCore条件である。

---

## 1. 中核原則

RDLAIの初期設計原則を、次の五点に圧縮する。

1. **常時最大推論をしない。**
2. **Purpose / Bに応じて取得断面と計算断面を切り替える。**
3. **Functionを型付き・有限契約を持つ演算モジュールとして使う。**
4. **自己検査とDurability検査は条件付きで起動する。**
5. **大規模な圧縮・再評価・再構成にはオフライン相を持てる。**

これらは、有限計算資源の配分問題として接続する。

```text
有限計算資源
↓
Purpose / Risk を判定
↓
必要な B と RIB_B を構成
↓
必要なFunctionsだけ起動
↓
局所応答
↓
必要なら比較・耐久検査・再構成
↓
高コスト処理はオフライン相へ送る
```

---

## 2. 常時最大推論をしない

### 2.1 基本方針

すべての相互作用に対して最大規模の推論を起動しない。

挨拶、定型操作、既知の局所問題などは、限定されたPurpose / B、RIB_B、Function群、計算資源で処理する。

```text
interaction conditions
↓
軽量な B
↓
限定 RIB_B
↓
軽量 interp / Functions
↓
response
```

一方で、重大な判断、高い失敗コスト、反復する不整合、複数断面間の強い矛盾などでは、取得範囲・検査深度・計算資源を増やす。

### 2.2 暫定コスト関数

推論コストは固定値ではなく、例えば次の実装変数に依存させられる。

```text
RequiredCost = f(
  task_complexity,
  uncertainty_model,
  failure_cost,
  reversibility,
  relation_context,
  repeated_mismatch,
  available_resources,
  latency_budget
)
```

`uncertainty_model` は実装上の不確実性指標であり `ξ` ではない。

Core `E / H` を使う場合は、SPECの定義を満たす比較が構成できるときだけ含める。

> **知能を「常に深く考える能力」と同一視せず、必要なときだけ必要な深さまで計算資源を配分できることも能力として扱う。**

---

## 3. Boundary / Section Router

### 3.1 Routerの役割

RDLAIには、現在のPurposeに対して、

- 何を対象にするか
- どの時間幅を見るか
- どの尺度を見るか
- どの相互作用を取得するか
- どのFunctionを使うか
- どの程度の計算深度を使うか

を選択するRouterを置ける。

入力候補：

```text
task / request
current interaction state
history
risk / failure cost
resource state
coverage / unresolved
previous results
```

出力候補：

```text
Purpose
B_current
acquisition policy for RIB_B
Function set
representation
reasoning_depth
resource_budget
provenance requirements
```

### 3.2 表現変更とB変更を分ける

Graph / Field / Flow / Sequence / Hierarchy / State / Languageなど、表現形式を変えるだけで必ずCore `B` が変わるとは限らない。

同じPurpose・対象範囲・尺度・時間断面を保ったまま表現だけを変えるなら、同一B内のrepresentation shiftとして扱える。

一方、

- 問いが変わる
- 対象範囲が変わる
- 時間幅が変わる
- 尺度が変わる
- 取得する関係が変わる

場合は、`B` の変更候補として記録する。

### 3.3 B-switch

現在の有限断面で結果が不十分でも、単純に同じ推論を長く回すとは限らない。

```text
B1
↓
RIB_B1 / M_B1
↓
F1 / Functions
↓
coverage不足・反復失敗・高リスク
↓
B-switch candidate
↓
B2
```

ここでは、

> 「推論が足りない」のか  
> 「取得断面が悪い」のか  
> 「Function契約が合っていない」のか  
> 「現在の構造断面では扱えない」のか

を分けて検査する。

---

## 4. Multi-B Projection

必要に応じて、同じ対象SILNを複数のBから並列に見ることができる。

```text
same target SILN
 ├─ B_graph    → M_Bg / RIB_Bg → F_g
 ├─ B_temporal → M_Bt / RIB_Bt → F_t
 ├─ B_risk     → M_Br / RIB_Br → F_r
 └─ B_social   → M_Bs / RIB_Bs → F_s
```

これは、

```text
B1 → SILN1
B2 → SILN2
```

という意味ではない。

同じ対象について異なる有限断面を並列に構成する操作である。

各 `F_i` を必ず一つの結論へ潰す必要はない。

- 一致部分
- 断面固有部分
- 矛盾部分
- coverage差

を保持し、追加検査へ回せる。

---

## 5. Functionを演算単位として使う

RDLAIの内部処理をすべて `M_B` の一部として表現しない。

RDL_Functionsに従い、Functionは概念的に、

```text
Function_B = Operator(
  Purpose / B_f,
  input contract,
  local constraints,
  transform / evaluate rule,
  output contract,
  provenance,
  unresolved / failure conditions
)
```

として扱う。

### 5.1 型付き接続

Function入力は常に `RIB_B` とは限らない。

```text
RIB_B
F
state
profile
record
other Function output
```

などを契約に応じて受け取れる。

ただし、役割の異なる型を直接同一視しない。

例えば、

```text
Function_A output = F
Function_B input  = RIB_B
```

なら、そのまま接続せず、action / environment / acquisition等を介して次のRIB条件を形成し、Bで `RIB_B` を構成する必要がある。

### 5.2 Provenance

RDLAIでは、出力だけでなく、

- どのBか
- どのRIB_Bを取得したか
- どのFunctionを通ったか
- どのモデル・記録を使ったか
- coverageはどこまでか
- unresolvedは何か

を必要に応じて保持する。

---

## 6. 相手・文脈に応じた局所計算

RDLAIは「誰に対しても同じ計算量」を使わない。

ただし、

```text
人物 → 固定計算量
```

のような固定対応にはしない。

より適切なのは、

```text
current relation
× current task
× interaction history
× risk
× resource state
→ current computation policy
```

である。

反復関係では、共有語彙、説明密度、既知の前提、過去の成功したFunction構成等を局所キャッシュとして利用できる。

ただし、その相手を「完全に分かったモデル」として終端化しない。

ここで残る再検査可能性を `ξ` の量として保存するのではなく、

```text
coverage
assumptions
staleness
unresolved
provenance
```

として実装上明示する。

---

## 7. Core E / H と実装指標を分ける

RDLAIでは、すべてのエラー・不確実性・負荷をCore `E / H` に押し込まない。

Core標準比較を使うなら、

```text
RIB_B(t)   = Section_B({RIB_i(t)})
F(t)       = interp(M_B, RIB_B(t))

RIB_B(t+Δ) = Section_B({RIB_i(t+Δ)})
F'(t+Δ)    = interp(M_B, RIB_B(t+Δ))

E(t+Δ)     = Δ(F, F')
```

とし、未解消差分が履歴的に残る場合のみ `H` を構成する。

一方で実装上は、別に、

```text
loss
confidence
uncertainty
latency
queue_size
memory_pressure
security_score
coverage_gap
```

等を持ってよい。

これらは自動的に `E / H / ξ` ではない。

---

## 8. メタ循環は条件付きで起動する

自己検査・メタ認知・耐久検査は高コストであるため、常時全面的には回さない。

暫定的に、

```text
L0  定型Function
L1  通常推論
L2  B / Function 切替
L3  局所Durability検査
L4  自己構造の再検査・再構成
```

といった深度を置ける。

L3以上へ進む候補条件は、

- 同種の失敗が反復する
- 複数Bの結果が強く競合する
- coverage不足が高リスク領域へ重なる
- Function契約違反が反復する
- 更新後に性能が不安定化する
- Core定義上の `H ≥ θ` が成立する

などである。

`H ≥ θ` は唯一のメタ推論トリガーではない。実装上のrisk / security / integrity条件から検査を起動してもよい。

---

## 9. Durabilityによる自己検査

RDLAI自身、内部Function、記憶、Router、取得規則を検査対象にできる。

```text
Target:
  RDLAI SILN
  or Function
  or current finite model

↓ choose B_test
↓ stress / remove / B-change / history / constraint / simulation
↓ observe outputs / failures / coverage
↓
maintain / transform / transition / break
```

外部対象へ向ければ品質・安全検査、自己へ向ければSelf Inspectionになる。

ただし「破断した＝攻撃された」と即断しない。

```text
observed break
↓
attack?
fault?
contract mismatch?
stale model?
missing acquisition?
resource exhaustion?
internal inconsistency?
```

を後段で分類する。

---

## 10. オフライン相 / 睡眠相

### 10.1 定義

RDLAIにおける睡眠相とは、

> **オンライン応答から計算資源を一部または大部分切り離し、履歴整理、圧縮、モデル比較、Durability検査、必要な再構成へ資源を配分する運用相**

である。

これはCore Primitiveでも、生物の睡眠をそのまま模倣する定義でもない。

### 10.2 睡眠相と `M_Δ` を同一視しない

通常のキャッシュ整理・圧縮・インデックス更新は、Core再構成を必要としない。

```text
maintenance sleep
→ cache cleanup
→ compression
→ provenance repair
→ stale-data inspection
```

一方、既存 `M_B` では反復的な差分を扱えず、T1の再構成条件が成立する場合には、

```text
M_Δ
↓ 展開・検査・選別
M_B'
```

へ接続できる。

したがって、

```text
sleep ≠ M_Δ
```

である。

### 10.3 睡眠深度候補

```text
浅い相
→ キャッシュ整理
→ 重複圧縮

通常相
→ 履歴比較
→ stale model検査
→ Function耐久検査

深い相
→ Multi-B再検査
→ 自己モデル候補展開
→ 必要なら M_Δ / 再構成

保守相
→ セキュリティ
→ データ整合性
→ provenance検査
```

### 10.4 外界ゲート

安全系を認知系から分離できる場合、深いオフライン相では外界入力を強く抑制できる。

```text
RDLAI cognition
→ offline inspection / reconstruction

external safety shell
→ power / hardware monitor
→ security
→ emergency interrupt
→ recovery control
```

---

## 11. 最小実行ループ

```text
1. interaction / request を受ける
2. Purpose / Risk を判定する
3. B候補を生成する
4. 関与RIBから acquisition により RIB_B を構成する
5. M_B が RIB_B を解釈し F を形成する
6. 必要なFunction群を型付きで実行する
7. provenance / coverage / unresolved を保持する
8. response / action を返す
9. action により後続RIB条件が変化する
10. 必要な場合だけ F / F' を比較し Core E を構成する
11. 未解消Eが残る場合だけ H を更新する
12. risk / failure / durability / H を用いて次の処理深度を選ぶ
13. 必要なら B-switch / Function-switch / Multi-B を実行する
14. 必要なら Durability検査を実行する
15. 構造再構成が必要なら M_Δ → M_B' へ接続する
16. 高コスト処理はオフライン相へ送る
17. 再接続前に安全・契約・耐久を検査する
```

このループで重要なのは、

```text
Input → Model → Answer
```

という一方向処理に閉じないことである。

RDLAIの応答は環境・相手・ツール・記録を変え、その変化が後続するRIB条件へ戻る。

---

## 12. 初期実装で扱わないもの

本書では、次を将来課題として外す。

- 多数のRDLAI個体からなる大規模共同体
- AI間の政治制度・統治
- 人工国家
- AI生態系の長期進化
- 自律的資源経済
- 世代交代・複製制度

これらは別文書で扱う。

---

## 13. 実装上の禁止事項

RDLAI v0.2では、少なくとも次を避ける。

```text
EFPを現行Core入力として使う
raw input = RIB_B とする
AI全体 = M_B とする
B変更 = 表現形式変更 とする
B1 → SILN1 / B2 → SILN2 とする
noise / entropy / uncertainty / unknown = ξ とする
ξをランダム注入する
queue_size / stress / compute load = H とする
prediction error = Core E と無条件に置く
sleep = M_Δ とする
Function = M_B とする
```

---

## 14. 一文圧縮

> **RDLAIとは、SILNとRIB群の相互作用を有限なPurpose / Bで扱い、必要なRIB_Bだけを取得し、型付きFunctionへ計算資源を配分し、結果と未解決部分を記録し、必要なときだけ断面変更・耐久検査・再構成・オフライン処理を起動するRDL応用AI設計である。**

この設計自体もSILNとして対象化され、Durability検査と再構成の対象になる。
