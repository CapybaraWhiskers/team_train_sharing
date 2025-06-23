勤怠打刻と日報投稿を行う簡易的なWebシステム

.env でAPIホストやポートを設定し、Flaskのバックエンドでは以下のエンドポイントを提供しています。
POST /attendance/clock-in と POST /attendance/clock-out: それぞれ打刻情報をSQLiteに保存し、タイムスタンプを返す
POST /report/: Markdown形式のレポートを保存
GET /reports: 保存済みレポート一覧をJSONで返す

フロントエンドの index.html には、打刻ボタンとMarkdown入力欄、送信したレポート一覧を表示する領域が用意されています。
また、docker-compose.yml でFlaskバックエンドとNginxフロントエンドをまとめて起動できるよう定義されています。

