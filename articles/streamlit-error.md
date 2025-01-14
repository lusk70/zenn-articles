---
title: "Streamlitでのファイルアップロード時のエラー解決" # 記事のタイトル
emoji: "🐕" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["streamlit", "python", "エラー解決"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

こんにちは。らすく([@lusk_eng](<[lusk_eng](https://twitter.com/lusk_eng)>))です。

Streamlit を使っていた際に、ファイルアップロード時に謎なエラーが出たので、解決策を残しておきます。

# エラー内容

```
AxiosError: Request failed with status code 403
```

![エラー画面](/images/streamlit-error/streamlit_error.png)

# 実行環境

- Streamlit: 1.33.0
- Python: 3.10.10
- ローカル環境: MacOS

# 解決策

streamlit のバージョンを 1.29.0 以前にダウングレードする。

```
pip uninstall streamlit
pip install streamlit==1.29.0
```

コミュニティをのぞいていると、ずっとこの問題は残り続けているみたいですね。
設定ファイルでセキュリティレベルを下げるよりかは、バージョンを下げる方がまだ安心ですね。

# 参考

https://discuss.streamlit.io/t/file-upload-fails-with-error-request-failed-with-status-code-403/27143/53
