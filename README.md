# 杉野パーソナルジム 予約・顧客管理システム (ReGYM)

※ 本プロジェクトは営業・実務を想定して構築された架空クライアント向けデモアプリケーションです。

Next.js (App Router) と Supabase を活用した、パーソナルジム向けのフルスタック予約・店舗管理システムです。  
一般会員向けのオンライン予約システムに加え、オーナー（管理者）向けのシフト管理・枠ブロック・会員招待機能を備えています。

- **本番デモ URL:** [https://YOUR-APP-NAME.vercel.app](https://YOUR-APP-NAME.vercel.app)
- **テスト用アカウント:**
  - **オーナー:** `admin@admin.admin` / `(設定したパスワード)`
  - **一般会員:** `member@member.mem` / `(設定したパスワード)`

---

## 🛠 技術構成 (Tech Stack)

| カテゴリ | 選定技術 |
|---|---|
| **フロントエンド** | Next.js 16 (App Router, Server Actions), React 19, TypeScript, Tailwind CSS |
| **バックエンド / DB** | Supabase (Auth, PostgreSQL, Row Level Security) |
| **デプロイ / CI/CD** | Vercel, GitHub (PRベースでの機能開発) |
| **セキュリティ** | `server-only`, PostgreSQL Security Definer Functions |

---

## ✨ 主な機能

### 📱 会員向け機能
- **リアルタイム空き枠算出:** トレーナーのシフトと既存予約から 75 分刻みの空き枠をオンデマンド計算。
- **直感的な予約フロー:** 14 日間のカレンダーから日付・時間帯を選択して即時予約。
- **予約確認・キャンセル:** 24 時間前までの論理キャンセル（キャンセルポリシーの適用）。

### 🔐 オーナー（管理者）向け機能
- **予約ダッシュボード:** 全会員の予約一覧確認および代理キャンセル（24時間ルール免除）。
- **シフト管理:** 曜日ごとのトレーナー稼働スケジュールの動的設定。
- **枠ブロック設定:** 急病・店舗メンテナンス等に対応する特定時間の予約受付停止（`status = 'blocked'`）。
- **会員招待フロー:** `service_role` を活用した安全な新規会員招待とプロフィール連動。

---

## 💡 開発における工夫・技術的解決（Highlight）

### 1. 物理的な二重予約防止 (Database-Level Protection)
単なるアプリケーション層でのチェックに頼らず、PostgreSQL の**部分ユニークインデックス (Partial Unique Index)** を採用。
`status IN ('confirmed', 'blocked')` のレコードに対して `(trainer_id, start_time)` の重複を DB レベルで遮断し、ミリ秒単位の競合（レースコンディション）が発生しても確実に整合性を保ちます。

### 2. RLS 無限再帰（Infinite Recursion）の解消と安全な特権判定
`profiles` テーブルのアクセス制御において、「オーナー判定のために `profiles` を参照する」ことによる無限ループを検知・解消。
`security definer` を付与した DB ヘルパー関数 `is_owner()` を定義し、RLS を安全に迂回しながら判定する構造に刷新しました。

### 3. `service_role` キーの完全な物理隔離
会員招待機能に必要な管理者権限（`service_role`）を扱うため、専用のモジュール (`lib/supabase/admin.ts`) に `import "server-only"` を配置。
クライアントサイド（ブラウザ）の JavaScript バンドルへの特権キー漏洩をビルドタイムで物理的に遮断しています。

---

## 🗄 データベース設計 & マイグレーション

リポジトリ内の `/supabase/migrations` にて、初期作成からポリシー修正までの全変更履歴を連番 SQL ファイルとして完全管理しています。

- `0001_initial_schema.sql`: テーブル定義・インデックス作成
- `0002_rls_policies.sql`: 基本アクセス制御
- `0003_get_booked_slots.sql`: 空き枠算出ストアドプロシージャ
- `0004_fix_owner_rls_recursion.sql`: `is_owner()` による RLS 無限再帰解消
- `0005_owner_management_policies.sql`: 管理者機能用 RLS パッチ

---

## 💻 開発プロセスの可視化

本プロジェクトでは、実務のチーム開発に沿った厳格な Git フローを採用しています。
各フェーズ（Phase 0〜5）ごとに Issue/PR を作成し、丁寧なコミットメッセージとコードレビューログを記録しています。詳細は [Pull Requests](../../pulls?q=is%3Apr+is%3Aclosed) をご覧ください。
