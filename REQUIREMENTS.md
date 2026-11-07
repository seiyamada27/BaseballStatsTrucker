# 草野球成績管理アプリ 要件定義書

## 1. プロジェクト概要・目的
- **プロジェクト名**: baseball Stats Tracker (草野球成績管理システム)
- **目的**: 草野球チームの試合結果および個人打撃成績を管理し、各種セイバーメトリクス指標（打率、出塁率、長打率、OPS）を自動集計・可視化することで、チームの戦力分析を支援する。
- **背景**: 手作業によるExcel管理からの脱却を図り、ブラウザ上で直感的に成績登録・閲覧できるWEBシステムを構築する。

## 2. スコープ定義 (Scope Management)

### 2.1. インスコープ（今回開発する範囲 / MVP）
- 選手情報の基本CRUD（作成・取得・更新・削除）
- 試合情報および試合単位での打撃成績登録機能
- 通算成績および各打撃指標（打率、出塁率、長打率、OPS）の自動計算
- チーム内成績ランキングの閲覧機能
- チーム全体試合単位成績、通算成績
- 野球場マップでの打球方向表示、打球方向
割合表示

### 2.2. アウトスコープ（今回の対象外 / 将来のフェーズ）
- ユーザー認証・権限管理（ログイン・ログアウト機能）
- 投手成績（防御率、奪三振数等）の管理機能
- 1打席ごとの配球・球種トラッキング機能
- 外部SNS連携およびモバイルアプリ化

## 3. システム構成・動作環境 (System Architecture)

### 3.1. 開発・実行環境
- **Language**: Java 17 (LTS)
- **Framework**: Spring Boot 3.x
- **Template Engine**: Thymeleaf
- **Database**: MySQL 8.0
- **Build Tool**: Gradle (または Maven)
- **IDE / Editor**: Visual Studio Code
- **Version Control**: Git / GitHub (Git Flow準拠)

### 3.2. アーキテクチャ構成
本システムは三層アーキテクチャ（MVCパターン）を採用する。
- **Presentation Layer**: Thymeleaf + HTML5 / CSS3 / JavaScript
- **Business Layer**: Spring Boot Service Class（成績・指標計算ロジック）
- **Data Access Layer**: Spring Data JPA / MySQL

## 4. 機能要件 (Functional Requirements)

### 4.1. 選手管理機能 (Player Management)
- **F-01: 選手登録**: 選手名、背番号、守備位置、投打（右投右打など）を登録できる。
- **F-02: 選手一覧**: 登録済みの選手情報をテーブル形式で全件表示する。
- **F-03: 選手詳細・編集**: 指定した選手情報を更新できる。
- **F-04: 選手削除**: 不要になった選手情報を削除できる（関連する成績データの扱いを考慮する論理削除/物理削除方針の策定）。

### 4.2. 成績登録機能 (Stats Input)
- **F-05: 試合成績入力**: 選手を選択し、指定した試合での打撃成績を入力できる。
  - **入力項目**: 試合日、対戦相手、打数(AB)、安打数(H: 単打/二塁打/三塁打/本塁打の内訳)、四死球数(BB/HBP)、犠飛数(SF)、打点(RBI)、三振数(SO)

### 4.3. 成績集計・可視化機能 (Stats Aggregation)
- **F-06: 指標自動計算**: 記録された成績から以下の指標を動的に算出する。
  - **打率 (BA)** = $\frac{\text{安打}}{\text{打数}}$
  - **出塁率 (OBP)** = $\frac{\text{安打} + \text{四死球}}{\text{打数} + \text{四死球} + \text{犠飛}}$
  - **長打率 (SLG)** = $\frac{\text{単打} + (2 \times \text{二塁打}) + (3 \times \text{三塁打}) + (4 \times \text{本塁打})}{\text{打数}}$
  - **OPS** = 出塁率 + 長打率
- **F-07: ランキング表示**: 規定打席以上の選手を各種指標（打率順、OPS順など）でソート表示できる。
- **F-08: 打球方向表示**: 野球場のどの位置に打球が飛んだのかを、打撃成績（左中二、中前など）をもとに表示する。

## 5. データモデル・データ辞書 (Data Dictionary)

| テーブル | 論理項目名 | 物理項目名 | 型 | 制約 | 備考 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **players** | 選手ID | `id` | BIGINT | PK, Auto Increment | 主キー |
| | 選手名 | `name` | VARCHAR(50) | NOT NULL | |
| | 背番号 | `uniform_number` | INT | NOT NULL | 1〜99 |
| | 投打 | `throws_bats` | VARCHAR(10) | NULL可 | 例: 右投右打 |
| **batting_records** | 成績ID | `id` | BIGINT | PK, Auto Increment | 主キー |
| | 選手ID | `player_id` | BIGINT | FK, NOT NULL | players(id)を参照 |
| | 試合日 | `game_date` | DATE | NOT NULL | YYYY-MM-DD |
| | 対戦相手 | `opponent` | VARCHAR(100) | NULL可 | |
| | 打数 | `at_bats` | INT | NOT NULL | デフォルト: 0 |
| | 単打 | `singles` | INT | NOT NULL | デフォルト: 0 |
| | 二塁打 | `doubles` | INT | NOT NULL | デフォルト: 0 |
| | 三塁打 | `triples` | INT | NOT NULL | デフォルト: 0 |
| | 本塁打 | `home_runs` | INT | NOT NULL | デフォルト: 0 |
| | 四死球 | `walks` | INT | NOT NULL | デフォルト: 0 |
| | 犠飛 | `sac_flies` | INT | NOT NULL | デフォルト: 0 |

## 6. 非機能要件 (Non-Functional Requirements)

### 6.1. 可用性・信頼性 (Availability)
- 入力エラー時（ゼロ除算が発生する計算：打数が0の状態で打率を求める等）にシステムがクラッシュせず、`0.000` または `---` を表示する例外処理を実装する。

### 6.2. セキュリティ (Security)
- **SQLインジェクション対策**: JPA / ORMを利用したパラメータバインディングを徹底する。
- **XSS（クロスサイトスクリプティング）対策**: Thymeleafのエスケープ表示（`th:text`）を使用する。

### 6.3. 保守性・運用ルール (Maintainability & Git Rules)
- **ブランチ運用方針**: Git Flow風の運用の適用
  - `main`: 本番稼働用ブランチ
  - `develop`: 開発統合ブランチ
  - `feature/機能名`: 個別機能開発ブランチ（例: `feature/player-crud`）
- **コミットログ規約**: Prefixesを付与（`feat:`, `fix:`, `docs:`, `refactor:` など）

---

## 7. 画面一覧および画面遷移図 (Screen Flow)

1. **S-01: ダッシュボード/成績ランキング画面 (`/`)**
   - 画面遷移: メニューから選手一覧・登録画面へ遷移可能。
2. **S-02: 選手管理画面 (`/players`)**
   - 新規選手登録フォームおよび既存選手テーブルを表示。
3. **S-03: 成績入力画面 (`/records/new`)**
   - ドロップダウンで選手を選択し、試合結果数値を入力して保存。