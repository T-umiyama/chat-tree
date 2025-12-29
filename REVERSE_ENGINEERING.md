# ChatTree リバースエンジニアリング ドキュメント

## 📋 目次
1. [プロジェクト概要](#プロジェクト概要)
2. [アーキテクチャ](#アーキテクチャ)
3. [技術スタック](#技術スタック)
4. [ディレクトリ構造](#ディレクトリ構造)
5. [主要コンポーネント](#主要コンポーネント)
6. [データフロー](#データフロー)
7. [Chrome拡張機能の仕組み](#chrome拡張機能の仕組み)
8. [OpenAI vs Claude サポート](#openai-vs-claude-サポート)
9. [ノード作成とレイアウト](#ノード作成とレイアウト)
10. [ナビゲーションシステム](#ナビゲーションシステム)
11. [エクスポート機能](#エクスポート機能)
12. [ビルドプロセス](#ビルドプロセス)

---

## プロジェクト概要

**ChatTree** は、ChatGPT（OpenAI）とClaude.ai（Anthropic）の会話をインタラクティブなツリー構造で可視化するChrome拡張機能です。

### 主要機能
- 📊 **グラフ可視化**: 会話を対話型グラフとして表示
- 🔀 **非線形ナビゲーション**: 会話の任意の部分に直接ジャンプ
- 🔍 **検索機能**: メッセージを検索して特定の内容を見つける
- 📤 **エクスポート**: Markdown、Obsidian、XML形式で会話をエクスポート
- 🤝 **クロスプラットフォーム**: OpenAI ChatGPTとAnthropic Claude.aiの両方に対応
- ✏️ **メッセージ編集**: 過去のメッセージを編集して会話を分岐
- 💬 **分岐応答**: 任意のメッセージに対して新しい応答を生成

---

## アーキテクチャ

### 高レベルアーキテクチャ

```
┌─────────────────────────────────────────────────┐
│         Chrome Side Panel (React UI)           │
│  ┌──────────────────────────────────────────┐  │
│  │  ConversationTree Component              │  │
│  │  - ReactFlow Graph Visualization         │  │
│  │  - Node & Edge Management                │  │
│  │  - User Interactions                     │  │
│  └──────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
                     ↕
┌─────────────────────────────────────────────────┐
│      Chrome Extension Background Script        │
│  ┌──────────────────────────────────────────┐  │
│  │  - Header Capture (webRequest API)       │  │
│  │  - Message Routing                       │  │
│  │  - DOM Manipulation (scripting API)      │  │
│  │  - Conversation Fetching                 │  │
│  └──────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
                     ↕
┌─────────────────────────────────────────────────┐
│          ChatGPT / Claude.ai Page              │
│  - DOM Elements (article[data-testid])         │
│  - Conversation UI                             │
│  - Navigation Buttons                          │
└─────────────────────────────────────────────────┘
```

### データフロー

```
1. ユーザーがChatGPT/Claudeページを開く
   ↓
2. Background Scriptが認証ヘッダーをキャプチャ
   ↓
3. Side Panelが会話履歴を要求
   ↓
4. Background ScriptがAPI経由で会話データを取得
   ↓
5. Side Panelがノードとエッジのグラフとしてレンダリング
   ↓
6. ユーザーがノードをクリック
   ↓
7. ナビゲーションステップを計算
   ↓
8. Background ScriptがDOMを操作してページ上でナビゲート
```

---

## 技術スタック

### フロントエンド
- **React 19.1.0** - UIフレームワーク
- **TypeScript 5.8.3** - 型安全性
- **@xyflow/react 12.6.0** - グラフ可視化ライブラリ
- **@dagrejs/dagre 1.1.4** - グラフレイアウトアルゴリズム
- **react-markdown 10.1.0** - Markdownレンダリング
- **Tailwind CSS 3.4.17** - スタイリング

### ビルドツール
- **Vite 6.3.4** - ビルドツールとバンドラー
- **PostCSS 8.4.32** - CSS処理
- **ESLint 8.57.1** - コード品質

### Chrome APIs
- **chrome.storage** - セッションストレージ
- **chrome.tabs** - タブ管理
- **chrome.webRequest** - ヘッダーキャプチャ
- **chrome.scripting** - DOM操作
- **chrome.sidePanel** - サイドパネルUI
- **chrome.runtime** - メッセージング

---

## ディレクトリ構造

```
chat-tree/
├── public/                      # 静的アセット
│   ├── manifest.json           # Chrome拡張機能マニフェスト
│   └── logo*.png              # 拡張機能アイコン
├── src/
│   ├── components/            # Reactコンポーネント
│   │   ├── ConversationTree.tsx    # メイングラフコンポーネント
│   │   ├── CustomNode.tsx          # カスタムノード表示
│   │   ├── ContextMenu.tsx         # 右クリックメニュー
│   │   ├── SearchBar.tsx           # 検索インターフェース
│   │   ├── ExportButton.tsx        # エクスポート機能
│   │   ├── ExportModal.tsx         # エクスポートダイアログ
│   │   ├── CopyButton.tsx          # コピー機能
│   │   ├── CopyModal.tsx           # コピーダイアログ
│   │   ├── LoadingStates.tsx       # ローディングUI
│   │   └── RefreshButton.tsx       # リフレッシュボタン
│   ├── hooks/                 # カスタムReactフック
│   │   └── useConversationTree.ts  # 状態管理フック
│   ├── utils/                 # ユーティリティ関数
│   │   ├── nodeCreation.ts         # OpenAIノード作成
│   │   ├── claudeNodeCreation.ts   # Claudeノード作成
│   │   ├── nodeNavigation.ts       # OpenAIナビゲーション
│   │   ├── nodeNavigationClaude.ts # Claudeナビゲーション
│   │   ├── nodeProcessing.ts       # ノード処理ヘルパー
│   │   ├── conversationTreeHandlers.ts # イベントハンドラー
│   │   └── dagre.ts                # レイアウト設定
│   ├── types/
│   │   └── interfaces.ts           # TypeScript型定義
│   ├── constants/
│   │   └── constants.ts            # 定数定義
│   ├── background.ts          # Chrome拡張機能バックグラウンドスクリプト
│   ├── App.tsx               # メインReactアプリ
│   ├── main.tsx              # Reactエントリーポイント
│   └── index.css             # グローバルスタイル
├── scripts/
│   └── build.js              # ビルド後の処理スクリプト
├── vite.config.ts            # Vite設定
├── tailwind.config.cjs       # Tailwind設定
├── tsconfig.json             # TypeScript設定
└── package.json              # プロジェクト依存関係
```

---

## 主要コンポーネント

### 1. ConversationTree.tsx
メインのグラフ可視化コンポーネント。

**責務:**
- ReactFlowグラフのレンダリング
- ノードとエッジの状態管理
- ユーザーインタラクションの処理（クリック、コンテキストメニュー）
- 会話データの取得とリフレッシュ
- ノードの可視性更新

**主要な状態:**
```typescript
- nodes: ノードの配列
- edges: エッジの配列
- conversationData: 元の会話データ
- provider: 'openai' | 'claude'
- isLoading: ローディング状態
- menu: コンテキストメニュー状態
- lastActiveChildMap: 最後のアクティブな子ノードのマップ
- previousPathNodeIds: 以前のパス上のノードID
```

**主要な関数:**
- `handleRefresh()`: 会話履歴を取得
- `updateNodesVisibility()`: DOMでノードの可視性をチェック
- `handleNodeClick()`: ナビゲーションステップを計算
- `onNodeContextMenu()`: コンテキストメニューを表示

### 2. CustomNode.tsx
グラフ内の各会話メッセージを表すカスタムノードコンポーネント。

**機能:**
- メッセージコンテンツの表示（Markdownサポート）
- ロール別のスタイリング（user/assistant/system）
- ダブルクリックで全画面表示
- 可視性状態のビジュアル表示（grayscale for hidden）
- タイムスタンプ表示
- コンテンツタイプインジケーター

**スタイル:**
- 固定サイズ: 300x120px
- ユーザーメッセージ: 黄色の背景
- アシスタントメッセージ: グレーの背景
- 非表示メッセージ: グレースケール + ストライプパターン

### 3. ContextMenu.tsx
ノード上の右クリックメニュー。

**アクション:**
- **Select**: ノードにナビゲート
- **Edit this message** (ユーザーメッセージの場合): メッセージを編集して再送信
- **Respond to this message** (アシスタントメッセージの場合): 新しい応答を作成

**フロー:**
```
1. ユーザーが右クリック
2. メニュー表示
3. アクション選択
4. バックグラウンドにメッセージ送信
5. バックグラウンドがDOMを操作
6. UIをリフレッシュ
```

### 4. background.ts
Chrome拡張機能のバックグラウンドサービスワーカー。

**主要機能:**

#### ヘッダーキャプチャ
```typescript
captureHeaders() {
  // OpenAI APIリクエストから認証ヘッダーを傍受
  chrome.webRequest.onBeforeSendHeaders.addListener(...)
}

captureClaudeOrgId() {
  // Claude APIリクエストから組織IDを傍受
  chrome.webRequest.onBeforeRequest.addListener(...)
}
```

#### メッセージハンドラー
```typescript
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  switch(request.action) {
    case "getHeaders": // 保存されたヘッダーを返す
    case "fetchConversationHistory": // 会話データを取得
    case "checkNodes": // ノードがDOMに存在するかチェック
    case "checkNodesClaude": // Claudeノードをチェック
    case "editMessage": // メッセージを編集
    case "respondToMessage": // メッセージに応答
    case "executeSteps": // ナビゲーションステップを実行
    case "goToTarget": // ノードにスクロール
    // ... その他のアクション
  }
})
```

#### DOM操作
```typescript
// ページ上のネイティブイベントをトリガー
triggerNativeArticleEvents() {
  // すべてのarticle要素を見つけてホバーイベントを発火
  // これによりボタンが表示される
}

// メッセージを編集
editMessage(messageId, message) {
  // 1. 編集ボタンを見つけてクリック
  // 2. textareaに新しいメッセージを設定
  // 3. 送信ボタンをクリック
}

// ブランチを選択（ナビゲーション）
selectBranch(steps) {
  // 各ステップについて:
  // 1. 要素を見つける
  // 2. ホバーイベントをトリガー
  // 3. ナビゲーションボタン（左/右）をクリック
  // 4. DOM変更を待つ
}
```

---

## データフロー

### 会話データの取得

#### OpenAI (ChatGPT)
```typescript
1. ユーザーがchatgpt.comを訪問
2. バックグラウンドがwebRequestでヘッダーをキャプチャ
3. ヘッダーをchrome.storage.sessionに保存
4. UIが会話履歴を要求
5. バックグラウンドが保存されたヘッダーを使ってフェッチ:
   GET https://chatgpt.com/backend-api/conversation/{id}
6. レスポンスデータ構造:
   {
     mapping: { [nodeId]: { message, children, parent } },
     current_node: "最後のノードID",
     title: "会話タイトル"
   }
```

#### Claude (Anthropic)
```typescript
1. ユーザーがclaude.aiを訪問
2. バックグラウンドがAPIリクエストから組織IDをキャプチャ
3. UIが会話履歴を要求
4. バックグラウンドが認証情報（クッキー）を使ってフェッチ:
   GET https://claude.ai/api/organizations/{orgId}/chat_conversations/{id}?tree=True
5. レスポンスデータ構造:
   {
     chat_messages: [
       { uuid, text, content, sender, parent_message_uuid }
     ],
     current_leaf_message_uuid: "最後のメッセージID"
   }
```

### ノードの可視性チェック

**OpenAI:**
```typescript
checkNodesExistence(nodeIds) {
  // ページにスクリプトを注入
  // 各nodeIdについて:
  //   document.querySelector(`[data-message-id="${id}"]`)をチェック
  // null === 非表示, 存在 === 表示を返す
}
```

**Claude:**
```typescript
checkNodesExistenceClaude(nodeTexts) {
  // ページにスクリプトを注入
  // 各nodeTextについて:
  //   '.grid-cols-1'コンテナを検索
  //   テキストコンテンツを正規化して比較
  //   （HTMLエンティティ、Markdown、アーティファクトを処理）
  // 一致が見つからない === 非表示
}
```

---

## Chrome拡張機能の仕組み

### Manifest V3設定

```json
{
  "manifest_version": 3,
  "name": "ChatTree",
  "permissions": [
    "storage",      // ヘッダー/データの保存
    "tabs",         // タブ情報へのアクセス
    "webRequest",   // リクエストの傍受
    "scripting",    // DOM操作のためのスクリプト注入
    "activeTab",    // アクティブなタブとのインタラクション
    "sidePanel"     // サイドパネルUIの表示
  ],
  "host_permissions": [
    "https://chatgpt.com/backend-api/*",
    "https://claude.ai/api/organizations/*"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "side_panel": {
    "default_path": "index.html"
  }
}
```

### Side Panel統合

```typescript
// ユーザーがChatGPT/Claudeページに移動したとき
chrome.tabs.onUpdated.addListener((tabId, info, tab) => {
  if (url === ChatGPT || url === Claude) {
    chrome.sidePanel.setOptions({
      tabId,
      path: 'index.html',
      enabled: true
    })
  }
})

// 拡張機能アイコンをクリックしたときにパネルを開く
chrome.sidePanel.setPanelBehavior({
  openPanelOnActionClick: true
})
```

### メッセージパッシング

```typescript
// UIからバックグラウンドへ
const response = await chrome.runtime.sendMessage({
  action: "fetchConversationHistory"
})

// バックグラウンドでの処理
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === "fetchConversationHistory") {
    fetchConversationHistory()
      .then(data => sendResponse({ success: true, data }))
      .catch(error => sendResponse({ success: false, error }))
    return true // 非同期レスポンスのためにチャンネルを開いたまま
  }
})
```

---

## OpenAI vs Claude サポート

### データ構造の違い

**OpenAI (グラフベース):**
```typescript
interface OpenAIConversationData {
  mapping: {
    [nodeId]: {
      id: string
      message: { content, author, create_time }
      parent: string | null
      children: string[]
    }
  }
  current_node: string
}
```

**Claude (リストベース):**
```typescript
interface ClaudeConversation {
  chat_messages: Array<{
    uuid: string
    text: string
    content: ContentBlock[]
    sender: "human" | "assistant"
    parent_message_uuid: string
  }>
  current_leaf_message_uuid: string
}
```

### プロバイダー検出

```typescript
const isClaude = 'chat_messages' in response.data
setProvider(isClaude ? 'claude' : 'openai')
```

### ノード識別の違い

**OpenAI:** `data-message-id` 属性を使用
```typescript
document.querySelector(`[data-message-id="${messageId}"]`)
```

**Claude:** テキストコンテンツマッチングを使用
```typescript
// '.grid-cols-1'コンテナを検索
// 正規化されたテキストコンテンツを比較
containers.forEach(container => {
  if (normalizedText === normalizedTarget) {
    // 見つかった!
  }
})
```

### ナビゲーションの違い

**OpenAI:**
- ボタンの位置が固定
- ユーザー: [コピー, 編集, 左, 右]
- アシスタント: [..., 左, 右, コピー, いいね, 悪いね, 読み上げ, 再生成]

**Claude:**
- ボタンを動的に検索
- インデックス: 0 = 編集, 1 = 前へ, 2 = 次へ

---

## ノード作成とレイアウト

### OpenAI ノード作成 (nodeCreation.ts)

```typescript
createNodesInOrder(conversationData, checkNodes) {
  // 1. ルートノードを見つける（親なし）
  let rootNode = findFirstContentParent(...)

  // 2. 有効な子孫を再帰的に作成
  const findFirstValidDescendant = (node) => {
    // system/toolメッセージをスキップ
    // 有効なコンテンツを持つ最初の子孫を見つける
  }

  // 3. 各ノードについて:
  //    - 親/子の関係を設定
  //    - メッセージコンテンツを抽出
  //    - ノードデータを作成（ラベル、ロール、タイムスタンプ）
  //    - エッジを作成

  // 4. 可視性をチェック
  const existingNodes = await checkNodes(nodeIds)

  // 5. Dagreでレイアウト
  return layoutNodes(nodes, edges)
}
```

### Claude ノード作成 (claudeNodeCreation.ts)

```typescript
createClaudeNodesInOrder(conversationData, checkNodes) {
  // 1. ルートノードを作成
  const rootNode = { id: 'root', ... }

  // 2. 各chat_messageについて:
  messages.forEach(message => {
    // - parent_message_uuidを使ってノードを作成
    // - '00000000-0000-4000-8000-000000000000' === ルート
    // - 親の子配列に追加
  })

  // 3. エッジを作成
  nodes.forEach(node => {
    if (node.parent) {
      edges.push({ source: parent, target: node })
    }
  })

  // 4. 可視性をチェック
  const existingNodes = await checkNodes(nodeTexts)

  // 5. Dagreでレイアウト
  return layoutNodes(nodes, edges)
}
```

### Dagreレイアウト

```typescript
layoutNodes(nodes, edges) {
  // 1. グラフを初期化
  nodes.forEach(node => {
    dagreGraph.setNode(node.id, {
      width: 300,
      height: 120
    })
  })

  // 2. エッジを追加
  edges.forEach(edge => {
    dagreGraph.setEdge(edge.source, edge.target)
  })

  // 3. レイアウトを計算
  dagre.layout(dagreGraph)

  // 4. 計算された位置をノードに適用
  return nodes.map(node => ({
    ...node,
    position: {
      x: dagrePosition.x - width/2,
      y: dagrePosition.y - height/2
    }
  }))
}
```

---

## ナビゲーションシステム

### ナビゲーションの仕組み

目標: 非表示のノードにナビゲートする

**アルゴリズム:**
```
1. 対象ノードから開始
2. ノードが非表示の間:
   a. 親ノードを見つける
   b. 対象が親のchildren配列のどこにあるかを見つける
   c. 現在表示されている兄弟を見つける
   d. 表示されている兄弟から対象への移動ステップを計算
   e. 親に移動して繰り返す
3. ステップを逆順にする（上から下へナビゲートする必要がある）
4. 各ステップを実行:
   a. ノードを見つける
   b. ホバーイベントをトリガー（ボタンを表示）
   c. ナビゲーションボタン（左/右）をクリック
   d. DOM更新を待つ
```

### OpenAI ナビゲーション例

```typescript
// 対象: ノードC（非表示）
// 現在表示: ノードA
// 親の子: [A, B, C]

calculateSteps(nodes, "C") {
  // 親でC（インデックス2）とA（インデックス0）を見つける
  // 2ステップ右に移動する必要がある
  return [
    { nodeId: "A", stepsRight: 1, role: "user" },  // A -> B
    { nodeId: "B", stepsRight: 1, role: "user" }   // B -> C
  ]
}

// これらのステップを実行:
selectBranch(steps) {
  for (step of steps) {
    element = querySelector(`[data-message-id="${step.nodeId}"]`)
    triggerHoverEvents(element)
    rightButton = findRightButton(element)
    rightButton.click()
    waitForDOMChange()
  }
}
```

### Claude ナビゲーション

```typescript
// 類似だがテキストベースのマッチングを使用
calculateStepsClaude(nodes, targetId, lastActiveChildMap) {
  // lastActiveChildMapを使って最後のアクティブな子を追跡
  // これによりページのリロード後も正しいブランチを見つけられる

  while (currentNode?.data?.hidden) {
    activeChildIndex = lastActiveChildMap[parent.id]
      || parent.children.findIndex(visible)

    // 移動が右か左かを計算
    steps.push({
      nodeText: node.data.text,  // ID ではなくテキスト!
      stepsLeft: ...,
      stepsRight: ...
    })
  }
}

selectBranchClaude(steps) {
  for (step of steps) {
    // テキストで要素を見つける
    element = findElementByText(step.nodeText)
    buttons = findButtons(element)
    button = buttons[step.stepsLeft > 0 ? 1 : 2]
    button.click()
    waitForDOMChange()
  }
}
```

---

## エクスポート機能

### サポートされる形式

1. **Markdown** - シンプルなMarkdown形式
2. **Obsidian** - Obsidianノート用のフロントマター付きMarkdown
3. **XML** - 構造化されたXMLフォーマット

### エクスポートの実装

```typescript
// エクスポートパスを構築
const buildExportPath = (nodes, onNodeClick) => {
  // 1. すべてのリーフノード（子なし）を見つける
  const leaves = nodes.filter(n => n.children.length === 0)

  // 2. 各リーフについて:
  leaves.forEach(leaf => {
    // a. リーフにナビゲートするステップを計算
    const steps = onNodeClick(leaf.id)

    // b. ステップを実行してパスを作成
    let path = []
    let current = root
    steps.forEach(step => {
      path.push(current)
      current = navigateToNext(current, step)
    })

    // c. パスを保存
    allPaths.push(path)
  })

  // 3. すべてのパスをエクスポート形式でフォーマット
}

// Markdown形式
exportAsMarkdown() {
  return paths.map(path =>
    path.map(node => `**${node.role}**: ${node.label}`).join('\n\n')
  ).join('\n\n---\n\n')
}

// Obsidian形式
exportAsObsidian() {
  return `---
title: ${conversationData.title}
created: ${new Date().toISOString()}
tags: [conversation, ai]
---

${markdownContent}`
}

// XML形式
exportAsXML() {
  return `<?xml version="1.0"?>
<conversation>
  <metadata>
    <title>${title}</title>
  </metadata>
  <paths>
    ${paths.map(path => `
    <path>
      ${path.map(node => `
      <message role="${node.role}" timestamp="${node.timestamp}">
        ${escapeXML(node.label)}
      </message>
      `).join('')}
    </path>
    `).join('')}
  </paths>
</conversation>`
}
```

---

## ビルドプロセス

### Vite設定 (vite.config.ts)

```typescript
export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      input: {
        main: 'index.html',        // サイドパネルUI
        background: 'src/background.ts'  // バックグラウンドスクリプト
      },
      output: {
        entryFileNames: (chunk) => {
          // background.js は特定の名前が必要（manifest.jsonで参照）
          return chunk.name === 'background'
            ? 'background.js'
            : '[name]-[hash].js'
        }
      }
    }
  }
})
```

### ビルドスクリプト (scripts/build.js)

```javascript
// ビルド後:
// 1. manifest.jsonをdistディレクトリにコピー
fs.copyFileSync('public/manifest.json', 'dist/manifest.json')

// 2. ロゴをコピー
['logo16.png', 'logo48.png', 'logo128.png'].forEach(file => {
  fs.copyFileSync(`public/${file}`, `dist/${file}`)
})
```

### ビルドコマンド

```bash
# 開発
npm run dev          # Vite開発サーバー起動

# 本番ビルド
npm run build        # TypeScriptをコンパイル + Viteビルド + ビルドスクリプト実行

# Linting
npm run lint         # ESLintでコードチェック
```

### ビルド出力

```
dist/
├── index.html              # サイドパネルUI
├── assets/
│   ├── index-[hash].js    # バンドルされたReactアプリ
│   ├── index-[hash].css   # スタイル
│   └── ...
├── background.js          # バックグラウンドサービスワーカー
├── manifest.json          # 拡張機能マニフェスト
├── logo16.png
├── logo48.png
└── logo128.png
```

---

## 主要なアルゴリズムとパターン

### 1. 再帰的ツリートラバーサル

OpenAIノード作成で使用:
```typescript
const createChildNodes = (node: OpenAINode) => {
  if (node.children.length === 0) return

  const validChildren = node.children
    .map(childId => findFirstValidDescendant(mapping[childId]))
    .filter(child => child !== null)

  validChildren.forEach(child => {
    child.parent = node.id
    newNodes.push(child)
    createChildNodes(child)  // 再帰
  })
}
```

### 2. BFS/DFSパス検索

ナビゲーションパスを見つける:
```typescript
// 対象から上に向かってトラバース（実質的にDFS）
while (currentNode?.data?.hidden) {
  // 親を見つける
  // ナビゲーションステップを計算
  // 親に移動
}
```

### 3. オブザーバーパターン

DOM変更の監視:
```typescript
const waitForDomChange = () => {
  return new Promise((resolve) => {
    const observer = new MutationObserver((mutations) => {
      if (mutations.length > 0) {
        observer.disconnect()
        resolve()
      }
    })
    observer.observe(element, {
      childList: true,
      subtree: true
    })
  })
}
```

### 4. キャッシング/メモ化

最後のアクティブな子を追跡:
```typescript
const [lastActiveChildMap, setLastActiveChildMap] = useState<Record<string, string>>({})

// ノードがクリックされたとき
setLastActiveChildMap(prev => ({
  ...prev,
  [parentId]: clickedNodeId
}))

// ナビゲーション中に使用
const cachedActiveChildId = lastActiveChildMap[parent.id]
```

### 5. イベント駆動アーキテクチャ

Chrome拡張機能のメッセージング:
```typescript
// UIがイベントを送信
chrome.runtime.sendMessage({ action: "editMessage", ... })

// バックグラウンドがイベントを処理
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  // リクエストを処理
  // 結果を返す
})
```

---

## セキュリティとプライバシー

### データ処理
- **ローカルのみ**: すべての処理はブラウザ内で行われる
- **外部サーバーなし**: データは外部サービスに送信されない
- **セッションストレージ**: ヘッダーはセッションに一時的に保存される
- **cookieのみ**: Claude APIはブラウザのクッキーを使用

### 権限の使用
```typescript
// ストレージ - ヘッダーとorgIdの保存
chrome.storage.session.set({ storedRequestHeaders: headers })

// webRequest - 認証ヘッダーのキャプチャ
chrome.webRequest.onBeforeSendHeaders.addListener(...)

// スクリプティング - DOM操作
chrome.scripting.executeScript({
  target: { tabId },
  func: () => { /* DOMコード */ }
})
```

### ホスト権限
制限されたAPI エンドポイントのみ:
```json
{
  "host_permissions": [
    "https://chatgpt.com/backend-api/*",
    "https://claude.ai/api/organizations/*"
  ]
}
```

---

## トラブルシューティングとデバッグ

### よくある問題

1. **ボタンが見つからない**
   - 原因: ボタンがレンダリングされていない
   - 解決策: `triggerNativeArticleEvents()` がホバーイベントをトリガー

2. **認証ヘッダーが見つからない**
   - 原因: ページが完全にロードされる前にフェッチ
   - 解決策: 3回の再試行メカニズム（500msの遅延）

3. **Claudeノードが見つからない**
   - 原因: テキストの正規化の不一致
   - 解決策: HTMLエンティティ、Markdown、アーティファクトタグを処理

### デバッグツール

```typescript
// ログをバックグラウンドコンソールに送信
chrome.runtime.sendMessage({
  action: "log",
  message: "Debug info here"
})

// ノードの可視性をチェック
const existingNodes = await checkNodes(nodeIds)
console.log('Hidden nodes:', existingNodes)

// ナビゲーションステップを検査
const steps = calculateSteps(nodes, targetId)
console.log('Navigation steps:', steps)
```

---

## 今後の改善可能性

### パフォーマンス
- [ ] 大きなグラフの仮想化
- [ ] ノード作成のメモ化
- [ ] 遅延ロード/インクリメンタルレンダリング

### 機能
- [ ] グラフの検索とフィルタリング
- [ ] カスタムノードのスタイリング
- [ ] 会話のブックマーク/お気に入り
- [ ] 会話のマージ/比較
- [ ] テーマサポート（ダーク/ライトモード）

### 技術的負債
- [ ] エラー境界とエラー処理の改善
- [ ] ユーザビリティのためのローディング状態の改善
- [ ] より堅牢なDOM セレクター
- [ ] ユニットテストとE2Eテスト
- [ ] より良いTypeScriptの型カバレッジ

---

## 結論

ChatTreeは、洗練されたツリー可視化とナビゲーションシステムを備えた、よく設計されたChrome拡張機能です。

### 主な長所:
- ✅ クリーンな分離されたアーキテクチャ
- ✅ 包括的なTypeScriptの型定義
- ✅ OpenAIとClaudeの両方をサポート
- ✅ ローカル処理によるプライバシー優先
- ✅ 堅牢なDOM操作処理

### 主な設計パターン:
- フックによるReact状態管理
- Chrome拡張機能のメッセージパッシング
- 非同期/待機パターンのDOM操作
- 有向グラフのトラバーサルアルゴリズム
- イベント駆動アーキテクチャ

このドキュメントはコードベースの完全なリバースエンジニアリングを提供し、アーキテクチャ、データフロー、主要アルゴリズム、および実装の詳細をカバーしています。
