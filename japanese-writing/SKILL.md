---
name: japanese-writing
description: Methods of Japanese writing. Run only when the user explicitly calls this skill.
---

# Japanese writing
## GOAL
Write Japanese output according to the Methods.

## Why
At first, **the goal of writing is to move people**.

However, LLM output tends to
- Treat writing a lot as the goal itself
  - And assume more volume is better
- As a result, neither the reader nor the LLM knows "what is being conveyed"

This is fatal because in Japanese the point gets harder to grasp as the text gets longer.

So, change the output of Japanese following methods.

## Method-1: Within 50 characters in one sentence.
**Why**:
Because Japanese lets you stretch a sentence endlessly, and the point gets lost.

**Detail**
When an explanatory sentence exceeds 50 characters, these problems occur:

- The points are not organized
- The explanation is not split properly

**How to follow method**:
- Use bullet points
- Put one piece of information in one bullet
- Explain each bullet within 50 characters

**Example**:
NG:
```markdown
ソースコードは多くの人間が編集するとスパゲティコードになりやすく、分かりにくくなる。大規模開発になればなるほど痛感されることである。従って「分かりやすい変数を書くこと」「クラスの責務を分け、他設計をすること」が求められる
```

OK:
```markdown
ソースコードを編集するときは、次を意識して複雑化を防止する
- 変数名をわかりやすくすること
- クラスの責務を分けること
```

## Method-2: Write "point" first
**Why**
Because of how Japanese works, the point tends to land at the end of a sentence.

**Detail**
English is a language for explaining and moving people. Japanese is a language for decorating words for emotion.

So the following can happen:

- The point lands at the end of the sentence
- What you want to say is located in the middle of the sentence

Example: the point is "did you check the logs". Decoration comes before it.
```text
これまでの操作履歴から観察される限りは、運用者による手動操作はないとは信じていますが、最初にログの確認をされているかどうかが確認したいことです。
```

**How to follow method**:
- Write the point first
- Put any extra explanation after it, as bullet points

**Example**:
NG:
```markdown
今回のマイグレーション失敗問題については、早朝帯に作業していたことや緊急で作業し焦りがあったことも原因だと考えられますが、まず何よりマイグレーションのスクリプトが正しく書かれていませんでした。
```

OK:
```markdown
今回のマイグレーション失敗問題の根本原因は次の通り
- マイグレーションのスクリプトが正しく書かれていなかったこと

根本原因と関連して、次も原因の候補として挙げたい
- 早朝の緊急作業で焦りがあったこと
```