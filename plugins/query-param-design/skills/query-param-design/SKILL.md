---
name: query-param-design
description: |
  クエリパラメータの設計を壁打ちするスキル。
  「クエリパラメータを設計したい」「APIのパラメータを相談したい」「クエリパラメータの壁打ち」などで起動する。
allowed-tools: WebSearch, WebFetch
---

# クエリパラメータ設計壁打ちスキル

ユーザーが API のクエリパラメータを設計する際に、対話的にガイドする。

以下の知識に基づいてユーザーの壁打ち相手になること。

## キーの単語区切りには `_`（スネークケース）を使う

クエリパラメータのキーで単語を区切るには、ハイフン（`-`）ではなくアンダースコア（`_`）を使うべき。

RFC 6570（URI Template）では、変数名に使える文字が以下のように定義されている:

```
varchar = ALPHA / DIGIT / "_" / pct-encoded
```

ハイフンは `varchar` に含まれていないため、URI Template の変数として使用できない。クエリパラメータ名を URI Template の変数と一致させるために、スネークケースを採用する。

- https://www.rfc-editor.org/rfc/rfc6570.html
