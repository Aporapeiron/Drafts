# RDL 関係拘束類似比較

## 1. 目的

本書は、RDLにおいて複数の対象・案件・ノード・状態が「どの程度似ているか」を、語彙表面や埋め込み空間だけではなく、**有限境界における関係拘束の配置と強さの比較**として扱うための基本契約を定義する。

本書における類似は、対象同士が世界そのものにおいて「本当に似ている」ことを認証するものではない。

類似は常に、有限な境界・問い・時点・観測断面・目的の下で形成された比較観測である。

```text
Similarity
=
Similarity(B, Q, t, O, Purpose)
```

有限な比較境界を用いる以上、未回収関係 ξ は残存する。

---

## 2. 基本位置づけ

RDLにおける関係拘束類似比較は、概ね次の位置にある。

```text
対象 / 案件 / Node
        ↓
Relation Observation
        ↓
Constraint Evaluation
        ↓
Relation Constraint Profile
        ↓
Relation Constraint Comparison
        ↓
Relation Similarity Observation
        ↓
cluster / pattern extraction
        ↓
candidate relation structure
        ↓
M_B 更新候補
```

関係拘束類似比較は、それ自体でCommitmentやM_B更新を意味しない。

```text
Similarity Observation
≠ Commitment
≠ Active Constraint
≠ Truth
```

---

## 3. 関係拘束プロファイル

### 3.1 定義

ある対象 `X` に対する関係拘束プロファイルを、現在の有限境界において比較対象として選択された関係拘束群とする。

```text
Profile(X | B, Q, t, O, Purpose)
```

初期実装では、各要素は少なくとも次を持つ。

```text
RelationConstraintEntry
├ relation identity
├ strength
├ polarity
├ observation status
└ provenance / boundary reference
```

必要に応じて以下を追加できる。

```text
freshness
authority
source
convergence
target scope
evaluator identity/version
```

### 3.2 strength の意味

`strength` は「真理性」ではない。

```text
高い relation constraint strength
≠ 真実
≠ 正解
≠ 絶対的確実性
```

`strength` は、現在の有限境界と問いにおいて、そのrelationが解釈・予測・選択・応答をどの程度拘束しているかを表す局所的な量である。

---

## 4. 類似比較の基本原則

### 4.1 類似は関係拘束配置の近さとして扱う

対象 `A` と `B` の類似を、選択されたrelation群における拘束プロファイルの近さとして観測する。

```text
Similarity(A, B)
=
Compare(
    Profile(A | B_cmp),
    Profile(B | B_cmp)
)
```

ここで `B_cmp` は比較用に宣言された有限境界である。

### 4.2 比較不能を低類似へ潰さない

次は区別しなければならない。

```text
弱い拘束
≠ 関係なし
≠ NOT_OBSERVED
≠ UNRESOLVED
```

特に、片側のrelationが未観測・未解決の場合、それを `strength = 0` とみなしてはならない。

```text
A.r = 0.8
B.r = UNRESOLVED

→ 差 0.8 として扱わない
→ comparison coverage の不足として扱う
```

### 4.3 類似度と比較範囲を分離する

比較結果は、単一のscoreだけで表現しない。

最低限、

```text
score
coverage
compared_relations
conflicting_relations
unresolved_relations
```

を分離する。

例:

```text
score    = 0.86
coverage = 0.61
```

これは、

> 現在比較可能な関係断面では高い類似が観測されたが、比較可能だった関係は全体の61%である

ことを意味する。

---

## 5. 最小比較契約

### 5.1 relation alignment

最初に、比較可能なrelation identityを揃える。

```text
Profile A              Profile B

r1  ───────────────→   r1
r2  ───────────────→   r2
r3                    unresolved
                    ←  r4
```

同一relation identityでない項目を、暗黙に同一意味へ統合してはならない。

relation aliasやontology mappingを導入する場合は、それ自体を独立した有限契約として扱う。

### 5.2 polarity

polarityが一致している場合と反転している場合を分離する。

```text
SUPPORT vs SUPPORT
→ strength distanceを比較可能

OPPOSE vs OPPOSE
→ strength distanceを比較可能

SUPPORT vs OPPOSE
→ relation conflict

UNRESOLVED
→ similarity scoreへ直接投入しない
```

### 5.3 strength distance

初期実装では単純な距離でよい。

`strength ∈ [0, 1]` とした場合:

```text
similarity_r(A, B)
=
1 - |strength_A(r) - strength_B(r)|
```

ただし、これは**比較適格な同一relationかつpolarity条件を満たす場合のみ**適用する。

### 5.4 aggregate

比較可能relation集合を `R_cmp` とすると、

```text
score
=
weighted_mean(
    similarity_r(A, B)
    for r in R_cmp
)
```

とできる。

重みは現在の問い・目的に対するrelevance等から与えてよいが、重みの出所と評価器identity/versionは回収可能でなければならない。

---

## 6. coverage

coverageは、今回の比較でどの程度の関係断面を実際に比較できたかを表す。

単純形:

```text
coverage
=
|R_cmp|
/
|R_selected|
```

ここで、

- `R_selected`: 比較対象として選択されたrelation集合
- `R_cmp`: 両側で比較適格だったrelation集合

とする。

coverageが低い場合、高いscoreが得られても強い類似主張へ昇格してはならない。

```text
high score + low coverage
≠ strong global similarity
```

