# NBD-Bass-Pachinko

**閾値観測下の市場拡散動態：NBD-Bass 統合ベイズモデルによる日本パチンコ市場の構造変化分析**

*Threshold-Adjusted Diffusion Dynamics: A Joint NBD-Bass Bayesian Model for Japan's Pachinko Market*

[![License: MIT (code)](https://img.shields.io/badge/code-MIT-blue.svg)](LICENSE)
[![License: CC BY 4.0 (paper)](https://img.shields.io/badge/paper-CC%20BY%204.0-lightgrey.svg)](LICENSE-PAPER)
[![Julia 1.10+](https://img.shields.io/badge/Julia-1.10%2B-9558B2.svg)](https://julialang.org/)
[![Turing.jl](https://img.shields.io/badge/Turing.jl-0.43%2B-purple)](https://turinglang.org/)

---

## 概要

日本パチンコ市場における 1994–2019 年の参加人口減少（2930 万人 → 890 万人、約 70% 減）を、**Bass 拡散モデル + 負の二項分布 (NBD) 頻度モデル** の階層ベイズ統合で分析するプロジェクトです。

「年 1 回以上利用」という閾値観測がもたらす識別バイアスを NBD 閾値補正で明示的に取り除き、

- **離脱率の構造変化点 $c_2 = 2014$**（スマートフォン普及期と整合）
- **市場の濃縮化**（α 減少、E[λ] = 56→78 回/年への上昇）
- **真の潜在プレイヤープール $M \approx 3800$ 万人**（観測値の 15% 過小評価を補正）

を頑健に推定しました。

📄 **論文（日本語フルペーパー）**: [`paper/paper.md`](paper/paper.md) / [`paper/paper.html`](paper/paper.html)
📓 **メイン推定ノートブック**: [`notebooks/04_Pachinko_NBD_Bass_Joint.ipynb`](notebooks/04_Pachinko_NBD_Bass_Joint.ipynb)

---

## モデル概要

### 個人レベル：NBD 頻度

$$X_i(t) \mid \lambda_i(t) \sim \text{Poisson}(\lambda_i(t)),\quad \lambda_i(t) \sim \text{Gamma}(r(t), \alpha(t))$$

### 集団レベル：Bass 拡張 3 状態 ODE

$$\frac{ds}{dt} = -(p_1 + q_1 a)\,s,\quad \frac{da}{dt} = (p_1 + q_1 a)(s + \alpha x) - p_2\,a,\quad \frac{dx}{dt} = p_2\,a - \alpha(p_1 + q_1 a)\,x$$

### 観測：二重尤度

- **観測 1**：年次参加人口 (N=26)
$$A_\text{obs}(t) \sim \mathcal{N}\bigl(M \cdot a(t) \cdot P_\text{active}(t),\ \sigma^2\bigr)$$
- **観測 2**：頻度ビン分布 (N=9 年×5 ビン)
$$\mathbf{n}_t \sim \text{Multinomial}\bigl(1500,\ \hat{\mathbf{p}}_\text{bin}(r(t), \alpha(t))\bigr)$$

詳細は [`paper/paper.md` 第 3 章](paper/paper.md#3-モデル) を参照。

---

## リポジトリ構成

```
NBD-Bass-Pachinko/
├── README.md                   ← このファイル
├── LICENSE                     ← コード MIT
├── LICENSE-PAPER               ← 論文・図 CC BY 4.0
├── CITATION.cff                ← 学術引用情報
├── .gitignore
│
├── paper/                      ← 論文一式
│   ├── paper.md                ← Markdown 本体（約 35,000 字）
│   ├── paper.html              ← MathJax 対応の自己完結 HTML
│   ├── paper.css
│   └── figs/                   ← 8 点の図 (PNG)
│
├── notebooks/                  ← Julia ノートブック (Turing.jl)
│   ├── 01_bass_pachinko.ipynb              ← 標準 Bass + M(t) 検証
│   ├── 02_Pachinko_NBD_Bass.ipynb          ← Step1: 年別 NBD MLE
│   ├── 03_Pachinko_NBD_Bass_3State.ipynb   ← 2 段階 Bass 推定
│   ├── 04_Pachinko_NBD_Bass_Joint.ipynb    ← ★ 統合ベイズ（メイン）
│   ├── 05_Pachinko_NBD_Bass_Regulation.ipynb ← 規制ダミー版
│   └── 06_Pachinko_NBD_Bass_Mt.ipynb       ← 時変 M(t) 拡張
│
├── data/                       ← 公開ソースの集計データ
│   ├── p-datasets2.csv         ← 1989–2019 年次パネル
│   └── pachinko-frequency.csv  ← 2008–2020 頻度分布
│
└── docs/                       ← 補助文書
    ├── PROJECT_NOTES.md        ← 開発メモ・Julia tips
    └── RESEARCH_LOG.md         ← 研究ログ（モデル比較表）
```

---

## 主要結果（事後分布要約）

| パラメータ | 中央値 | 95% CI | 識別性 |
|---|---:|:---:|:---:|
| 離脱ピーク $c_2$ | **2014.0** 年 | [2010, 2018] | ○ |
| 市場規模 $M$ | **3808** 万人 | [3200, 4665] | ○ |
| NBD α 年率変化 $a_1$ | **−0.022** | [−0.038, −0.006] | △（有意） |
| NBD 形状 $r$ (2013) | 0.369 | [0.348, 0.391] | ○ |
| 観測ノイズ $\sigma$ | 162.6 万人 | [123, 234] | ○ |
| $p_{2,\text{base}}$ | 0.145 | [0.076, 0.282] | △ |

すべての $\hat R \le 1.006$、最小 ESS = 696。

---

## 再現方法

### 環境

- Julia 1.10+
- パッケージ：
```julia
using Pkg
Pkg.add([
  "CSV", "DataFrames",
  "DifferentialEquations", "OrdinaryDiffEq",
  "Turing", "MCMCChains",
  "Distributions", "SpecialFunctions",
  "Plots", "StatsPlots", "Statistics"
])
```

### 実行手順

```bash
git clone https://github.com/kimseok1973/NBD-Bass-Pachinko.git
cd NBD-Bass-Pachinko
julia --project=.

# notebooks/04_Pachinko_NBD_Bass_Joint.ipynb を Jupyter または VS Code で開いて実行
```

MCMC は 4 コアで約 22 分（M2 Pro）。NUTS sampler、4 chains × 1000 iterations。

---

## データ出典

- **パチンコ参加人口**：公益財団法人日本生産性本部『レジャー白書』各年版 (1994–2019)
- **頻度分布調査**：日本遊技関連事業協会『遊技業界データブック』各年版 (2008–2020)
- **補助データ**：労働時間・賃金統計（総務省）、人口統計（総務省）、台数統計（業界資料）

すべて公開資料に基づく集計値です。

---

## 引用

学術的に本研究を引用される場合：

```bibtex
@misc{kim2026nbdbass,
  author       = {Kim, Tetsu},
  title        = {Threshold-Adjusted Diffusion Dynamics: A Joint NBD-Bass Bayesian Model for Japan's Pachinko Market},
  year         = {2026},
  howpublished = {GitHub repository},
  url          = {https://github.com/kimseok1973/NBD-Bass-Pachinko}
}
```

詳しくは [`CITATION.cff`](CITATION.cff) を参照。

---

## ライセンス

- **コード（`notebooks/`, スクリプト）**：MIT License → [`LICENSE`](LICENSE)
- **論文本体・図・データ整理**（`paper/`, `docs/`, `data/`）：CC BY 4.0 → [`LICENSE-PAPER`](LICENSE-PAPER)

データ自体は出典機関（レジャー白書、日遊協）の著作物に基づく派生集計値です。

---

## 連絡

Issue または Pull Request 大歓迎です。特に以下の方向の拡張に興味があります：

- 月次・四半期データへの拡張
- 共変量入りモデル（ホール数、新台投入、広告 GRP）
- 階層的縦断モデル（BG/NBD 拡張）
- 他カテゴリ（フィットネス、ストリーミング、リテール）への適用

---

*Last updated: 2026-05-20*
