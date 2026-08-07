# 非エンジニア向けリリースノート
## テンプレート
```
# {リポジトリ名} のリリース

## ユーザーへの影響あり
### Feature: ブログ記事をリアルタイムで編集できるように
* [github PR](https://github.com/{organization-name}/{repository-name}/pull/{pull-request-id})
  * ...(PRが複数ある場合はリンクを追加)

**変更内容**
* Before: 編集のたびに手動で記事を保存する必要があった。
* After: 編集を10秒止めると、記事が自動保存される。
* 備考: 有料会員のみ対象。無料会員では無効。

<必要に応じてスクリーンショットまたは動画>

### Bugfix: 絵文字が表示されない
* [github PR](https://github.com/{organization-name}/{repository-name}/pull/{pull-request-id})
  * ...(PRが複数ある場合はリンクを追加)

**変更内容**
* Before: vx.x アップデート以降、絵文字が表示されなかった。
* After: 記事内で絵文字が表示される。
* 備考: なし。

<必要に応じてスクリーンショットまたは動画>

## ユーザーへの影響なし
### Refactor: 管理者によるBAN
* [github PR](https://github.com/{organization-name}/{repository-name}/pull/{pull-request-id})
  * ...(PRが複数ある場合はリンクを追加)

**変更内容**
* Before: 管理者が悪質なユーザーを即時BANできなかった。
* After: 管理画面の「BAN管理」タブから、メールアドレス指定でBANできる。
* 備考: なし。

<必要に応じてスクリーンショットまたは動画>

### Other: 依存ライブラリの更新
* [github PR](https://github.com/{organization-name}/{repository-name}/pull/{pull-request-id})
  * ...(PRが複数ある場合はリンクを追加)

**変更内容**
* Before: 一部のライブラリが古かった。
* After: ライブラリを最新化。挙動の変更なし。
* 備考: なし。
```

## 備考
AI はスクリーンショットや動画を撮る必要はない。`<必要に応じてスクリーンショットまたは動画>` とだけ書くこと。
