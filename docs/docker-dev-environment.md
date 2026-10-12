# Docker 開発環境の構築計画（2026-10-12）

ホストPCに Node.js プロジェクトを直接作らず、アプリもDBも Docker Compose 上で動かして開発する。
ファイルはユーザーが自分で書く。この資料は考えるポイントと完了条件のみを示す。

## 前提の整理

- Next.js は1つのプロジェクトに画面（フロントエンド）と API Routes（バックエンド）が同居する。
  そのため「フロント用・バック用」でコンテナを分ける必要はなく、**`web` と `db` の2サービス**で始める。
- ソースコードはホストの `apps/web/` に置き、コンテナにマウントする（エディタはホスト、実行はコンテナ）。

## 目標の構成（案A：Next.js 1つ＋内部で層を分ける。2026-10-12 決定）

```
MyBookLibrary/
├── docker-compose.yml        # web と db の2サービス
├── .env / .env.example       # 作成済み
├── docs/
└── apps/
    └── web/                  # Next.js（画面もAPIもここ）
        ├── Dockerfile        # 開発用（Node.js イメージ）
        ├── package.json
        ├── drizzle.config.ts # 課題2で追加
        ├── public/
        └── src/
            ├── app/          # ルーティング（page.tsx, route.ts, layout.tsx）
            ├── components/   # 画面部品
            ├── server/       # サーバー専用コード（db/, services/）
            ├── lib/          # 共通ユーティリティ
            └── types/
```

- 「バックエンド」は `src/server/` と `src/app/api/` で表す。別サーバーの `apps/api/` は作らない。
- 将来 `apps/api/` や `packages/shared/` を足したくなったら、`apps/` 配下なので移動せず追加できる。
- `src/` 以下の `app/` と `public/` は `create-next-app`（`--src-dir` を指定）で自動生成される。`components/` `server/` `lib/` `types/` は自分で作る。
- サービス名は `web` と `db` にそろえる。。
- 公式ドキュメント（Project structure and organization）と、作成時に最新内容を突き合わせること。

## 課題0-1：PostgreSQL を起動する

考えるポイントと完了の確認は `original-project-survey.md` 7章「Docker Compose の方針」。

## 課題0-2：Next.js 用の Dockerfile と web サービス

### 考えるポイント（上から順に）

1. ベースイメージ：Node.js のどの版・どの種類（`node:24` / `node:24-slim` / `node:24-alpine`）にするか。違いは何か。
2. `WORKDIR` をどこにするか。
3. 開発用なので、ソースは `COPY` せずボリュームでマウントする。では Dockerfile に何を書けば十分か。
4. `node_modules` の置き場所：ホストの `apps/web/` をマウントすると、コンテナ内の `node_modules` が隠れる・Windows とLinux でネイティブモジュールが混ざる問題がある。名前付きボリュームで分ける方法を調べる。
5. ポート：コンテナの 3000 をホストに公開する。`next dev` はコンテナ外から見えるよう待ち受け先（`0.0.0.0`）を意識する。
6. ファイル変更の検知：Windows ホストからのマウントでは変更が伝わらずホットリロードが効かないことがある。`WATCHPACK_POLLING` などの対処を調べる。
7. DB への接続：コンテナ内から見た DB のホスト名は `localhost` ではなく**サービス名**になる。`.env` の `DATABASE_URL` をどう扱うか考える（ホスト用とコンテナ用で異なる）。
8. 起動順：`depends_on` と `db` の `healthcheck` を組み合わせる。

### 鶏と卵の問題

Next.js プロジェクトはまだ無い。ホストに Node を使わず作るには、
`docker compose run --rm web npx create-next-app@<版> .` のように**コンテナ内で作成する**方法がある。
その時点では `next dev` を起動コマンドにできないので、作成前と作成後で起動コマンドをどうするか考える。

### 完了条件

- `docker compose up -d` で `web` と `db` の両方が起動し、`docker compose ps` で確認できる。
- ブラウザで http://localhost:3000 に Next.js の初期画面が表示される。
- ホストで `apps/web/` 内のページを編集すると、ブラウザに反映される。
- `web` コンテナ内から `db` に接続できる（例：`docker compose exec web` から `pg_isready` 相当、または後の課題で Drizzle から接続）。
- 生成された `apps/web/` のファイルに `node_modules` がホスト側に作られていない、または `.gitignore` で除外されている。
- Dockerfile の各行と compose の各項目が「何のためにあるか」を説明できる。

### 答え合わせ

元のアプリの `app/Dockerfile`（元は app/ 構成） は**本番用**（ビルドして実行）で、今回の**開発用**とは目的が違う。違いを比較すると学習になる。

## 未決事項

- Node.js のイメージ版（ホストは v24.19.0。元のアプリの要件は 22 以上）。
- Next.js の版（元のアプリは 16.2.6）。
