# 学習記録

## 現在地

- 学習用フォルダの準備と元プロジェクトの調査が完了（2026-10-12）。
- 調査結果は `docs/original-project-survey.md` を参照。
- 学習用 `MyBookLibrary/` で `git init` 済み。`.gitignore`・`.env`・`.env.example` を作成済み。
- 技術選定：PostgreSQL 18 + Drizzle（元のアプリの MySQL + Prisma から変更）。DBは Docker Compose で起動する。
  詳細は `original-project-survey.md` の7章。
- アプリの実装は未着手。

## 次の課題（2026-10-12 に順番を変更）

開発環境ごと Docker 上に作る方針にしたため、環境構築を先に行う。詳細は `docker-dev-environment.md`。

- 課題0-1「Docker Compose で PostgreSQL を起動して接続する」（旧課題2）。
- 課題0-2「`apps/web/Dockerfile` を書き、コンテナ内で Next.js プロジェクトを作って初期画面を表示する」。
- 課題1「Next.jsで本の一覧を表示する（DBなし）」。完了条件は `original-project-survey.md` の6章。
- 課題2「Drizzle で `books` テーブルを作り、一覧を DB から読む」。

## 記録用テンプレート

### 日付・取り組んだ課題

- 期待する動作と完了条件:
- できるようになったこと:
- つまずいた原因:
- 動作確認の結果:
- まだ分からないこと:
- 次に取り組むこと:
