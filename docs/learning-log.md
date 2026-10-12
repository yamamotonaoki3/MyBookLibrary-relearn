# 学習記録

## 現在地

- 学習用フォルダの準備と元プロジェクトの調査が完了（2026-10-12）。
- 調査結果は `docs/original-project-survey.md` を参照。
- 学習用 `MyBookLibrary/` で `git init` 済み。`.gitignore`・`.env`・`.env.example` を作成済み。
- 技術選定：PostgreSQL 18 + Drizzle（元のアプリの MySQL + Prisma から変更）。DBは Docker Compose で起動する。
  詳細は `original-project-survey.md` の7章。
- 課題0-1（PostgreSQL を Docker Compose で起動）が完了。アプリの実装は未着手。

## 次の課題（2026-10-12 に順番を変更）

開発環境ごと Docker 上に作る方針にしたため、環境構築を先に行う。詳細は `docker-dev-environment.md`。

- ~~課題0-1「Docker Compose で PostgreSQL を起動して接続する」~~ 完了（2026-10-12）。
- 課題0-2「`apps/web/Dockerfile` を書き、コンテナ内で Next.js プロジェクトを作って初期画面を表示する」。
- 課題1「Next.jsで本の一覧を表示する（DBなし）」。完了条件は `original-project-survey.md` の6章。
- 課題2「Drizzle で `books` テーブルを作り、一覧を DB から読む」。

## 記録

### 2026-10-12 課題0-1：Docker Compose で PostgreSQL を起動して接続する

- 期待する動作と完了条件: `docker compose up -d` で `db` が healthy になり、psql と adminer から接続できる。
- できるようになったこと:
  - `db`（`postgres:18.6`）と `adminer` を compose に書き、接続情報を `${...}` と `.env` で渡せた。
  - PostgreSQL 18 のデータ置き場が `/var/lib/postgresql`（17 以前の `/data` 付きと違う）だと調べて書けた。
  - `healthcheck`（`pg_isready`）、名前付きボリューム、`127.0.0.1` へのポート限定を設定できた。
- つまずいた原因:
  - YAML の字下げと、`volumes` / `ports` がリストである点（`- ` が必要）。エラーは「何行目の何文字目か」と、その直前の行を見て読む。
  - `restart: unless-stop` の綴り誤り。`docker compose config` は値の正しさまでは検査しない。
  - ファイル名の誤り（`docker-coompose.yml`）と、空白入りの名前の `.env .test`（`.gitignore` に合わず実値が漏れる恐れがあった）。
  - 元の MyBookLibrary とフォルダ名が同じで、Compose のプロジェクト名が衝突した。トップレベルの `name: mybooklibrary-relearn` で分離した。
- 動作確認の結果: `docker compose ps` で `db` が healthy。`psql` で接続して `\l` を確認。adminer（Server は `db`）でログインし、`public` スキーマにテーブルがないことを確認。
- まだ分からないこと（要自己確認）:
  - adminer の Server が `localhost` でなく `db` になる理由。
  - `down` → `up -d` でデータが残る場所と、`down -v` との違い。
- 次に取り組むこと: 課題0-2（`apps/web/Dockerfile` と `web` サービス）。

## 記録用テンプレート

### 日付・取り組んだ課題

- 期待する動作と完了条件:
- できるようになったこと:
- つまずいた原因:
- 動作確認の結果:
- まだ分からないこと:
- 次に取り組むこと:
