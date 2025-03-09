# projectbrief.md

## 1. プロジェクト概要

### プロジェクト名
仮称: **Prototype Scaffolder CLI**

### 背景
- システム開発のプロトタイプを素早く作りたいが、毎回の環境構築や初期設定が面倒で生産性が落ちている。
- Docker コンテナをベースにすることで、開発環境を共有しやすくしたい。
- コマンド一つでフロントエンド・バックエンド・DB のコンテナが立ち上がるアプリケーションの“原型”を生成するツールが欲しい。

### 目的
- コマンド (`prototype init` など) を叩くだけで、React + Laravel + MySQL のコンテナ群を含むひな形が自動生成される。
- 今後は Next.js, Ruby on Rails, Vue.js, Svelte, Redis などにも対応可能な拡張性を持たせる。
- これによってプロトタイプ作成や新規プロジェクトの立ち上げを極力短縮する。

### ユースケース例
1. 新規サービスのアイデア検証やデモ用プロトタイプをすぐに作りたい。
2. 新しいメンバーがプロジェクトに参加するとき、環境構築の手間を減らしたい。
3. チーム内で共通のテンプレートを整備し、プロトタイプ作成を標準化したい。

---

## 2. 技術概要

### 2.1 CLIツール本体
- **コマンド定義・CLIエントリポイント**
  - 例: `prototype init [--frontend=react] [--backend=laravel] [--db=mysql]`
  - `init`, `ls (list)`, `help`, `plugin install`, `plugin ls` などのサブコマンドを用意する想定。
- **テンプレート展開機能**
  - 選択した技術スタックのひな形ディレクトリをコピー or スキャフォールド。
  - `.env` や `docker-compose.yml` 内の各種変数を書き換えて生成。
- **Docker 実行補助**
  - `docker-compose build` や `docker-compose up` を実行するためのスクリプト。
  - プロジェクトディレクトリに移動して実行するだけで環境が起動する。
- **プラグイン (テンプレートモジュール) 管理**
  - React / Laravel / MySQL などを「プラグイン」と見なして組み合わせ可能にする。
  - CLI 本体から切り離して実装しやすくすることで、他スタックへの拡張を容易にする。

### 2.2 テンプレート構造
- **base/**  
  - `docker-compose.yml` の大枠を保管。コンテナ連携やネットワーク設定を定義。
- **frontend/ (react-vite-tailwind など)**  
  - フロントエンド用の Dockerfile, package.json, vite.config.ts, Tailwind 設定などを格納。
  - React のサンプルコンポーネントなどを含む。
- **backend/ (laravel など)**  
  - バックエンド用の Dockerfile, composer.json, Laravel のベースアプリケーションを格納。
- **db/ (mysql など)**  
  - DB 用 Dockerfile, 初期化用 SQL, .env などを格納。

### 2.3 主要ファイルと書き換えトークン例
- **docker-compose.yml**  
  - `${PROJECT_NAME}`, `${FRONTEND_SERVICE}`, `${BACKEND_SERVICE}`, `${DB_SERVICE}` などを変数化し、CLI が適切に置換。
- **.env テンプレート**  
  - DB 接続情報やポート番号など、ユーザーが指定した内容を反映。
- **package.json / composer.json**  
  - プロジェクト名やバージョンを可変にする場合はトークンを使用する。

### 2.4 拡張性とプラグイン
- **プラグインの追加方法**
  1. テンプレートファイル一式 (Dockerfile, 各設定ファイル, 初期ソース) を用意。
  2. `plugin.json` (プラグイン名, バージョン, 依存プラグイン, ディレクトリ構成など) を作成。
  3. CLI にプラグインを登録する (`prototype plugin install <PLUGIN_PATH>` を想定)。
- **テンプレート同士の依存管理**
  - Next.js + Laravel + MySQL などを同時利用する際、ポート番号の衝突防止やコンテナ名の重複防止を考慮。
  - CLI が最終的にマージした `docker-compose.yml` を生成してくれる。

---

## 3. ディレクトリ構造

以下はプロジェクト全体の一例です。CLI のソースコードと、テンプレートを管理するディレクトリを分割しています。
```
project-root/
├─ cli/
│   ├─ commands/
│   │   ├─ init.js
│   │   ├─ ls.js
│   │   └─ plugin.js
│   ├─ utils/
│   │   ├─ fileGenerator.js
│   │   ├─ configLoader.js
│   │   └─ dockerComposeMerger.js
│   ├─ index.js                 # CLI エントリポイント
│   └─ package.json             # CLI ツールの依存管理 (Node.js 前提の場合)
│
├─ templates/
│   ├─ base/
│   │   └─ docker-compose.yml
│   ├─ frontend/
│   │   └─ react-vite-tailwind/
│   │       ├─ Dockerfile
│   │       ├─ package.json
│   │       ├─ vite.config.ts
│   │       ├─ tailwind.config.js
│   │       └─ src/
│   │           └─ (React のサンプルコード)
│   ├─ backend/
│   │   └─ laravel/
│   │       ├─ Dockerfile
│   │       ├─ composer.json
│   │       ├─ .env.example
│   │       └─ (Laravel の初期コード一式)
│   └─ db/
│       └─ mysql/
│           ├─ Dockerfile
│           └─ init.sql
│
├─ plugins/
│   └─ (将来的に追加される Next.js などのテンプレートモジュールを配置)
│       └─ nextjs/
│           ├─ Dockerfile
│           ├─ package.json
│           ├─ plugin.json     # プラグイン定義
│           └─ src/
│               └─ (Next.js のサンプルコード)
│
├─ docs/
│   └─ projectbrief.md          # 本ドキュメント (設計概要等)
│
└─ README.md                    # リポジトリの使い方やセットアップ手順など
```
### 補足
- `cli/` 以下に CLI の実装をまとめます。Node.js + npm (or Yarn) + TypeScript で CLI を作る場合は、適宜 `index.ts` にするなど構成を調整してください。
- `templates/` 以下には、サービス単位 (frontend, backend, db) や、そのバージョンやフレームワーク単位 (react-vite-tailwind, laravel) でディレクトリを切ります。
- プラグインは将来的に独立したリポジトリとして管理できるようにしておくことも想定可能です (npm パッケージ化等)。
- `docs/` ディレクトリには上記のようなドキュメントファイルをまとめるとわかりやすいです。

---

## 4. 今後の展開

- **他フレームワークの追加**  
  - Next.js, Ruby on Rails, Flask, Vue.js, Angular, Svelte など、プラグインを追加していく。
- **データストアの追加**  
  - PostgreSQL, MongoDB, Redis などを追加し、より複雑なサービス構成にも対応。
- **プラグインのバージョン管理・依存解決**  
  - CLI ツールがプラグイン同士の依存関係を管理し、衝突を防ぐ仕組みを強化。
- **CLI ツールの配布方法**  
  - npm パッケージ化、Homebrew Tap を用いた配布などで社内外に公開。
- **ドキュメント整備**  
  - チームメンバーが自作のテンプレートを追加する際のガイドを用意。

これらを踏まえ、まずは React + Laravel + MySQL という最小限の範囲で動く CLI とテンプレートを実装・検証し、使い勝手やテンプレート設計のベストプラクティスを固めていきます。