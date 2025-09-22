# SPEC.md — メモ帳 Web アプリ（Prototype → Promote想定）

## 1. 目的 / スコープ
- ライトなメモを **素早く追加・編集・削除・一覧** できる最小機能のメモ帳。
- **オフライン動作**（localStorage）。アカウント/サーバー不要。
- プロトからテンプレリポへ「昇格」し、CI/Codex 自動修復を回しやすい構成。

## 2. 推奨スタック（実装の自由は残す）
- Next.js + TypeScript（App Router 可）／もしくは Vite + React + TypeScript
- 状態管理: React Hooks（外部ステート管理なし）
- スタイル: Tailwind（任意）
- 保存: **localStorage のみ**
- テスト: **Vitest + React Testing Library（最小でも1件）**
- package manager: **pnpm**（CIの scripts 名は固定：`dev/build/lint/typecheck/test`）

## 3. 機能要件（FRD）
### 3.1 メモ CRUD
- **追加**：タイトル（任意）+ 本文（必須）。保存直後に一覧へ反映。
- **編集**：既存メモの本文/タイトルを上書き保存。保存後は一覧へ戻り、該当行が更新されていること。
- **削除**：一覧から削除。**確認ダイアログ**を挟む（「削除／キャンセル」）。
- **一覧**：更新日時の降順（直近が上）。タイトルが空のときは本文冒頭をプレビュー。
- **検索/フィルタ（任意）**：入力中にクライアント側で前方一致 or 包含検索。

### 3.2 入力バリデーション
- 本文が空の保存は不可（トースト or インラインで「本文を入力してください」）。
- 入力文字数：本文 10,000 文字、タイトル 120 文字まで（超過時は警告して保存不可）。
- 連打対策：保存ボタンは処理中は disabled。

### 3.3 永続化（localStorage）
- キー：`"ai-notes:v1:notes"`
- 形式：JSON 文字列（下記データモデル）
- 保存/読み込みは専用モジュール `src/lib/storage.ts` に集約（テスト容易化のため）。

### 3.4 UX / 画面
- **一覧ページ**：
  - 追加ボタン（右下 FAB か ヘッダー）。
  - 各行：タイトル（なければ冒頭30字）/ 更新日時 / 「編集」「削除」。
  - 空状態：イラスト不要、テキスト「まだメモがありません。＋ボタンから追加」。
- **編集ページ**：
  - タイトル input（任意）/ 本文 textarea（必須）/ 保存ボタン。
  - 破棄ナビゲーション時は「未保存の変更があります」確認（任意）。

## 4. 非機能要件（NFR）
- パフォーマンス：初回ロード < 2s、操作は 100ms 目安で反応。
- アクセシビリティ：キーボード操作可能、ラベル/aria属性、コントラスト比を確保。
- i18n：文言は定数化（後で多言語化できる構造）。
- セキュリティ：localStorage のみ利用し、機密情報は扱わない。XSS: 入力はそのまま表示せず、テキストとして安全に描画（危険な HTML を挿入しない）。
- 互換：モダンブラウザ（最新2メジャー）。モバイル幅 360px〜 で崩れない。

## 5. データモデル
```ts
export type NoteId = string; // uuid
export type ISODate = string; // new Date().toISOString()

export interface Note {
  id: NoteId;
  title?: string;      // 0..120 chars
  body: string;        // 1..10_000 chars
  createdAt: ISODate;
  updatedAt: ISODate;
}

export interface NotesState {
  notes: Note[];
  // 予備: version: 1
}
