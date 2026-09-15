# Git 基礎コマンド

## バージョン確認
```bash
git --version
```
## 初期化
```bash
git init
```
## config
```bash
git config --local user.name "自分の名前"
git config --local user.email "自分のemail"
git config -l
```
表示の最終まで来たら'q'で終了

## 状態を見る
```bash
git status
```
## ステージに上げる
```
git add ファイル名
git add .
```
`git add .`で全てのファイルが対象

## コミットする(状態を登録するよ)
```bash
git commit -m "コミットメッセージ"
```

## 差分を見る
```bash
git diff
```

## ログを見る
```bash
git log
```

## 分岐
一覧
```bash
git branch
```

新しいブランチを作成してスイッチ
```bash
git switch -c "develop"
```

ブランチを移動
```bash
git switch "master"
```

## マージ
取り込むブランチに移動
```bash
git merge 取り込みたいブランチ
```

## ブランチ削除
使用済のブランチを削除する
```bash
git branch -d 削除したいブランチ
```

## 変更を取り消す
```bash
git restore ファイル名
```

## ステージングを取り消す
```bash
git restore --staged ファイル名
```

## コミットを取り消す
コミットのみ
```bash
git reset --soft HEAD^
```
コミット+ステージング
```bash
git reset HEAD^
```
変更が消えない

コミット+ステージング+変更
```bash
git reset --hard HEAD^
```
## コミットを遡る
```bash
git switch --detach コミットID
```
現在のコミットに戻る
```bash
git switch master
```
