# 授業用共有PCでのGit・GitHub利用ガイド

対象: 複数人で共有するPC(実習室・貸出PCなど)でGit/GitHubを使う受講者
方針: `git config --global` は使用しない(自分の設定が次に使う人に残らないようにするため)
前提: GitHub上にリポジトリを作成済みであること(リポジトリの新規作成手順は本ガイドでは扱わない)

---

## 1. なぜ `git config --global` を使わないのか

`git config --global` はPC全体(そのOSユーザー)の設定ファイル(`~/.gitconfig`)に書き込まれる。共有PCで一度でも使うと、

- 自分の名前・メールアドレスが他の受講者のコミットに紛れ込む
- 次に使う人が誤って自分のアカウントでpushしてしまう
- 認証情報(トークンや鍵)が他人からアクセス可能な状態で残る

といった事故が起きる。そのため本ガイドでは、設定は必ず**リポジトリ単位(ローカル設定)**または**その場限り(環境変数・一時ファイル)**で行う。

---

## 2. 事前確認

```bash
git --version
```

Gitが入っていない場合は学校事務局に確認する(学生個人の権限でインストールできない環境が多いため)。

---

## 3. リポジトリの準備(git init / git clone)

ローカル設定(4章)は「`.git` フォルダを持つリポジトリ」に対して行うものなので、その前にリポジトリを用意する必要がある。すでにGitHub上にあるものを使うか、新規に作るかで手順が異なる。

### 3-1. 既存のリポジトリを取得する場合(clone)

課題用リポジトリなど、GitHub上に既にあるものを使う場合はcloneする。

```bash
cd 作業したいフォルダ
git clone https://github.com/ユーザー名/リポジトリ名.git
cd リポジトリ名
```

認証が必要な場合は4章の認証方式(HTTPS+PATまたはSSH)を先に確認してから実行する。cloneが終わったら、リポジトリ直下(`.git` フォルダがある場所)にいる状態で4章のローカル設定に進む。

### 3-2. 新規にリポジトリを作る場合(init)

手元でゼロからプロジェクトを始める場合は次の手順でリポジトリ化する。

```bash
mkdir my-app
cd my-app
git init
```

`git init` を実行した時点でそのフォルダに `.git` フォルダが作られ、リポジトリとして扱えるようになる。GitHub側にも同じ名前のリポジトリを作成し、リモートとして登録する場合は次を実行する。

```bash
git remote add origin https://github.com/ユーザー名/リポジトリ名.git
```

`git init` した直後はローカル設定(4章)がまだ何も入っていない状態なので、次に必ず4章の手順を行う。

**初回pushの手順(PATを使う場合)**

新規リポジトリはリモート登録しただけではまだ何もGitHub側に送られていない。最初のpushまで行って初めて履歴ができる。あらかじめ4章のローカル設定と5-1のPAT発行を済ませたうえで、次を実行する。

```bash
# Credential Managerに保存しない設定(共有PCでは必須。5-1(1)参照)
git config credential.helper ""

git add .
git commit -m "Initial commit"

# 初回のpush(-u で追跡ブランチを設定)
git push -u origin main
```

ユーザー名を聞かれたらGitHubのユーザー名、パスワードを聞かれたらPAT(5-1(2)で発行したもの)を貼り付ける。デフォルトブランチが `main` でなく `master` の場合は読み替える。2回目以降は `git push` だけでよい。

SSH鍵を使う場合は先にリモートURLをSSH形式にする(5-2参照)。

---

## 4. リポジトリごとのユーザー設定(ローカルconfig)

`--global` を付けずに設定すると、そのリポジトリの `.git/config` にのみ書き込まれる。3章でclone/initしたリポジトリのフォルダに移動してから実行する。

```bash
cd 作業したいリポジトリのフォルダ
git config user.name "自分の名前"
git config user.email "自分のメールアドレス"
```

確認:

```bash
git config --list --local
```

このリポジトリを消せば設定も消える。他の受講者のリポジトリには影響しない。`git clone` や `git init` をやり直す(=新しい `.git/config` になる)たびに、この設定も再度必要になる点に注意。

---

## 5. GitHubへの認証方式

共有PCでは「認証情報を保存しない」ことが最優先。方式ごとの手順と後始末をまとめる。

### 5-1. HTTPS + Personal Access Token(PAT)

もっとも手軽。PCにログイン情報を保存しない設定と組み合わせて使う。

**(1) Credential Managerを使わない設定にする(リポジトリ単位)**

```bash
cd 作業リポジトリ
git config credential.helper ""
```

これで認証情報がOSのキーチェーンやWindows資格情報マネージャーに保存されなくなる(毎回入力が必要になる代わりに安全)。

**(2) GitHub側でPATを発行**

リポジトリのSettingsではなく、GitHubアカウント全体の個人設定から発行する。画面右上の自分のアイコン → Settings(特定のリポジトリを開いている状態のSettingsタブではない)→ 左メニュー下部の Developer settings → Personal access tokens → Fine-grained tokens で発行。

発行画面で特に設定する項目:

- 有効期限(Expiration): 「授業期間のみ」など短く設定する
- Repository access: トークンがアクセスできるリポジトリの範囲。「All repositories」(全リポジトリ)ではなく「Only select repositories」を選び、使用するリポジトリだけにチェックする
- Permissions: トークンに与える操作権限。リポジトリ内の「Contents」を「Read and write」にすればclone/push/pullに必要な権限は足りる(それ以外の項目は既定の「No access」のままでよい)

権限を必要最小限にしておくと、万が一トークンが流出しても被害を抑えられる。

**(3) push/pull時にユーザー名とPATを入力**

```bash
git clone https://github.com/ユーザー名/リポジトリ名.git
```

ユーザー名を聞かれたらGitHubのユーザー名、パスワードを聞かれたら**PATを貼り付け**(通常のパスワードはHTTPS認証では使えない)。

**(4) 授業終了後の後始末**

```bash
git config --unset credential.helper
```

万が一保存されてしまった場合は、OSの資格情報マネージャー(Windows: 資格情報マネージャー、Mac: キーチェーンアクセス)から `github.com` の項目を検索して削除する。GitHub側でもトークンをRevokeしておくと安全。

### 5-2. SSH鍵

鍵をPCに残さない/残しても他人が使えない運用にする。

**(1) その場限りの鍵ペアを作成(デフォルトの `~/.ssh` ではなく作業フォルダ内に作る)**

```bash
cd 作業リポジトリ
mkdir -p .sshkey
ssh-keygen -t ed25519 -f .sshkey/id_ed25519 -C "授業用" -N ""
```

`~/.ssh/id_ed25519` に置くとPC全体・他ユーザーからも見える可能性があるため、リポジトリ内の専用フォルダに保存する(`.gitignore` に追加して誤コミットを防ぐこと)。

**(2) 公開鍵をGitHubに登録**

```bash
cat .sshkey/id_ed25519.pub
```

内容をコピーし、これもリポジトリのSettingsではなくアカウント全体の個人設定から登録する。右上の自分のアイコン → Settings → SSH and GPG keys → New SSH key に貼り付け。

**(3) このリポジトリだけがその鍵を使うよう設定**

```bash
git config core.sshCommand "ssh -i .sshkey/id_ed25519 -o IdentitiesOnly=yes"
```

`~/.ssh/config` を書き換えないので、他のリポジトリやOSの設定に影響しない。

**(4) 動作確認**

```bash
git remote set-url origin git@github.com:ユーザー名/リポジトリ名.git
git fetch
```

**(5) 授業終了後の後始末(重要)**

```bash
rm -rf .sshkey
```

そしてGitHub側でも Settings → SSH and GPG keys から該当の鍵をDeleteする。鍵ファイルを消してもGitHub側に登録が残っていると使われ続けるため、必ずセットで削除する。

---

## 6. 基本的な作業の流れ

```bash
# 変更を確認
git status

# ステージング
git add ファイル名
# すべて追加する場合
git add .

# コミット
git commit -m "コミットメッセージ"

# リモートへ反映
git push origin ブランチ名

# 最新を取得
git pull origin ブランチ名
```

コミット前に必ず `git config --list --local` でuser.name/user.emailが自分のものになっているか確認する習慣をつけると事故が減る。

---

## 7. 授業終了時のチェックリスト

| 項目 | 確認方法 | 対応 |
|---|---|---|
| ローカルconfigに自分の情報が残っていないか | リポジトリごと削除するなら不要。残す場合は `git config --list --local` | 不要なら `git config --unset user.name` など |
| Credential Managerに認証情報が残っていないか | OSの資格情報マネージャー/キーチェーンで `github.com` を検索 | 見つかったら削除 |
| SSH鍵ファイルが残っていないか | 作業フォルダ内の `.sshkey` など | `rm -rf` で削除 |
| GitHub側にPAT/SSH鍵の登録が残っていないか | GitHub Settings → Developer settings / SSH and GPG keys | Revoke / Delete |
| ブラウザにGitHubのログインセッションが残っていないか | ブラウザ | ログアウト、可能ならプライベートウィンドウを使用 |

---

## 8. よくあるトラブル

**Q. `git commit` で `Please tell me who you are` と出る**
A. そのリポジトリでローカル設定(4章)をしていない。`git config user.name` / `user.email` を実行する。

**Q. `--global` を付けていないのにエラーが出ない/設定できてしまう**
A. 誰か(または自分)が過去に `--global` で設定済みの可能性がある。`git config --list --global` で確認し、共有PCであれば管理者に相談の上、該当設定の削除を検討する。

**Q. pushしようとすると別人のアカウントで認証されてしまう**
A. Credential Managerに他人の認証情報が残っている。5-1(4)の手順でOSの資格情報マネージャーを確認する。

---

以上。共有PCでは「自分のリポジトリフォルダの中だけで完結させ、使い終わったら痕跡を消す」ことを徹底する。
