# 元のMyBookLibrary 調査結果（2026-10-12）

引き継ぎメモ（`handoff-to-claude.md`）の依頼1〜6に対する調査記録。
元のプロジェクト（`C:\web_application_files\MyBookLibrary`）は読み取りのみで、ファイルは変更していない。
`.env` 系ファイルの中身は開いていない。

## 1. 適用される作業ルール

| 場所 | ファイル | 内容 |
| --- | --- | --- |
| `~/.claude/CLAUDE.md` | グローバル | テストデータに実データ禁止、秘密情報をコミット対象に書かない、ログイン画面はパスワード表示トグル＋確認欄 |
| `analysis/`（親） | なし | AGENTS.md・CLAUDE.md とも**存在しない** |
| 学習用 `MyBookLibrary/` | AGENTS.md / CLAUDE.md | AIはヒント・説明・レビュー担当。コードは明示依頼なしに編集しない |
| 元プロジェクト | CLAUDE.md（正）/ AGENTS.md（同期版） | Issue駆動、`main`直接push禁止、ブランチ名 `<prefix>/#<番号>-<概要>`、`npm run check`、Codexレビュー |

元プロジェクトのIssue駆動ルールは学習用には自動で適用されない。学習用で採用するかは自分で決める。

## 2. 学習用フォルダの確認結果

実在を確認した（Codexで未確認だった点は解消）。

```
analysis/
└── MyBookLibrary/
    ├── AGENTS.md / CLAUDE.md / README.md
    ├── app/.gitkeep
    └── docs/ handoff-to-claude.md, learning-log.md, original-project-survey.md（本書）
```

## 3. Gitの状態と管理単位の提案

| 場所 | 状態 |
| --- | --- |
| `analysis/` 以上の親 | Git管理されていない |
| 学習用 `MyBookLibrary/` | Git管理されていない |
| 元プロジェクト | Git管理済み。ブランチ `fix/#634-reading-status-button-ui`、未コミット変更あり、origin/mainより33コミット遅れ |

**提案**：学習用 `MyBookLibrary/` 直下で `git init` する。
理由：`analysis/` には今後ほかの学習対象が増える可能性があり、プロジェクト単位の方が履歴と公開範囲を分けやすい。
`git init` は未実行（ユーザーの判断待ち）。初期化前に `.gitignore`（`node_modules/`、`.env*`、`!.env*.example`、`.next/`）を自分で用意すること。

## 4. 元プロジェクトの機能・技術構成・起動方法

### 主な機能（README より）

読書ステータス（未読/読みたい/読書中/読了）、文学賞作品一覧と読了率、楽天ブックスAPI検索（バーコード対応）、お気に入り著者、レビュー・いいね・通報、フォロー、図書館在庫確認（カーリルAPI）、通知、認証（メール＋パスワード/Google OAuth、ロック、合言葉リセット）、PWA、お問い合わせ、設定、管理者機能。

### 技術構成

| 役割 | 技術 |
| --- | --- |
| フロント/バックエンド | Next.js 16.2（App Router + API Routes）、React 19.2、TypeScript 5 |
| DB / ORM | MySQL 8.4（Docker）/ Prisma 6.19 |
| 認証 | NextAuth.js v5 beta |
| バリデーション / UI | Zod v4 / Tailwind CSS v4 + shadcn/ui |
| テスト | Jest 30（node・jsdom・integration）、Playwright（E2E） |
| 要件 | Node.js 22以上 |

Prismaモデルは21個（User, Book, Author, Award, AwardEntry, ReadingStatus, Review, Like, Follow, Notification など）、マイグレーション25件。

### 起動方法（README の手順）

1. ルートの `.env.example` を `.env` にコピーして値を設定 → `docker compose up -d`（MySQL、開発用3306・テスト用3307）
2. `cd app && npm install`
3. `.env.local.example` を `.env.local` にコピーして値を設定
4. `npx prisma migrate dev` → `npm run dev` → http://localhost:3000

### 調べる資料

`docs/requirements.md`（要件の正）、`docs/features/`、`docs/api-spec.md`、`docs/er-diagram.md`、`docs/screen-transition.md`。

## 5. 実施した検証

| 項目 | 結果 |
| --- | --- |
| 環境 | Node v24.19.0 / npm 11.17.0 / Docker 29.8.1。MySQLコンテナ2つは起動中 |
| 型チェック `tsc --noEmit --incremental false` | エラー0 |
| Lint `eslint` | エラー0、警告1（`scripts/build-konomys-csv.mjs` の未使用変数） |
| 単体テスト `jest --selectProjects node jsdom` | 35スイート・410件すべて成功 |

注意：結果は元プロジェクトの**作業中ブランチ（未コミット変更を含む）**のもの。`main` の状態ではない。
`tsbuildinfo` を更新しないよう `--incremental false` で実行した。

