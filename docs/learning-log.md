# 学習記録

## 現在地

- 学習用フォルダの準備と元プロジェクトの調査が完了（2026-10-12）。
- 調査結果は `docs/original-project-survey.md` を参照。
- 学習用 `MyBookLibrary/` で `git init` 済み。`.gitignore`・`.env`・`.env.example` を作成済み。
- 技術選定：PostgreSQL 18 + Drizzle（元のアプリの MySQL + Prisma から変更）。DBは Docker Compose で起動する。
  詳細は `original-project-survey.md` の7章。
- アプリの実装は未着手。

## 次の課題

- 課題1「Next.jsで本の一覧を表示する（DBなし）」。完了条件は `original-project-survey.md` の6章。
- 課題2「Docker Compose で PostgreSQL を起動して接続する」。考えるポイントと完了の確認は7章。
- 課題3「Drizzle で `books` テーブルを作り、一覧を DB から読む」。

## 記録用テンプレート

### 日付・取り組んだ課題

- 期待する動作と完了条件:
- できるようになったこと:
- つまずいた原因:
- 動作確認の結果:
- まだ分からないこと:
- 次に取り組むこと:
