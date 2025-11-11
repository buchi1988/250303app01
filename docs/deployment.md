# デプロイガイド

250303app01 を本番環境にデプロイする方法について説明します。

## 目次

- [デプロイ前の準備](#デプロイ前の準備)
- [プラットフォーム別デプロイ方法](#プラットフォーム別デプロイ方法)
  - [Heroku](#heroku)
  - [AWS](#aws)
  - [Google Cloud Platform](#google-cloud-platform)
  - [Azure](#azure)
  - [Docker](#docker)
  - [VPS/専用サーバー](#vps専用サーバー)
- [環境変数の設定](#環境変数の設定)
- [データベースのセットアップ](#データベースのセットアップ)
- [セキュリティ設定](#セキュリティ設定)
- [モニタリングとロギング](#モニタリングとロギング)
- [バックアップ戦略](#バックアップ戦略)
- [トラブルシューティング](#トラブルシューティング)

## デプロイ前の準備

### 1. チェックリスト

デプロイ前に以下を確認してください：

- [ ] すべてのテストが通る
- [ ] 環境変数が適切に設定されている
- [ ] データベースマイグレーションが最新
- [ ] セキュリティ設定が適切
- [ ] ログ設定が本番環境用
- [ ] エラーハンドリングが適切
- [ ] パフォーマンステストが完了
- [ ] バックアップ戦略が確立されている

### 2. ビルド

本番環境用のビルドを作成：

```bash
# プロダクションビルド
npm run build
# または
python setup.py build

# ビルドの検証
npm run test:production
```

### 3. 環境設定

`.env.production` ファイルを作成：

```env
NODE_ENV=production
APP_ENV=production
DEBUG=false
LOG_LEVEL=error

# データベース
DB_HOST=your-production-db-host
DB_PORT=5432
DB_NAME=production_db
DB_USER=production_user
DB_PASSWORD=strong_production_password

# セキュリティ
JWT_SECRET=your-very-strong-secret-key
API_KEY=your-production-api-key
ALLOWED_ORIGINS=https://yourdomain.com

# その他
REDIS_URL=redis://your-redis-host:6379
STORAGE_PATH=/var/app/storage
```

## プラットフォーム別デプロイ方法

### Heroku

#### 準備

```bash
# Heroku CLI のインストール
# https://devcenter.heroku.com/articles/heroku-cli

# ログイン
heroku login

# アプリケーションを作成
heroku create your-app-name
```

#### デプロイ

```bash
# Git リモートを追加（自動的に追加されない場合）
heroku git:remote -a your-app-name

# 環境変数を設定
heroku config:set NODE_ENV=production
heroku config:set API_KEY=your-api-key

# データベースを追加（PostgreSQL）
heroku addons:create heroku-postgresql:hobby-dev

# デプロイ
git push heroku main

# マイグレーション実行
heroku run npm run db:migrate
# または
heroku run python manage.py migrate

# ログを確認
heroku logs --tail
```

#### Procfile の作成

```
web: npm start
worker: npm run worker
```

### AWS

#### AWS Elastic Beanstalk

**1. EB CLI のインストール**

```bash
pip install awsebcli
```

**2. 初期化とデプロイ**

```bash
# Elastic Beanstalk の初期化
eb init -p node.js-18 your-app-name --region us-west-2

# 環境を作成
eb create production-env

# デプロイ
eb deploy

# 環境変数を設定
eb setenv NODE_ENV=production API_KEY=your-api-key

# ログを確認
eb logs
```

#### AWS EC2 手動デプロイ

**1. EC2 インスタンスのセットアップ**

```bash
# SSH でインスタンスに接続
ssh -i your-key.pem ec2-user@your-instance-ip

# 必要なパッケージをインストール
sudo yum update -y
sudo yum install -y nodejs npm git

# アプリケーションをクローン
git clone https://github.com/your-repo/250303app01.git
cd 250303app01

# 依存関係をインストール
npm install --production

# PM2 をインストール（プロセス管理）
sudo npm install -g pm2

# アプリケーションを起動
pm2 start npm --name "app01" -- start

# 自動起動を設定
pm2 startup
pm2 save
```

**2. Nginx のセットアップ**

```bash
# Nginx をインストール
sudo yum install -y nginx

# 設定ファイルを作成
sudo nano /etc/nginx/conf.d/app01.conf
```

Nginx 設定:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

```bash
# Nginx を起動
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Google Cloud Platform

#### Google App Engine

**1. gcloud CLI のインストールと設定**

```bash
# gcloud CLI のインストール
# https://cloud.google.com/sdk/docs/install

# 初期化
gcloud init

# プロジェクトを作成
gcloud projects create your-project-id
gcloud config set project your-project-id
```

**2. app.yaml の作成**

```yaml
runtime: nodejs18
env: standard

instance_class: F2

automatic_scaling:
  min_instances: 1
  max_instances: 10
  target_cpu_utilization: 0.65

env_variables:
  NODE_ENV: "production"
  API_KEY: "your-api-key"

handlers:
  - url: /.*
    script: auto
    secure: always
```

**3. デプロイ**

```bash
# デプロイ
gcloud app deploy

# ログを確認
gcloud app logs tail -s default
```

### Azure

#### Azure App Service

```bash
# Azure CLI のインストール
# https://docs.microsoft.com/en-us/cli/azure/install-azure-cli

# ログイン
az login

# リソースグループを作成
az group create --name myResourceGroup --location "East US"

# App Service プランを作成
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku B1 --is-linux

# Web App を作成
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name your-app-name --runtime "NODE|18-lts"

# デプロイ
az webapp deployment source config-local-git --name your-app-name --resource-group myResourceGroup

# Git リモートを追加してプッシュ
git remote add azure <deploymentLocalGitUrl>
git push azure main

# 環境変数を設定
az webapp config appsettings set --name your-app-name --resource-group myResourceGroup --settings NODE_ENV=production API_KEY=your-api-key
```

### Docker

#### Dockerfile の作成

```dockerfile
# ベースイメージ
FROM node:18-alpine

# 作業ディレクトリ
WORKDIR /app

# 依存関係ファイルをコピー
COPY package*.json ./

# 依存関係をインストール
RUN npm ci --only=production

# アプリケーションファイルをコピー
COPY . .

# ビルド（必要な場合）
RUN npm run build

# ポートを公開
EXPOSE 3000

# ヘルスチェック
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js

# アプリケーションを起動
CMD ["npm", "start"]
```

#### docker-compose.yml の作成

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
      - REDIS_HOST=redis
    depends_on:
      - db
      - redis
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=app_db
      - POSTGRES_USER=app_user
      - POSTGRES_PASSWORD=secure_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app
    restart: unless-stopped

volumes:
  postgres_data:
```

#### ビルドとデプロイ

```bash
# イメージをビルド
docker build -t 250303app01:latest .

# コンテナを起動
docker run -d -p 3000:3000 --name app01 250303app01:latest

# Docker Compose を使用
docker-compose up -d

# ログを確認
docker logs -f app01
# または
docker-compose logs -f
```

### VPS/専用サーバー

#### Ubuntu/Debian での手動セットアップ

```bash
# システムを更新
sudo apt update && sudo apt upgrade -y

# Node.js をインストール
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Git をインストール
sudo apt install -y git

# PM2 をインストール
sudo npm install -g pm2

# アプリケーションをクローン
cd /var/www
sudo git clone https://github.com/your-repo/250303app01.git
cd 250303app01

# 依存関係をインストール
npm install --production

# 環境変数を設定
sudo nano .env.production

# PM2 で起動
pm2 start npm --name "app01" -- start
pm2 startup
pm2 save

# Nginx をインストールして設定
sudo apt install -y nginx
sudo nano /etc/nginx/sites-available/app01
```

## 環境変数の設定

本番環境で必要な環境変数：

```env
# アプリケーション
NODE_ENV=production
APP_ENV=production
APP_PORT=3000
APP_URL=https://yourdomain.com

# データベース
DB_HOST=your-db-host
DB_PORT=5432
DB_NAME=production_db
DB_USER=db_user
DB_PASSWORD=secure_db_password
DB_SSL=true

# Redis/キャッシュ
REDIS_HOST=your-redis-host
REDIS_PORT=6379
REDIS_PASSWORD=redis_password

# セキュリティ
JWT_SECRET=very-strong-jwt-secret-key
API_KEY=production-api-key
SESSION_SECRET=session-secret-key
ALLOWED_ORIGINS=https://yourdomain.com,https://www.yourdomain.com

# 外部サービス
AWS_ACCESS_KEY_ID=your-aws-key
AWS_SECRET_ACCESS_KEY=your-aws-secret
S3_BUCKET=your-bucket-name

# ログ・モニタリング
LOG_LEVEL=error
SENTRY_DSN=your-sentry-dsn
```

## セキュリティ設定

### SSL/TLS証明書

Let's Encrypt を使用した無料SSL証明書：

```bash
# Certbot をインストール
sudo apt install -y certbot python3-certbot-nginx

# SSL証明書を取得
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# 自動更新を設定
sudo certbot renew --dry-run
```

### ファイアウォール

```bash
# UFW を有効化
sudo ufw enable

# 必要なポートを開放
sudo ufw allow 22/tcp   # SSH
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS

# 状態を確認
sudo ufw status
```

## モニタリングとロギング

### PM2 モニタリング

```bash
# PM2 モニター
pm2 monit

# ログを表示
pm2 logs

# ステータス確認
pm2 status
```

### システムモニタリング

推奨ツール：
- **Prometheus + Grafana**: メトリクス収集と可視化
- **Sentry**: エラートラッキング
- **New Relic / Datadog**: APM
- **CloudWatch / Stackdriver**: クラウドネイティブモニタリング

## バックアップ戦略

### データベースバックアップ

```bash
# PostgreSQL バックアップ
pg_dump -h localhost -U db_user -d production_db > backup_$(date +%Y%m%d_%H%M%S).sql

# 自動バックアップスクリプト
sudo nano /usr/local/bin/backup.sh
```

```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/var/backups/app01"
mkdir -p $BACKUP_DIR

# データベースバックアップ
pg_dump -h localhost -U db_user -d production_db | gzip > $BACKUP_DIR/db_$DATE.sql.gz

# ファイルバックアップ
tar -czf $BACKUP_DIR/files_$DATE.tar.gz /var/www/250303app01/uploads

# 古いバックアップを削除（30日以上）
find $BACKUP_DIR -type f -mtime +30 -delete
```

```bash
# 実行権限を付与
sudo chmod +x /usr/local/bin/backup.sh

# cron で毎日実行
sudo crontab -e
# 追加: 0 2 * * * /usr/local/bin/backup.sh
```

## トラブルシューティング

### アプリケーションが起動しない

```bash
# ログを確認
pm2 logs
journalctl -u your-service -f

# ポートを確認
sudo netstat -tulpn | grep :3000

# プロセスを確認
ps aux | grep node
```

### データベース接続エラー

```bash
# 接続をテスト
psql -h your-db-host -U db_user -d production_db

# 接続設定を確認
cat .env.production | grep DB_
```

### メモリ不足

```bash
# メモリ使用量を確認
free -h
pm2 monit

# スワップを追加（必要な場合）
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## デプロイの自動化（CI/CD）

### GitHub Actions の例

`.github/workflows/deploy.yml`:

```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '18'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Build
      run: npm run build

    - name: Deploy to server
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_KEY }}
        script: |
          cd /var/www/250303app01
          git pull origin main
          npm install --production
          npm run build
          pm2 restart app01
```

---

最終更新: 2025-11-11
