# SubsTracker 開発ロードマップ

Wallosインスパイア・自宅ホスト型のサブスクリプション管理アプリを、Next.js + PostgreSQL + Docker で作る。最終的にCloudflare Tunnel経由で自宅サーバーから公開する。

---

## 技術スタック

| レイヤー | 技術 | 理由 |
|---|---|---|
| フロント+API | Next.js 15 (App Router) | フルスタック1本で完結、学習効果高い |
| 言語 | TypeScript | 型安全 |
| UI | Tailwind CSS + shadcn/ui | 素早く整ったUI |
| ORM | Prisma | スキーマファースト、型生成、マイグレーション込み |
| DB | PostgreSQL 16 | 本業と同じ |
| 認証 | Auth.js (NextAuth v5) | OIDC学習にも繋がる |
| グラフ | Recharts | Reactで定番 |
| メール | Resend | 無料枠3000通/月 |
| コンテナ | Docker / Docker Compose | 開発も本番も同じ環境 |
| ホスト | Raspberry Pi 5 or ミニPC | 自宅サーバー |
| 公開 | Cloudflare Tunnel | ポート開放不要、HTTPS自動 |

---

## Phase 0: 準備

### 0-1. ツールインストール
- [x] Node.js 20+ をインストール (`node -v` で確認)
- [x] Docker Desktop をインストール (`docker --version` で確認)
- [x] Git をインストール (`git --version` で確認)
- [x] VS Code or 好きなエディタを準備

### 0-2. プロジェクト作成
- [x] GitHubで空の `substracker` リポジトリを作成
- [x] `npx create-next-app@latest substracker` で雛形作成
  - TypeScript: Yes
  - ESLint: Yes
  - Tailwind CSS: Yes
  - `src/` directory: Yes
  - App Router: Yes
- [x] `git remote add origin` でGitHubに接続
- [x] 初回 push 完了

### 0-3. 動作確認
- [x] `npm run dev` で http://localhost:3000 が表示される

### 0-4. UI部品セットアップ
- [x] `npx shadcn@latest init` 実行
- [x] 基本コンポーネント追加 `npx shadcn@latest add button card input label select dialog form`

### 0-5. プロジェクト構成準備
- [x] `.env.example` を作成 (GitHubに上げる用テンプレート)
- [x] `.gitignore` に `.env` `data/` `node_modules/` が含まれているか確認

---

## Phase 1: MVP (最小限の動くもの)

**完成イメージ**: ログインして、サブスクを登録・編集・削除できる

### 1-1. Docker環境構築
- [x] プロジェクトルートに `docker-compose.yml` 作成 (DB用)
- [x] `docker compose up -d` で PostgreSQL 起動
- [x] `docker compose ps` で稼働確認
- [x] DBクライアント (TablePlus, DBeaver等) で接続確認

### 1-2. Prismaセットアップ
- [ ] `npm install prisma @prisma/client`
- [ ] `npx prisma init`
- [ ] `.env` に `DATABASE_URL` 設定
- [ ] `prisma/schema.prisma` に初期スキーマを書く
  - [ ] User モデル
  - [ ] Category モデル
  - [ ] Subscription モデル
- [ ] `npx prisma migrate dev --name init` で初回マイグレーション
- [ ] `npx prisma studio` で動作確認
- [ ] `src/lib/prisma.ts` でPrismaClientシングルトン作成

### 1-3. 認証システム (Auth.js)
- [ ] `npm install next-auth@beta bcryptjs`
- [ ] `npx auth secret` で AUTH_SECRET 生成
- [ ] `src/auth.ts` 作成 (Credentials Provider設定)
- [ ] `src/middleware.ts` で保護ルート設定
- [ ] ユーザー登録API (`/api/auth/register`)
- [ ] パスワードハッシュ化処理 (bcrypt)
- [ ] ログイン画面 (`/login`)
- [ ] 登録画面 (`/register`)
- [ ] ログアウト機能
- [ ] セッション動作確認

### 1-4. サブスクCRUD API
- [ ] `GET /api/subscriptions` (一覧取得)
- [ ] `POST /api/subscriptions` (新規作成)
- [ ] `GET /api/subscriptions/[id]` (単一取得)
- [ ] `PATCH /api/subscriptions/[id]` (更新)
- [ ] `DELETE /api/subscriptions/[id]` (削除)
- [ ] 認証チェック (各エンドポイントで session 確認)
- [ ] 所有者チェック (他人のサブスクを操作不可)
- [ ] バリデーション (Zod で入力検証)

### 1-5. 画面実装
- [ ] 共通レイアウト (ヘッダー、ナビゲーション)
- [ ] サブスク一覧画面 (`/`)
  - [ ] 一覧表示
  - [ ] ソート (支払日順、金額順、名前順)
  - [ ] 検索フィルタ
- [ ] 新規登録フォーム (`/subscriptions/new`)
  - [ ] 金額、通貨、周期入力
  - [ ] バリデーション表示
- [ ] 編集フォーム (`/subscriptions/[id]/edit`)
- [ ] 削除確認ダイアログ

