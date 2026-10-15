# FriWorkshop

テーマ0「GitとGitHubの基本・演習」の作業リポジトリです．各課題の作業内容と証跡（コマンドと実行結果）を以下にまとめます．

| 課題 | 内容 | 状態 |
|---|---|---|
| 00 | Gitのインストールと初期設定 | ⏳ |
| 01 | リポジトリの初期化と最初のコミット | ✅ |
| 02 | GitHubアカウント・2FA・SSHキー登録 | ⏳ |
| 03 | ローカルとGitHubリポジトリの連携 | ⏳ |
| 04 | ファイルの変更とpush/pull | ⏳ |
| 05 | .gitignoreの作成 | ✅ |
| 06 | ブランチの作成と切り替え | ✅ |
| 07 | Pull Requestの作成 | ⏳ |
| 08 | コミット履歴と差分の確認 | ✅ |
| 09 | コンフリクトの発生と解消 | ✅ |

---

## 00. Gitのインストールと初期設定

> ⏳ 未完了：Mac上での実行・GitHub上での操作後に追記します。

---

## 01. リポジトリの初期化と最初のコミット

`git init` でリポジトリを作り，`README.md` を作成して最初のコミットを行った．名前とメールアドレスはGitHubの公開用アドレス（noreply）を使用している．

```
$ git init -b main
Initialized empty Git repository in /sessions/rcw-01bhfdlqxcrz8fdbtclfq6zt/mnt/FriWorkshop/.git/
$ git config user.name generalLiebe
$ git config user.email 130543432+generalLiebe@users.noreply.github.com
$ echo "# FriWorkshop" > README.md
$ git add README.md
$ git commit -m "first commit: README.mdを追加"
[main (root-commit) 18a82f1] first commit: README.mdを追加
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
$ git log --oneline
18a82f1 first commit: README.mdを追加
$ git rev-parse HEAD
18a82f122160443140e858603009802b8bc033e8
```

**最初のコミットID：** `18a82f122160443140e858603009802b8bc033e8`

---

## 02. GitHubアカウント・2段階認証・SSHキー登録

> ⏳ 未完了：Mac上での実行・GitHub上での操作後に追記します。

---

## 03. ローカルリポジトリとGitHubリポジトリの連携

> ⏳ 未完了：Mac上での実行・GitHub上での操作後に追記します。

---

## 04. ファイルの変更とpush/pull

> ⏳ 未完了：Mac上での実行・GitHub上での操作後に追記します。

---

## 05. .gitignoreファイルの作成

### .gitignoreの内容

```gitignore
# 依存パッケージ（npm install で再生成できる・容量が大きい）
node_modules/

# 環境変数・秘密情報（APIキーやパスワードが漏れる）
.env
.env.*
!.env.example

# macOSが自動生成するメタデータ（作業内容と無関係）
.DS_Store

# ログ・ビルド成果物（実行のたびに変わる／再生成できる）
*.log
dist/
```

### 無視する理由

| パターン | 理由 |
|---|---|
| `node_modules/` | `npm install` でいつでも再生成でき，容量も非常に大きいため． |
| `.env`（`.env.*`） | APIキーやパスワードなどの秘密情報が入るため．公開すると漏洩する．（ひな形の `.env.example` だけは共有する） |
| `.DS_Store` | macOSが自動生成するフォルダ表示用のメタデータで，作業内容と無関係なため． |
| `*.log`，`dist/` | 実行やビルドのたびに生成・変化するもので，ソースから再生成できるため． |

### 反映と確認

ダミーの `node_modules/`・`.env`・`.DS_Store` を置いた状態で確認した．`!!` は「無視されている」ことを示す．コミット後の `git status` はクリーンである．

```
$ git status --short --ignored
?? .gitignore
!! .DS_Store
!! .env
!! node_modules/
$ git add .gitignore
$ git commit -m "chore: .gitignoreを追加"
[main ce6cf89] chore: .gitignoreを追加
 1 file changed, 14 insertions(+)
 create mode 100644 .gitignore
$ git status
On branch main
nothing to commit, working tree clean
$ git check-ignore -v node_modules/dummy.js .env .DS_Store
.gitignore:2:node_modules/	node_modules/dummy.js
.gitignore:5:.env	.env
.gitignore:10:.DS_Store	.DS_Store
```

---

## 06. ブランチの作成と切り替え

`develop` ブランチを作成・切り替えし，自己紹介ページ `practice/profile.md` を追加してコミットした．`main` に戻ると `practice/` が存在しないことから，変更が `develop` ブランチだけにあることを確認できる．

```
$ git branch
* main
$ git switch -c develop
Switched to a new branch 'develop'
$ git add practice/profile.md
$ git commit -m "feat: 自己紹介ページ(practice/profile.md)を追加"
[develop feb1f50] feat: 自己紹介ページ(practice/profile.md)を追加
 1 file changed, 4 insertions(+)
 create mode 100644 practice/profile.md
$ git branch
* develop
  main
$ git switch main
Switched to branch 'main'
$ git branch
  develop
* main
$ ls practice
ls: cannot access 'practice': No such file or directory
$ git switch develop
Switched to branch 'develop'
$ git log --oneline --all --graph
* feb1f50 feat: 自己紹介ページ(practice/profile.md)を追加
* ce6cf89 chore: .gitignoreを追加
* 18a82f1 first commit: README.mdを追加
```

