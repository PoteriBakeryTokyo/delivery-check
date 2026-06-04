# Poteri Bakery Tokyo 配送チェックアプリ

## システム構成

- **フロントエンド**: `index.html`（GitHub Pages, publicリポジトリ）
- **バックエンド**: Google Apps Script（GASエディタで直接管理、リポジトリ外）
- **通知**: LINE Messaging API（Push messages to group）
- **ストレージ**: Google Sheets + Google Drive

## GAS変更時のルール（重要）

GASコードはリポジトリ管理外のため、変更ミスが発見しにくい。以下を必ず守ること。

### 1. doPostルーティングを必ず確認する

GASには `doPost` 関数内に `switch (payload.action)` のルーティングテーブルがある。
関数を**追加・削除・リネーム**した場合、必ずルーティングも合わせて更新する。

```
action名         → 呼び出し関数
createrow        → handleCreateRow
updatecol        → handleUpdateCol
uploadphoto      → handleUploadPhoto
sendyokomail     → handleYokoMail       ← リネーム済（旧: handleSendYokoMail）
sendreturnmail   → handleReturnMail
sendshiimail     → handleSendShiiMail
load             → handleLoad
ping             → handlePing
```

### 2. GAS変更を依頼する前にdoPostを確認する

AIがGAS関数の変更・追加を提案する際は、先にユーザーに `doPost` 関数のコードを共有してもらい、
ルーティングとの整合性を確認してから変更内容を伝える。

### 3. 変更後は必ずGASテスト関数で動作確認する

各アクションに対応するテスト関数をGASエディタで実行して確認する。

```javascript
// 横浜到着LINEテスト
function testHandleYokoMail() {
  handleYokoMail({ name: 'テスト', yokoArrival: '11:45～' });
}
// 返品LINEテスト
function testHandleReturnMail() {
  handleReturnMail({ name: 'テスト', returnPhotoUrls: '' });
}
```

## エラー設計の原則

- GASは `try-catch` でエラーを捕捉し `{ status: 'error', message: '...' }` を返す
- クライアント側 `postToGAS` は `status: 'error'` を受け取ったら `throw` する（実装済）
- これにより、GAS内部エラーが「✓ 送信済」と誤表示されることを防ぐ

## 過去のトラブル事例

| 日付 | 事象 | 根本原因 | 対策 |
|---|---|---|---|
| 2026-06 | 横浜到着LINEが届かない | ①LINE_TOKEN失効 ②`doPost`の`sendyokomail`が旧関数名`handleSendYokoMail`を参照 | ①トークン再発行 ②ルーティング修正 ③`postToGAS`にエラーチェック追加 |

## ブランチ運用

- 本番: `main`（GitHub Pages で公開）
- 開発: `claude/...` ブランチで作業後、確認済みのものをmainにマージ
- 保留中: `claude/add-ikebukuro-section`（池袋セクション追加、ユーザー判断待ち）