### 1-6. カテゴリ管理
- [ ] Category CRUD API
- [ ] カテゴリ管理画面 (`/settings/categories`)
- [ ] サブスクフォームでカテゴリ選択

### 1-7. Phase 1 完成チェック
- [ ] アカウント作成 → ログインできる
- [ ] サブスクを5件くらい登録できる
- [ ] 一覧で並び替え・検索できる
- [ ] 編集・削除できる
- [ ] 別アカウントで他人のデータが見えないこと

---

## Phase 2: 実用機能

**完成イメージ**: 日常使いできるレベルになる

### 2-1. ダッシュボード
- [ ] メトリックカード4種
  - [ ] 月額合計
  - [ ] 年額合計
  - [ ] 登録件数
  - [ ] 今月の支払い合計
- [ ] 直近の支払いリスト (7日以内)
- [ ] カテゴリ別の支出割合 (バー表示)

### 2-2. 計算ロジック
- [ ] `src/lib/calculations.ts` 作成
  - [ ] サブスクごとの月額換算 (年額・週額からの変換)
  - [ ] 合計計算
  - [ ] 次回支払日までの日数計算
- [ ] ユニットテスト (任意だが推奨)

### 2-3. カレンダー画面
- [ ] `/calendar` ルート作成
- [ ] 月表示カレンダー実装 (`react-big-calendar` or 自前)
- [ ] 支払日をマーキング
- [ ] 日付クリックで詳細表示
- [ ] 前月・翌月ナビゲーション

### 2-4. 次回支払日の自動更新
- [ ] 「支払日が今日のサブスクを次の周期に進める」処理
- [ ] `/api/cron/update-payments` 実装
- [ ] CRON_SECRET で認証
- [ ] 動作確認 (手動で叩いてみる)

### 2-5. 通知システム
- [ ] Resendアカウント作成、APIキー取得
- [ ] `npm install resend`
- [ ] メール送信ユーティリティ `src/lib/email.ts`
- [ ] 通知設定をユーザー単位で管理 (DB追加)
  - [ ] 何日前に通知するか
  - [ ] 通知ON/OFF
- [ ] 通知バッチAPI `/api/cron/send-notifications`
- [ ] メールテンプレート (HTML)
- [ ] 動作確認

### 2-6. Phase 2 完成チェック
- [ ] ダッシュボードで月額・年額が正しく表示
- [ ] カレンダーで支払予定が見える
- [ ] 支払日になると自動で次回支払日が更新される
- [ ] 通知メールが届く

---

## Phase 3: 拡張機能 (お好みで)

優先度は自由。気になるものから取り組む。

### 3-1. 統計画面
- [ ] `/stats` ルート作成
- [ ] カテゴリ別円グラフ (Recharts)
- [ ] 月別支出推移グラフ
- [ ] 年間予測表示

### 3-2. 多通貨対応
- [ ] 為替レート取得API (exchangerate-api.com など)
- [ ] `exchange_rates` テーブル追加
- [ ] レート更新バッチ (日次)
- [ ] メイン通貨設定 (ユーザー単位)
- [ ] 表示時にメイン通貨に換算

### 3-3. ロゴ自動取得
- [ ] Clearbit Logo API か Google Favicon API で取得
- [ ] サブスク名入力時に自動取得
- [ ] 手動アップロード機能 (任意)

### 3-4. 通知チャネル拡張
- [ ] Discord Webhook
- [ ] Slack Webhook
- [ ] (任意) Pushover, Telegram, Gotify

### 3-5. データインポート/エクスポート
- [ ] CSV エクスポート
- [ ] CSV インポート
- [ ] JSON バックアップ

### 3-6. OIDC ログイン
- [ ] GoogleプロバイダーをAuth.jsに追加
- [ ] (任意) GitHub プロバイダー
- [ ] 既存ユーザーとのリンク機能

### 3-7. PWA化
- [ ] `next-pwa` インストール
- [ ] `manifest.json` 作成
- [ ] アイコン用意 (PWA Asset Generator)
- [ ] Service Worker動作確認
- [ ] ホーム画面追加できることを確認 (スマホ)

### 3-8. ダークモード
- [ ] `next-themes` でテーマ切り替え
- [ ] Tailwind の dark: バリアント活用

### 3-9. 多言語対応
- [ ] `next-intl` でi18n
- [ ] 日本語・英語の辞書ファイル

---

## Phase 4: 本番デプロイ (自宅サーバー)

### 4-1. ハードウェア準備
- [ ] Raspberry Pi 5 (or ミニPC) を入手
- [ ] microSD or SSD に Raspberry Pi OS Lite 64bit を書き込み
- [ ] 初回起動、SSH接続確認
- [ ] 固定IP化 (ルーターのDHCP予約)
- [ ] タイムゾーン設定 `sudo timedatectl set-timezone Asia/Tokyo`
- [ ] システム更新 `sudo apt update && sudo apt upgrade -y`
- [ ] ufw でファイアウォール (SSHのみ許可)

