# ec-data-platform-dataform

Olist Brazilian E-Commerce データプラットフォームの **Dataform 代替実装**です。

## 概要

本プロジェクトは [`ec-data-platform`](https://github.com/torimimasataka/ec-data-platform) で構築した dbt パイプラインを、Google Cloud Dataform で再実装したものです。

> **注意**: dbt 版と Dataform 版は同時運用を想定していません。本番環境ではどちらか一方を選択して使用してください。

## 技術スタック

- **Dataform** (Google Cloud Dataform)
- **BigQuery** (GCPプロジェクト: `ec-data-platform-2026`)
- **データセット**: US リージョン

## データセット構成

| レイヤー | データセット | 説明 |
|---|---|---|
| 01_sources | `01_import` | ローデータへの参照（declaration + src views） |
| 02_preprocess | `02_preprocess` | クレンジング済みテーブル |
| 03_unification | `03_unification` | 統合テーブル（全テーブルを JOIN した幅広テーブル） |
| 04_intermediate | `04_intermediate` | 中間集計テーブル |
| 05_application | `05_application` | ダッシュボード向け最終出力テーブル |

## プロジェクト構造

```
definitions/
├── 01_sources/
│   ├── raw_*.sqlx           # declaration: 既存BQテーブルへの参照
│   └── src_*.sqlx           # view: ソースデータの選択
├── 02_preprocess/
│   └── prep_*.sqlx          # table: クレンジング・型変換
├── 03_unification/
│   └── unf_order_items.sqlx # table: 全テーブルを統合した幅広テーブル
├── 04_intermediate/
│   └── t_*.sqlx             # table: 分析軸別の集計テーブル
└── 05_application/
    └── app_*.sqlx           # table: ダッシュボード向け最終テーブル
```

## 実行方法

### 前提条件

- Google Cloud SDK がインストール済みであること
- Dataform CLI がインストール済みであること
- BigQuery への書き込み権限があること

### セットアップ

```bash
# Dataform CLI のインストール
npm install -g @dataform/cli

# 依存関係のインストール
cd ec-data-platform-dataform
npm install

# 認証情報の設定
dataform init-creds bigquery
```

### 実行

```bash
# ドライラン（実行計画の確認）
dataform run --dry-run

# 全テーブルの実行
dataform run

# 特定のテーブルのみ実行
dataform run --actions prep_orders

# タグ指定実行（タグが設定されている場合）
dataform run --tags preprocess
```

## dbt との対応関係

| dbt 構文 | Dataform 構文 |
|---|---|
| `{{ config(materialized='table') }}` | `config { type: "table" }` |
| `{{ config(materialized='view') }}` | `config { type: "view" }` |
| `{{ ref('table_name') }}` | `${ref("table_name")}` |
| `{{ source('olist_raw', 'table') }}` | `${ref("table")}` (declaration で定義) |

## データソース

[Olist Brazilian E-Commerce Public Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) を使用しています。
