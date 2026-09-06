---
name: japanese-writing
description: Methods of Japanese writing. Run only when the user explicitly calls this skill.
---

# Japanese writing
## GOAL
Write Japanese output according to the "Methods".

## Why
At first, **the goal of writing is to move people**.

However, LLM tends to set goal of writing that "writing a lot", and assumes more volume is better.

As a result, reader and the LLM itself suffers with "what is being told" because of the babel.

This is fatal especially in Japanese cause of the point gets harder to grasp as the text gets longer.

So, change the output of Japanese following methods.

## Method-1: Within 50 characters in one sentence.
**Why**:
Japanese lets you stretch a sentence endlessly, and the point gets lost.

**Detailed reason**
When an explanatory sentence exceeds 50 characters, these problems occur:

- The points are not organized
- So, reader has cost about thinking what is discussed

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
ソースコードを編集するときは、次の観点で複雑化を防止する
- 変数名をわかりやすくすること
- クラスの責務を分けること
```

## Method-2: Write "point" first
**Why**
Because of Japanese feature, the point tends to land at the end of a sentence.

**Detailed reason**
English is a language for explaining and moving people, opposite of Japanese is a language for decorating words for emotion.

So the following can happen:

- The point is tended to land at the end of the sentence
- So, reader should reconsider "what is the point?"

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
先日のマイグレーション失敗問題については、早朝帯に作業していたことや緊急で作業し焦りがあったことも原因だと考えられますが、まず何よりマイグレーションのスクリプトが正しく書かれていませんでした。
```

OK:
```markdown
先日発生したマイグレーション失敗問題の「根本原因」は次の通り
- マイグレーションのスクリプトが正しく書かれていなかったこと

根本原因とは別に、次も原因の候補として挙げたい
- 早朝の緊急作業で焦りがあったこと
```

## Method-3: Use plain form(だ/である) not polite form(です/ます)
**Why**
Because polite form(です/ます) of Japanese stretches the text long.

**Detailed reason**
It's thought that polite form is good for Japanese.

But in explanation, it can happen:

- a little longer than plain form
- So, reader feels "a little bit longer and hard to read"

Example:
```text
不具合が発生していた原因をご説明致します
- 昨日のデプロイ内容に不備があることが分かりました
- 根本原因はテスト不足でございます
```

**How to follow method**:
- Use plain form(だ/である)

**Example**:
NG:
```markdown
不具合が発生していた原因をご説明致します
- 昨日のデプロイ内容に不備があることが分かりました
- 根本原因はテスト不足でございます
```

OK:
```markdown
不具合発生の原因説明
- 昨日のデプロイ内容に不備があった
- 根本原因はテスト不足である
```