# GitHub への push 手順

リポジトリ `kimseok1973/NBD-Bass-Pachinko` への初回 push 手順です。

OneDrive 同期フォルダ内では git 初期化が不安定になりがちなので、**OneDrive 外**にクローンしてから作業することを推奨します。

## 推奨手順（OneDrive 外で作業）

### Step 1: 任意の作業フォルダ（例：`C:\dev`）で空リポジトリをクローン

```powershell
cd C:\dev
git clone https://github.com/kimseok1973/NBD-Bass-Pachinko.git
cd NBD-Bass-Pachinko
```

### Step 2: 準備済みファイル一式をコピー

PowerShell で：

```powershell
$src = "C:\Users\kimse\OneDrive\ドキュメント\Claude\Projects\パチンコ市場\NBD-Bass-Pachinko"
$dst = "C:\dev\NBD-Bass-Pachinko"
robocopy $src $dst /E /XD .git
```

または手動で以下を `$src` から `$dst` へコピー：

- `README.md`, `LICENSE`, `LICENSE-PAPER`, `CITATION.cff`, `.gitignore`, `Project.toml`
- `paper/` フォルダ（中身）
- `notebooks/` フォルダ（中身）
- `data/` フォルダ（中身）
- `docs/` フォルダ（中身）

### Step 3: コミット & push

```powershell
cd C:\dev\NBD-Bass-Pachinko
git add -A
git status   # 確認

git commit -m "Initial commit: NBD-Bass joint Bayesian model for pachinko market

- Full paper (Japanese, ~35k chars) in paper/paper.md and paper/paper.html
- 6 Julia notebooks: standard Bass, NBD, 2-stage, joint, regulation, time-varying M
- Data: leisure white paper aggregates and frequency survey (2008-2020)
- License: MIT (code) + CC BY 4.0 (paper)
- See README.md for overview and results"

git push origin main
```

初回 push 時にユーザー名・パスワード（または Personal Access Token）を求められます。

## 認証について

- **HTTPS + PAT 認証**（推奨）：GitHub Settings → Developer settings → Personal access tokens で `repo` スコープのトークンを発行し、`git push` 時のパスワード欄に入力
- **SSH 認証**：`git@github.com:kimseok1973/NBD-Bass-Pachinko.git` を remote にして公開鍵で push
- **GitHub CLI**：`gh auth login` で事前認証してから `gh repo clone` → `git push`

## OneDrive 内で直接 push する場合（非推奨だが可能）

OneDrive の sync が一時停止できれば、OneDrive 内でも git は使えます：

```powershell
# OneDrive を一時停止
# システムトレイの OneDrive アイコン → 同期一時停止 → 2時間など

cd "C:\Users\kimse\OneDrive\ドキュメント\Claude\Projects\パチンコ市場\NBD-Bass-Pachinko"
git init -b main
git remote add origin https://github.com/kimseok1973/NBD-Bass-Pachinko.git
git add -A
git commit -m "Initial commit"
git push -u origin main

# 完了後、OneDrive 同期を再開
```

注意点：

- `.git/` フォルダは大量の小ファイルを含み、OneDrive 同期で衝突しやすい
- 同期中に lock file (`.git/index.lock`) が消えると `git status` 等がエラーになる
- 大規模なリポジトリでは pack ファイルの再生成に時間がかかる

確実なのは「OneDrive 外で作業 + push」のフローです。

## 公開後の追加 push

以降の更新は通常通り：

```powershell
git add -A
git commit -m "Update: <変更内容>"
git push
```

## チェックリスト

push 前に以下を確認することを推奨します：

- [ ] `README.md` の冒頭バッジが意図通り表示される
- [ ] `paper/paper.html` をブラウザで開いて数式・図がレンダリングされる
- [ ] `notebooks/04_Pachinko_NBD_Bass_Joint.ipynb` を Jupyter で開いて出力セルが残っている
- [ ] `data/p-datasets2.csv` の文字エンコーディング (UTF-8) と数値が正しい
- [ ] `LICENSE` と `LICENSE-PAPER` の著作権者名が正しい
- [ ] `.gitignore` で必要なファイルが除外されていない（特に `Project.toml` は含めること）

完了後、GitHub のリポジトリ Settings → About でトピックタグ（`bayesian-inference`, `diffusion-models`, `julia`, `nbd`, `pachinko` 等）を追加すると発見されやすくなります。
