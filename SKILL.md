---
name: git-rebase
description: 現在のブランチを指定したブランチの上に rebase する。競合時は git-resolve-conflicts に委譲する。ユーザーが「rebaseして」「mainの上に乗せ替えて」「最新のmainに追従したい」など、ブランチの再構成や rebase を求めている場合は必ずこのスキルを使う。
---

# Skill: git-rebase

## 引数

`git-rebase <target>` の形で呼ばれる。`<target>` の解釈:

- `main` のようなローカルブランチ名 → そのままローカルの `<target>` を rebase 先にする。
- `origin/main` のようなリモート追跡ブランチ名 → fetch 済みである前提でそのまま使う。**このスキル自身は `git fetch` を実行しない**。ローカルの `origin/main` が古い可能性がある場合は、実行前にその旨をユーザーに一言警告する（自動 fetch はしない）。

`<target>` が省略された場合は何を rebase 先にするか確認する。

## 前提条件（実行前に必ず確認）

- `git status` を実行し、作業ツリーに未コミット/未ステージの変更がないか確認する。
- 変更がある場合は rebase を開始せず、ユーザーに commit か stash を促す（無断で stash/commit しない）。

## 手順

1. `git status` で作業ツリーがクリーンであることを確認する。
2. `<target>` が `origin/...` のようなリモート追跡ブランチ名の場合、fetch はせず、ローカルの当該ref が古い可能性がある旨を実行前に一言警告する。
3. `git rebase <target>` を実行する。
4. 結果を判定する。
   - 成功: `git log --oneline -5` で確認して報告し、終了。
   - コンフリクト発生: git-resolve-conflicts を実行し、解決後に自動で `git rebase --continue` まで完了させる。
5. rebase 完了後、`git status --short` で作業ツリーの最終状態を確認する。

## 禁止事項

- `git rebase --abort` をユーザーの明示的指示なく実行すること。
- rebase 対象ブランチが `main` や共有ブランチ本体である場合（＝自分が rebase される側になる操作）を無断で行うこと。このスキルは常に「現在のブランチを動かす」側であることを確認する。
- 未 push のコミットが書き換わることについて警告を省略すること。

## 出力

- rebase 前に「どのブランチをどこに乗せ替えるか」を一言明示する。
- 完了後、コミットグラフの要約と、push が必要な場合は force push が必要になる旨を報告する。