### 未実施（理由）

- 統合テスト・E2E：DBへの書き込みを伴うため、範囲の確認が必要。
- `npm run build` / `npm run dev`：`.next/` の生成など元プロジェクトのファイルを変更するため。
- ブラウザでの動作確認、外部API（楽天・カーリル・NDL）の呼び出し。

## 6. 最初の小さな課題（提案）

**課題1：Next.js プロジェクトを作り、本の一覧を表示する（DBなし）**

- 内容：`app/` に Next.js（TypeScript, App Router）を作成し、トップページにコード内の固定配列（3冊程度）からタイトル・著者・読書ステータスを一覧表示する。
- なぜ最初か：元アプリの中心である「本」と「読書ステータス」を、DB・認証・外部APIなしで扱える。
- 完了条件：
  - `npm run dev` で http://localhost:3000 に3冊が表示される。
  - 本の型（`type Book`）を自分で定義し、ステータスは4種類のどれかしか入らない。
  - `npm run lint` と `tsc --noEmit` がエラー0。
  - 「ページ表示時にどのファイルが実行されるか」を自分の言葉で説明できる。
- 考えるポイント：ステータスを string にするか union 型にするか。固定データをどこのファイルに置くか。
- 答え合わせ：自分で作った後に、元の `prisma/schema.prisma` の `Book` / `ReadingStatus` と比較する。
- テストデータは架空の書名・著者名を使う。

その後の候補：Docker MySQL + Prisma で一覧をDBから取得 → ステータス変更 → 認証。

## 7. 決定事項（2026-10-12）

### Git

- 学習用 `MyBookLibrary/` 直下で `git init` 済み（ブランチ `main`、コミットはまだない）。
- GitHub には元の `MyBookLibrary` がすでにあるため、公開する場合は別名にする（例：`MyBookLibrary-relearn`）。
- `.gitignore` 作成済み。`.env` と `.env.*` は除外し、`*.example` だけをコミット対象にしている。

### 技術選定：元のアプリからの変更点

| 役割 | 元のアプリ | 学習版 | 変更理由 |
| --- | --- | --- | --- |
| DB | MySQL 8.4 | **PostgreSQL 18**（`postgres:18`） | 実務でよく使われる。元のアプリの丸写しを防ぐ |
| ORM | Prisma 6 | **drizzle-orm 0.45.x** | SQLに近い書き方でSQLの知識が身につく。スキーマもTypeScriptで書く |
| マイグレーション | Prisma Migrate | **drizzle-kit 0.31.x** | Drizzleと組で使う |
| DBドライバ | （Prisma内蔵） | **postgres（postgres.js）3.4.x** | Drizzle公式ドキュメントの例に多い |
| DBの起動 | Docker Compose | **Docker Compose**（同じ） | — |

その他（Next.js / TypeScript / NextAuth など）は元のアプリに合わせる予定。認証を作る段階で `@auth/drizzle-adapter` を検討する。
Drizzleは1.0前のため、`package.json` ではバージョンを固定する（`^` を付けない、または lockfile を必ずコミットする）。

答え合わせの範囲：元の `schema.prisma` とは「どんなテーブル・列が必要か」を比較できる。DB操作コードは書き方が違うため比較できない。

MySQLとの違いで学習ポイントになるもの：自動採番（`serial` / `identity`）、日時型（`timestamp with time zone`）、enum、文字列比較の大文字小文字。

### Docker Compose の方針

- リポジトリ直下に `docker-compose.yml` を**自分で作成する**（課題2で扱う）。
- 接続情報は `docker-compose.yml` に直接書かず、`${POSTGRES_USER}` のような変数展開にする。Composeは同じフォルダの `.env` を自動で読む。
- 実値は `.env`（作成済み・コミットしない）、キーの一覧は `.env.example`（コミットする）にある。
  `.env` は推測されにくいランダム値で作成済み。値は自分でファイルを開いて確認する。
- 書くときに考えるポイント：
  - `services` に何という名前のサービスを置くか。イメージは `postgres:18`。
  - `postgres` イメージが初期設定に使う環境変数（`POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB`）。
  - ポートの対応（ホスト側 `${POSTGRES_PORT}` → コンテナ側 5432）。
  - データをコンテナ削除後も残すための名前付きボリューム。
  - 起動確認用の `healthcheck`（`pg_isready` コマンド）。
- 完了の確認：`docker compose up -d` → `docker compose ps` で healthy になる → `docker compose exec <サービス名> psql -U <ユーザー> -d mybooklibrary` で接続できる。
- 答え合わせ：元のアプリの `docker-compose.yml`（MySQL版）と見比べる。

## ユーザーに判断してほしいこと

1. 学習用でもIssue駆動を採用するか。
2. 統合テスト・E2Eを元プロジェクトで動かして見ておきたいか（テスト用DBへの書き込みあり）。