### 4-2. Docker導入
- [ ] `curl -fsSL https://get.docker.com | sh`
- [ ] `sudo usermod -aG docker $USER`
- [ ] 動作確認 `docker run hello-world`

### 4-3. 本番用Dockerfileを書く
- [ ] `Dockerfile` (マルチステージビルド)
  - [ ] deps ステージで依存解決
  - [ ] builder ステージでビルド
  - [ ] runner ステージで最小化
- [ ] `next.config.js` に `output: 'standalone'`
- [ ] `.dockerignore` で `node_modules/` `.next/` `.env*` 除外
- [ ] ローカルでビルド確認 `docker build -t substracker .`

### 4-4. 本番用 docker-compose.yml
- [ ] `docker-compose.prod.yml` 作成
- [ ] app と db の本番設定
- [ ] ボリュームでDBデータ永続化
- [ ] 環境変数を .env から読み込み
- [ ] ヘルスチェック設定

### 4-5. Cloudflare Tunnel設定
- [ ] Cloudflareアカウント作成
- [ ] ドメイン取得 or 既存ドメインを Cloudflare に移管
- [ ] Zero Trust ダッシュボードでトンネル作成
- [ ] cloudflared を Pi にインストール
- [ ] systemd サービスとして起動
- [ ] Public Hostname 設定 (`substracker.your-domain.com` → `localhost:3000`)

### 4-6. デプロイ
- [ ] サーバーにプロジェクトを git clone
- [ ] `.env` を作成 (本番用シークレット)
- [ ] `docker compose -f docker-compose.prod.yml up -d`
- [ ] Prisma マイグレーション実行 `docker compose exec app npx prisma migrate deploy`
- [ ] ブラウザから https://substracker.your-domain.com にアクセス確認
- [ ] ユーザー登録 → ログインまで通す

### 4-7. Cron設定
- [ ] crontab に通知バッチを登録
  - 毎日9時: 通知送信
  - 毎日0時: 次回支払日更新
- [ ] CRON_SECRET 環境変数を設定
- [ ] curl で動作確認

### 4-8. バックアップ整備
- [ ] `backup.sh` 作成 (pg_dump → gzip)
- [ ] 日次 cron で自動実行
- [ ] 7日以上前を削除する処理
- [ ] (任意) Cloudflare R2 にオフサイトバックアップ
- [ ] リストア手順を README に記載

### 4-9. 運用整備
- [ ] unattended-upgrades でセキュリティパッチ自動適用
- [ ] ログローテーション設定
- [ ] (任意) Uptime Kuma で死活監視
- [ ] (任意) Healthchecks.io で cron 監視

### 4-10. ドキュメント整備
- [ ] README に概要・スクリーンショット
- [ ] 開発環境セットアップ手順
- [ ] 本番デプロイ手順
- [ ] バックアップ・リストア手順
- [ ] (公開するなら) ライセンス表記

---

## マイルストーン

| 達成ライン | 目安期間 | 状態 |
|---|---|---|
| Phase 0 完了 | 半日〜1日 | 雛形が動く |
| Phase 1 完了 | 1〜2週間 | 自分でローカル使える |
| Phase 2 完了 | +1〜2週間 | 日常使いできる |
| Phase 3 部分着手 | 適宜 | 個別機能追加 |
| Phase 4 完了 | +数日 | 自宅サーバーで公開 |

無理に全部やる必要はなく、**Phase 2 まで完成すれば実用レベル**。Phase 3 は欲しい機能だけつまみ食いでOK。

---

## 学習ポイント (Spring Boot 経験者として注目すべきこと)

### Next.js特有
- [ ] Server Components と Client Components の境界 (`"use client"`)
- [ ] Server Actions (フォーム送信を直接サーバー関数で処理)
- [ ] App Router のファイルベースルーティング
- [ ] `route.ts` での REST API 実装パターン
- [ ] `middleware.ts` での横断的処理

### Prisma vs MyBatis の違い
- [ ] スキーマファースト (XMLマッピングではなく `.prisma` ファイル)
- [ ] 自動型生成 (TypeScript の型が DB スキーマから自動)
- [ ] マイグレーション管理 (Flyway 的な役割込み)
- [ ] N+1 対策の `include` / `select`
- [ ] 生SQL書きたい時の `$queryRaw`

### Docker関連
- [ ] image と container の違い
- [ ] volume でデータ永続化
- [ ] network でコンテナ間通信
- [ ] マルチステージビルドで本番イメージ最小化
- [ ] `docker compose` vs `docker compose -f xxx.yml`

### インフラ
- [ ] Linux 基本操作 (systemd, cron, ufw)
- [ ] Cloudflare Tunnel の仕組み
- [ ] Let's Encrypt と Cloudflare の証明書の違い
- [ ] バックアップ戦略

---

## 困った時の確認先

- Next.js: https://nextjs.org/docs
- Prisma: https://www.prisma.io/docs
- Auth.js: https://authjs.dev
- shadcn/ui: https://ui.shadcn.com
- Docker: https://docs.docker.com
- Cloudflare Tunnel: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/

---

最終更新: 2026-05-14
