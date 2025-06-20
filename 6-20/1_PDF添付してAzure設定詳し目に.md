### GPT回答
re環境でのリソース作成からEntra ID（旧Azure AD）のアプリ登録、VM構築までの手順をREADMEに追記しました。特に以下をカバーしています。

* **リソースグループ作成**：`az group create`
* **Azure ADアプリ登録**：アプリの新規登録、リダイレクトURI設定、クライアントシークレット発行、API許可
* **環境変数**：必要な .env のキーと説明
* **VMデプロイ**：VM作成、NSG設定、Docker／Composeインストール、デプロイ手順

ご確認のうえ、不足やご要望があればお知らせください！

<br><br><br>

### スキャホールド（アプリケーションの足場・骨組み）
```bash
# プロジェクト構成
overall-attendance-report-system/
├── docker-compose.yml
├── .env.example
├── README.md
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   │   ├── index.js
│   │   ├── config.js
│   │   ├── db.js
│   │   ├── routes/
│   │   │   ├── auth.js
│   │   │   ├── attendance.js
│   │   │   └── report.js
│   │   └── migrations/
│   │       └── 001_init.sql
└── frontend/
    ├── Dockerfile
    ├── nginx.conf
    ├── package.json
    └── src/
        ├── index.js
        ├── App.jsx
        ├── components/
        │   ├── ClockButton.jsx
        │   ├── Dashboard.jsx
        │   └── ReportEditor.jsx
        └── i18n.js
```

---

## docker-compose.yml

```yaml
version: '3.8'
services:
  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - db_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  backend:
    build: ./backend
    restart: always
    env_file: .env
    depends_on:
      - db
    ports:
      - "4000:4000"

  frontend:
    build: ./frontend
    restart: always
    ports:
      - "80:80"

volumes:
  db_data: {}
```

---

## .env.example

```env
# Database
DB_HOST=db
DB_PORT=5432
DB_USER=appuser
DB_PASSWORD=securepass
DB_NAME=attendance

# Azure AD / Entra ID
AZURE_TENANT_ID=your-tenant-id
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET=your-client-secret

# Public URL (例: https://example-app.azurewebsites.net)
APP_BASE_URL=http://localhost
```

---

## README.md

````markdown
# 勤怠・日報管理システム

## 概要
- 新卒研修プロジェクトとして、勤怠打刻・日報投稿を行うWebアプリケーション。
- Docker ComposeでAPI、DB、フロントエンドをまとめて起動可能。
- Microsoft Entra ID (旧Azure AD)による認証・認可。

## ローカルセットアップ
1. リポジトリをクローン
2. `.env.example` をコピーし `.env` を作成。環境変数を設定
3. `docker-compose up --build` を実行
4. ブラウザで `http://localhost` を開く

## Azure 環境設定
### 1. リソースグループ作成
```bash
az group create \
  --name attendance-rg \
  --location japaneast
````

### 2. Azure AD アプリ登録 (Entra ID)

1. Azure Portal で「Azure Active Directory」→「アプリ登録」→「新規登録」をクリック
2. 名前：`attendance-app`
3. サポートされるアカウントの種類：任意の組織ディレクトリ(任意のAzure AD)
4. リダイレクトURI： `http://localhost/auth/azure/callback`（ローカル検証時）
5. 登録後、「認証」から同リダイレクトURIを追加
6. 「証明書とシークレット」で「新しいクライアントシークレット」を作成し、値をコピー
7. 「API のアクセス許可」で以下を追加：

   * Microsoft Graph → OpenID、プロファイル、email → Delegated Permissions
8. 必要に応じて管理者同意を付与

### 3. 環境変数設定

| 変数名                   | 内容                                           |
| --------------------- | -------------------------------------------- |
| AZURE\_TENANT\_ID     | Azure AD テナント ID                             |
| AZURE\_CLIENT\_ID     | アプリ登録で発行されたアプリケーション (クライアント) ID              |
| AZURE\_CLIENT\_SECRET | 上記で作成したクライアント シークレットの値                       |
| APP\_BASE\_URL        | アプリの公開URL（例：`https://<your-domain>` またはローカル） |

### 4. VM デプロイ手順

1. VM 作成（例: Ubuntu LTS）

   ```bash
   az vm create \
     --resource-group attendance-rg \
     --name attendance-vm \
     --image UbuntuLTS \
     --admin-username azureuser \
     --generate-ssh-keys
   ```
2. ネットワーク セキュリティ グループ (NSG) でポート 80, 443, 4000 を開放
3. SSH で VM に接続
4. Docker と Docker Compose のインストール

   ```bash
   sudo apt update && \
   sudo apt install -y docker.io docker-compose && \
   sudo usermod -aG docker $USER
   ```
5. リポジトリをクローンし、`.env` を配置
6. `docker-compose up -d --build` を実行
7. ブラウザで `http://<VMのパブリックIP>` またはドメイン名を開く

## データベースマイグレーション

* 初回起動時、`backend/src/migrations/001_init.sql` が自動適用されます。

## アーキテクチャ図

* ER図、シーケンス図は `docs/` に配置予定

```
```