**変更内容：** `practice/profile.md`（GitHubユーザー名とワークショップの目標を書いた自己紹介ページ）を新規追加．このブランチは課題07でPRを通して `main` にマージする．

---

## 07. Pull Request（PR）の作成

> ⏳ 未完了：Mac上での実行・GitHub上での操作後に追記します。

---

## 08. 過去のコミット履歴の確認と差分の確認

- `git log --oneline`：コミットを1行ずつ表示
- `git diff`：作業ツリーの未コミットの変更を表示
- `git diff <コミットA> <コミットB>`：2つのコミット間の差分を表示
- `git diff main develop --stat`：ブランチ間で変わったファイルの一覧

```
$ git log --oneline
105ac95 merge: feature/englishを取り込み、日英併記でコンフリクトを解消
43c128f feat: あいさつに「世界」を追加
732fea6 feat: あいさつを英語にする
48b84f9 feat: あいさつファイルを追加
ce6cf89 chore: .gitignoreを追加
18a82f1 first commit: README.mdを追加
$ git log --oneline --all --graph
*   105ac95 merge: feature/englishを取り込み、日英併記でコンフリクトを解消
|\  
| * 732fea6 feat: あいさつを英語にする
* | 43c128f feat: あいさつに「世界」を追加
|/  
* 48b84f9 feat: あいさつファイルを追加
| * feb1f50 feat: 自己紹介ページ(practice/profile.md)を追加
|/  
* ce6cf89 chore: .gitignoreを追加
* 18a82f1 first commit: README.mdを追加
$ git diff
diff --git a/practice/greeting.md b/practice/greeting.md
index 2837bf3..8b22570 100644
--- a/practice/greeting.md
+++ b/practice/greeting.md
@@ -1,3 +1,4 @@
 # あいさつ
 
 メッセージ: こんにちは、世界！ / Hello, world!
+練習用の追記です。
$ git diff --stat
 practice/greeting.md | 1 +
 1 file changed, 1 insertion(+)
$ git restore practice/greeting.md
$ git diff 48b84f9 105ac95
diff --git a/practice/greeting.md b/practice/greeting.md
index 4b49a80..2837bf3 100644
--- a/practice/greeting.md
+++ b/practice/greeting.md
@@ -1,3 +1,3 @@
 # あいさつ
 
-メッセージ: こんにちは
+メッセージ: こんにちは、世界！ / Hello, world!
$ git diff main develop --stat
 practice/greeting.md | 3 ---
 practice/profile.md  | 4 ++++
 2 files changed, 4 insertions(+), 3 deletions(-)
```

---

## 09. コンフリクトの発生と解消

### 手順

1. `main` に `practice/greeting.md`（`メッセージ: こんにちは`）を作成．
2. `feature/english` ブランチで同じ行を `Hello, world!` に変更してコミット．
3. `main` で同じ行を `こんにちは、世界！` に変更してコミット．
4. `main` に `feature/english` をマージ → コンフリクト発生．

```
$ git switch -c feature/english
Switched to a new branch 'feature/english'
$ sed -i 's/メッセージ: こんにちは/メッセージ: Hello, world!/' practice/greeting.md
$ git commit -am "feat: あいさつを英語にする"
[feature/english 732fea6] feat: あいさつを英語にする
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git switch main
Switched to branch 'main'
$ sed -i 's/メッセージ: こんにちは/メッセージ: こんにちは、世界！/' practice/greeting.md
$ git commit -am "feat: あいさつに「世界」を追加"
[main 43c128f] feat: あいさつに「世界」を追加
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git merge feature/english
Auto-merging practice/greeting.md
CONFLICT (content): Merge conflict in practice/greeting.md
Automatic merge failed; fix conflicts and then commit the result.
```

### コンフリクト発生時の `git status` とファイル内容

```
$ git status
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   practice/greeting.md

no changes added to commit (use "git add" and/or "git commit -a")
$ cat practice/greeting.md
# あいさつ

<<<<<<< HEAD
メッセージ: こんにちは、世界！
=======
メッセージ: Hello, world!
>>>>>>> feature/english
```

### コンフリクトマーカーの意味

| マーカー | 意味 |
|---|---|
| `<<<<<<< HEAD` | ここから下が現在のブランチ（`main`）側の内容 |
| `=======` | 2つの変更の区切り |
| `>>>>>>> feature/english` | ここより上が取り込もうとしたブランチ（`feature/english`）側の内容 |

### 解消

両方の意図を残すため，日本語と英語を併記する形に手作業で書き換え，マーカーをすべて削除してからコミットした．

```
$ cat practice/greeting.md
# あいさつ

メッセージ: こんにちは、世界！ / Hello, world!
$ git add practice/greeting.md
$ git commit -m "merge: feature/englishを取り込み、日英併記でコンフリクトを解消"
[main 105ac95] merge: feature/englishを取り込み、日英併記でコンフリクトを解消
$ git status
On branch main
nothing to commit, working tree clean
$ git log --graph --oneline
*   105ac95 merge: feature/englishを取り込み、日英併記でコンフリクトを解消
|\  
| * 732fea6 feat: あいさつを英語にする
* | 43c128f feat: あいさつに「世界」を追加
|/  
* 48b84f9 feat: あいさつファイルを追加
* ce6cf89 chore: .gitignoreを追加
* 18a82f1 first commit: README.mdを追加
```
