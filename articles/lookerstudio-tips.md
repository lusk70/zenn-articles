---
title: "Looker Studioのスパークラインの一癖" # 記事のタイトル
emoji: "📈" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["lookerstudio", "bi", "visualization"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

こんにちは。らすく([@lusk_eng](<[lusk_eng](https://twitter.com/lusk_eng)>))です。

Looker Studio のスパークラインの表示に関して、困った事例を紹介します。

# スパークラインとは

Looker Studio のスコアカードの機能で、スコアカードの値とその値の変動をグラフで表示する機能です。
![サンプル画像](https://cloud.google.com/static/looker/docs/studio/images/scorecard-sparkline-2023-04-12.png?hl%3Dja)

# 困ったこと

:::message alert
本来なら、正常に日毎の変動グラフが描画されるのに、変動のないグラフになっていることに気づいた。
:::

![エラー](/images/lookerstudio-error/error-graph.png)
_意図しない表示_

なぜか、時系列で変動のあるグラフが表示されるのに、変動のない横一線のグラフになってしまっていますね。

# 原因

値がマイナス値を含む場合に、正常な描画ができないようです。

![エラー](/images/lookerstudio-error/error-data.png)
_マイナス値がある場合_

しかし、null 値を含む場合は、描画自体は正常にできるみたいですね。
![nullデータ](/images/lookerstudio-error/null-data.png)
_null を含む場合_

# 解決策

原因はデータ品質が悪いことによるものなので、正しいデータにしておけよって話ですね。
データ品質を確保するために事前にデータに対してテストを実施しておけば、不慮のエラーは防げそうです。

# まとめ

Looker Studio のスパークラインを使う際には、データのマイナス値に注意が必要ということがわかりました。気をつけましょう！

# 公式ドキュメント

https://cloud.google.com/looker/docs/studio/scorecard-reference?hl=ja&visit_id=638733089091063541-2344269568&rd=1#sparkline
