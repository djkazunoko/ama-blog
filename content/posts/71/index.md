---
date: '2022-11-06T21:35:41+09:00'
draft: false
title: '【Git】git reset --soft、--mixed、--hardで変更を取り消す'
tags: ["未分類"]
slug: '71'
---

## 1. git reset

```
git reset [<mode>] [<commit>]
```

- `mode`
  - `--soft`：HEADの移動
  - `--mixed`：HEADの移動、インデックスの更新
  - `--hard`：HEADの移動、インデックスの更新、作業ディレクトリの更新
  - デフォルトで`--mixed`が指定される
- `commit`
  - 巻き戻したいcommitを指定
  - デフォルトでHEADが指定される

## 2. git resetの各モードについて

### 2-1. --soft (HEADの移動)

```
git reset --soft HEAD~
```
- 直近の`git commit`を取り消す

### 2-2. --mixed (インデックスの更新)

```
git reset --mixed HEAD~
```
- 直近の`git commit`と`git add`を取り消す

### 2-3. --hard (作業ディレクトリの更新)

```
git reset --hard HEAD~
```
- 直近の`git commit`と`git add`と作業ディレクトリの変更を取り消す

## 3. まとめ

- `--soft` ：HEAD が指し示すブランチを移動する (`--soft` を使うと処理はここで終了)
- `--mixed`：`--soft` + インデックスの内容を HEAD と同じにする (`--mixed`を使うと処理はここで終了)
- `--hard`：`--soft` + `--mixed` + 作業ディレクトリの内容をインデックスと同じにする


---

【参考】

- [Git - git-reset](https://git-scm.com/docs/git-reset#Documentation/git-reset.txt-emgitresetemltmodegtltcommitgt)
- [Git reset | アトラシアンの Git チュートリアル](https://www.atlassian.com/ja/git/tutorials/undoing-changes/git-reset)
- [Git - リセットコマンド詳説](https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%81%95%E3%81%BE%E3%81%96%E3%81%BE%E3%81%AA%E3%83%84%E3%83%BC%E3%83%AB-%E3%83%AA%E3%82%BB%E3%83%83%E3%83%88%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E8%A9%B3%E8%AA%AC)
- [第6話 git reset 3種類をどこよりもわかりやすい図解で解説！【連載】マンガでわかるGit ～コマンド編～ - itstaffing エンジニアスタイル](https://www.r-staffing.co.jp/engineer/entry/20191129_1)
