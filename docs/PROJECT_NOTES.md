# プロジェクト概要

パチンコ参加人口の減少をBass拡散モデルで分析するJuliaノートブック。

## ファイル構成

| ファイル | 内容 |
|---------|------|
| `bass_pachinko.ipynb` | メインノートブック。定数M → ホール比例M(t) → ホール+人口M(t) の3モデルを順次推定 |
| `../Pachinko/p-datasets2.csv` | パチンコ産業の年別データ（1989-2019年） |

## データ仕様: p-datasets2.csv

- CSVカラム名は日本語。主要カラム:
  - `年`: 1989-2019
  - `総人口`: 千人単位（例: 125265 = 1.25億人）
  - `店舗数`: ホール数（実数、例: 18113）
  - `パチンコ参加人口`: 万人単位（例: 29300 = 2930万人）。1994年以降のみ存在
- パチンコ参加人口が `missing` の行はフィルタで除外: `valid = .!ismissing.(df[!, "パチンコ参加人口"])`
- CSVカラム名は `"店舗数"` のまま（データソース準拠）。コード内の変数・ラベルは `halls` / `ホール` に統一

## モデル設計

### Bass拡散の解釈

「パチンコ市場からの離脱」がBass拡散に従うと仮定:
- F(t) = 離脱率（累積）、dF/dt = (p + qF)(1-F)
- P(t) = M(t) · (1 - F(t)) = 参加人口

F(t) のODEはM(t)に依存しない。M(t)は観測方程式にのみ現れる。

### M(t) のモデリングで注意すべき点

- **M(t) を自由な線形関数（α₀+α₁·Pop+α₂·Shops）にすると識別不能**: P(t)=M(t)·(1-F(t)) の積の分解が一意でないため、M が拡大し F で帳尻を合わせる非現実的な解に収束する（cor(α₀,F₀)≈0.6）
- **共変量をODE内に加法的に入れると収束失敗**: p+β·x が負になりF(t)が非物理的に減少。ESS≈10、loglik=-7.76e6
- **解決策**: M(t) の形状をデータ（ホール数比率）で固定し、スケールと弾力性のみ推定
  ```
  M(t) = M₀ · (Halls(t)/Halls₀)^α
  ```

### 人口の影響

- 1994-2019年の人口変動幅はわずか2.25%（vs ホール数47.5%）
- α_p の95%CIが0を含む場合、この期間のデータでは人口効果の分離は困難
- 将来の人口減シナリオ（5-20%減）の感度分析は参考値として解釈

## Julia / パッケージ上の注意

### ModelingToolkit.jl v11+
- `ODEProblem(sys, u0, tspan, p)` の4引数形式は非推奨
- 正しい形式: `ODEProblem(sys, [F => F0, p_bass => p_val, q_bass => q_val], tspan)`
- `using ModelingToolkit: t_nounits as t, D_nounits as D` で時間変数とDifferentialをインポート

### Turing.jl v0.43+
- チェーンからのパラメータ抽出: `vec(chain[:p])` で動作する
- `summarize(chain)` ではなく `summarystats(chain)` を使う
- ODE解のretcode確認: `sol.retcode != ReturnCode.Success`
- ODE解が失敗した場合: `Turing.@addlogprob! -Inf; return`

### MCMCでのODE
- ModelingToolkitのシンボリックシステムはモデル定義に使用
- MCMC内部では素のJulia関数 `bass_ode!(du, u, params, t)` を使用（高速化のため）
- ODE関数内の変数名はTuringモデルのパラメータ名と衝突しないよう `Fv, pp, qq` 等を使う

## 使用パッケージ一覧

```julia
using CSV, DataFrames
using ModelingToolkit, OrdinaryDiffEq
using Turing, MCMCChains
using Plots, StatsPlots, Statistics, Random
```
