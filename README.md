# Qiita 記事管理リポジトリ

このリポジトリは [Qiita CLI](https://github.com/increments/qiita-cli)（`@qiita/qiita-cli`）を使って、Qiitaの記事をGitで管理・公開するためのものです。

> **注意:** 以下の `npx qiita ...` コマンドは、必ずこの `qiita-articles` ディレクトリ内で実行してください。`@qiita/qiita-cli` はこのディレクトリの `node_modules` にローカルインストールされているため、親ディレクトリなど別の場所で実行すると `npm error could not determine executable to run` のようなエラーになります。
>
> ```sh
> cd qiita-articles
> npx qiita login
> ```

## ディレクトリ構成

- `public/` … 記事のMarkdownファイルを格納するディレクトリ
  - `.remote/` … Qiita上の最新状態をキャッシュするディレクトリ（CLIが自動管理。手動編集しない）
- `qiita.config.json` … Qiita CLIの設定ファイル（プレビュー用のhost/port等）
- `.github/workflows/publish.yml` … `main`/`master` ブランチへのpushをトリガーに記事を自動公開するGitHub Actions

## ファイル命名規則

記事ファイルは `public/` 配下に、**2桁の連番 + `.` + 識別用の名前**で保存する。

```
01.6014ed6748a28fc1b4dd.md
02.ses-engineer-future.md
```

- 連番は執筆・公開順
- 連番以降の名前は内容が分かる英語スラッグでよい（Qiita側のIDとは無関係）
- ファイル名とQiita記事ID（フロントマターの`id`）は別管理。ファイル名を変更してもQiita側の記事との紐付けは保たれる

## 事前準備

```sh
npx qiita login
```

初回のみ、Qiitaのアクセストークンを使ってCLIの認証を行う。

## 新規記事を書く手順

1. 記事ファイルを作成する

   ```sh
   npx qiita new <basename>
   ```

   `public/<basename>.md` が生成される（フロントマターは `title`/`tags`/`private`/`id: null` 等の空テンプレート）。

2. 既存ファイルの連番に続く番号でファイル名をリネームする

   ```sh
   mv public/<basename>.md public/03.<basename>.md
   ```

   ※ `qiita new` に連番オプションはないため、生成後に手動でリネームする。

3. フロントマター（`title` / `tags` / `private`）と本文を編集する

4. ローカルでプレビュー確認（任意）

   ```sh
   npx qiita preview
   ```

   `http://localhost:8888` でブラウザプレビューできる。

## Qiitaへの記事反映（公開・更新）

### 方法A: CLIから直接公開

```sh
# 特定の記事のみ公開・更新
npx qiita publish <basename>

# 変更があった記事をすべて公開・更新
npx qiita publish --all
```

- フロントマターの `id` が `null` の記事 → 新規投稿（POST）し、発行されたIDが自動的に `id` に書き込まれる
- `id` が入っている記事 → 更新（PATCH）

### 方法B: Gitにpushして自動反映（推奨）

[.github/workflows/publish.yml](.github/workflows/publish.yml) により、`main`/`master` ブランチへのpushをトリガーにGitHub Actionsが自動で `publish` 相当の処理を実行する。

```sh
git add public/03.<basename>.md
git commit -m "記事追加: <タイトル>"
git push
```

- 事前にリポジトリのSecretsに `QIITA_TOKEN` を設定しておく必要がある
- 新規記事の場合、CI側で発行されたIDがコミットされる（リポジトリ側に書き戻される）

## Qiita上の記事をpull（同期）する

Qiita側で直接編集した内容や、他環境で公開した記事をローカルに取り込む場合。

```sh
npx qiita pull
```

- Qiita上の全記事を取得し、`public/` 配下のファイルを最新化する
- ローカルとQiita側で内容が異なる場合は `.remote/` のキャッシュと比較して同期される

## よく使うコマンド一覧

| コマンド | 用途 |
| --- | --- |
| `npx qiita login` | Qiita APIの認証 |
| `npx qiita new <basename>` | 新規記事ファイルの作成 |
| `npx qiita preview` | ローカルプレビュー |
| `npx qiita publish <basename>` | 指定記事の公開・更新 |
| `npx qiita publish --all` | 変更があった記事をすべて公開・更新 |
| `npx qiita pull` | Qiita上の記事をローカルに同期 |