---

## 7. Comparison Result

初期Core候補としては、次のような型を想定できる。

```text
RelationConstraintComparison
├ left_profile
├ right_profile
├ boundary
├ evaluator_id
├ evaluator_version
└ provenance
```

比較結果:

```text
RelationSimilarityObservation
├ score
├ coverage
├ compared_relations
├ conflicting_relations
├ unresolved_relations
├ boundary
├ evaluator_id
├ evaluator_version
└ provenance
```

### 規範

- `score` MUST NOT imply Truth.
- `coverage < 1` MUST NOT be silently treated as complete comparison.
- `UNRESOLVED` MUST NOT collapse into zero strength.
- polarity conflict MUST NOT be averaged away without explicit policy.
- evaluator identity/version MUST be recoverable.
- comparison boundary MUST be recoverable.
- similarity observation MUST NOT directly create Commitment.
- current RDL comparison rules themselves MUST remain inspectable and revisable.

---

## 8. embedding similarity との違い

embedding similarityとrelation-constraint similarityは別物である。

```text
Embedding Similarity
=
表現空間上の距離・近さ

Relation Constraint Similarity
=
有限境界における関係拘束配置・極性・強さの近さ
```

したがって、

```text
表現は似ている
しかし
関係拘束配置は異なる
```

こともあり得る。

逆に、

```text
語彙表面は大きく異なる
しかし
同じようなrelation fieldに置かれている
```

ため、関係拘束類似が高くなる場合もある。

embeddingは候補生成器として利用できるが、その結果をRDL内部の関係拘束類似へ自動昇格してはならない。

---

## 9. Graph比較への拡張

単一relation strength比較の次に、対象の周辺relation配置を比較できる。

### v0: 直接relation比較

```text
A
├ r1
├ r2
└ r3
```

と

```text
B
├ r1
├ r2
└ r3
```

のstrength / polarity / statusを比較する。

### v1: 1-hop relation neighborhood

```text
A
├ support → X
├ oppose → Y
└ authority → Z
```

と

```text
B
├ support → X'
├ oppose → Y'
└ authority → Z'
```

のように、対象そのものだけでなく周辺relationの配置を比較する。

ここでは「同じ語であるか」より、

> 同じような関係場の位置に置かれているか

を比較できる。

### v2: multi-hop relation structure

必要に応じて複数hopへ拡張できるが、比較境界を無制限に広げてはならない。

```text
hop depth
relation kinds
target scope
observation window
```

を有限条件として明示する。

---

## 10. 構造抽出への接続

関係拘束類似比較は、単なる検索機能ではなく、構造抽出の前段として利用できる。

```text
多数の経験
↓
Relation Constraint Profile
↓
類似比較
↓
類似profile群
↓
共通relation
例外relation
対向relation
分岐条件
を抽出
↓
candidate relation structure
↓
有限Bで検査
↓
M_B更新候補
```

ここで重要なのは、

```text
cluster
≠ 新しい規則
```

である。

類似群が見つかったことだけで、新しいrelationやCommitmentを確立してはならない。

構造抽出結果もcandidateとして保持し、比較境界・Provenance・Authority・検証条件を通して初めてM_B更新候補となる。

---

## 11. 実装ロードマップ

### Stage 0

```text
RelationConstraintProfile
├ relation_id
├ strength
├ polarity
└ status
```

### Stage 1

```text
RelationConstraintComparison
↓
score
coverage
unresolved
conflict
```

### Stage 2

```text
1-hop graph similarity
```

### Stage 3

```text
profile clustering
↓
common relation extraction
```

### Stage 4

```text
candidate structure generation
↓
bounded validation
↓
M_B update candidate
```

---

## 12. 最小受入条件

初期実装が関係拘束類似比較として受け入れ可能であるためには、少なくとも以下を満たす。

1. 比較境界が明示または回収可能である。
2. relation identityを暗黙統合しない。
3. UNRESOLVEDを0へ潰さない。
4. scoreとcoverageを分離する。
5. polarity conflictを明示する。
6. evaluator identity/versionを回収可能にする。
7. comparison resultをTruthやCommitmentへ昇格しない。
8. profile入力が同一比較目的に対して比較適格か検査できる。
9. 比較不能relationを残存集合として保持する。
10. RDL自身の比較規則も再検査・更新可能な有限構造として扱う。

---

## 13. 圧縮表現

```text
対象 A
↓
Profile_A

対象 B
↓
Profile_B

same finite comparison boundary
↓
relation alignment
↓
polarity / status eligibility
↓
strength comparison
↓
SimilarityObservation
├ score
├ coverage
├ conflict
└ unresolved

SimilarityObservation
≠ Truth
≠ Commitment
≠ Complete similarity

↓
repeated comparison
↓
cluster / common relation extraction
↓
candidate M_B structure
```

---

## 14. 要約

RDLにおける類似は、対象そのものの絶対的近さではなく、

> **現在の有限境界において、選択された関係拘束配置がどの程度近く観測されるか**

として扱う。

そのため、関係拘束類似比較では単一scoreよりも、

```text
score
coverage
conflict
unresolved
boundary
provenance
```

を同時に保持することが重要である。

この比較層は、類似案件検索だけでなく、将来的なクラスタリング、共通関係抽出、例外条件分離、M_B再編候補生成の基礎となる。
