---
title: "dbt Cloud から dbt Cloud CLI に移行" # 記事のタイトル
emoji: "📺" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["dbt", "dbtcloud", "dbtcloudcLI"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

こんにちは。らすく([@lusk_eng](<[lusk_eng](https://twitter.com/lusk_eng)>))です。

dbt Cloud（GUI 操作） から dbt Cloud CLI に移行したので、その際のメモを残しておきます。

# 移行の背景

dbt Cloud では GUI 操作であるため、開発速度に課題を感じていました。

具体的には、マルチテナント運営をしている自社都合によりテナントが増えた際に、同じようなファイルをコピーして作業することが多く、GUI 上での開発ではやや面倒なことが多かったのです。

その点、コードエディタ上で開発できる CLI であれば、Python ライブラリの利用や、Copilot や Cursor などの AI コード補助ツールを利用して開発速度を上げることができると考えました。

# 移行の手順

**前提**

- dbt = 1.8.8
- bigquery = 1.8.3

## dbt Cloud CLI を利用する場合の設定

このあたりは公式ドキュメントを読んでもらえれば問題ないかと思います。

https://docs.getdbt.com/guides/dbt-cloud-cli

もしくは、dbt Cloud の設定画面に手順があるので、そちらからでも良いと思います。

https://cloud.getdbt.com/settings/profile/cloud-cli

手順通りで OK。難しいことはないです！

---

## dbt Cloud から dbt Cloud CLI に移行して困ったこと

ここからが本題で、dbt Cloud CLI に移行して困ったことがあったので、まとめていきます。

:::message alert
dbt コマンドで本番環境を直叩きしてしまう。
:::

## 結論

:::message alert
profiles.yml だけでは反映されない可能性があるので各種設定を確認する必要がある。
:::

dbt の設定ファイルは、以下の適用の優先順位となっている。

> 1. モデルレベルの設定
> 2. `dbt_project.yml` の設定
> 3. `profiles.yml` の設定

このことから、`dbt_project.yml` に明示的な設定があるとそちらを参照してしまう。

---

と思いきや、大事なのは以下のスキーマを決定するファイルです。

```sql :get_custom_schema.sql
{% macro generate_schema_name(custom_schema_name, node) -%}

	{%- set default_schema = target.schema -%}
	{%- if custom_schema_name is none -%}

		{{ default_schema }}

	{%- else -%}

		{{ default_schema }}_{{ custom_schema_name | trim }}

	{%- endif -%}

{%- endmacro %}
```

[How does dbt generate a model's schema name?](https://docs.getdbt.com/docs/build/custom-schemas#how-does-dbt-generate-a-models-schema-name)

`target`: profiles.yml に設定した情報を参照
`custom_schema_name`: dbt_project.yml に設定した shema 情報を参照

ということから、デフォルトの設定では、dbt_project.yml に設定がなければ profiles.yml を参照します。
dev 用として、profiles.yml を参照させたい場合は、このマクロファイルをカスタムする必要がありました。

```sql title="get_custom_schema.sql"
{% macro generate_schema_name(custom_schema_name, node) -%}

	{%- set default_schema = target.schema -%}
+	{%- if custom_schema_name is none or target.name == 'dev' -%}
-	{%- set default_schema = target.schema -%}

		-- devの場合は、profiles.ymlの設定値を参照させたい
		{{ default_schema }}

	{%- else -%}

		-- dbt_project.ymlの設定値を参照させたい
		{{ default_schema }}_{{ custom_schema_name | trim }}

	{%- endif -%}

{%- endmacro %}
```

# 最後に

ローカル環境では、ルートディレクトリにある `dbt_project.yml` を参照してくれると思うのですが、チーム運営する上でローカル環境の設定ファイルの横展が面倒なので、この辺りを自動的にやってくれる仕組みがあると嬉しいなと思います。

もし、ご存知の方がいらっしゃったらご教示いただけると嬉しいです。
