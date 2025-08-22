# 6. カバレッジ

## 📝 概要

テストカバレッジとは、プログラムコードがテストによってどの程度カバー（実行）されているかを示す指標です。Vitestでは簡単にカバレッジレポートを生成でき、コードの品質向上に役立てることができます。

このセクションでは、以下を学びます：

- カバレッジレポートの設定方法
- カバレッジレポートの実行方法
- レポートの見方と分析方法
- カバレッジ目標の設定方法

## ⚙️ カバレッジ設定

### 1. 必要なパッケージのインストール

Vitestでカバレッジレポートを生成するには、`v8`（V8エンジンのネイティブコードカバレッジ）または`istanbul`を使用します。今回は`v8`を使用します。

```bash
# プロジェクトのルートディレクトリで実行
cd my-nuxt-app
pnpm add -D @vitest/coverage-v8
```

### 2. vitest.config.tsの設定

カバレッジの設定をするために、`vitest.config.ts`ファイルを編集します。

```typescript
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true,
    environment: 'jsdom',
    coverage: {
      provider: 'v8', // カバレッジプロバイダの指定（デフォルトはv8）
      reporter: ['text', 'json', 'html'], // レポート形式を指定
      exclude: [
        'node_modules/**',
        '.nuxt/',
        '**/*.config.{js,ts}',
        'dist/**',
        '**/*.d.ts',
        '**/*.test.ts',
        '**/*.spec.ts',
        'test/**',
        'tests/**',
      ],
    },
  },
})
```

主な設定項目：

- `provider`: カバレッジを計測するプロバイダ（'c8'または'istanbul'）
- `reporter`: レポート形式（'text', 'json', 'html', 'lcov'など）
  - `text`: ターミナルに表示されるテキスト形式
  - `html`: HTMLファイルとして保存（視覚的に確認しやすい）
  - `json`: JSONファイルとして保存
  - `lcov`: LCOVフォーマットで保存（多くのカバレッジツールと互換性あり）
- `exclude`: カバレッジ計測から除外するファイルパターン

### 3. package.jsonにスクリプトを追加

テストカバレッジを実行しやすくするために、`package.json`のスクリプトセクションに以下を追加します：

```json
{
  "scripts": {
    // 既存のスクリプト
    "test:coverage": "vitest run --coverage"
  }
}
```

## 🚀 カバレッジレポートの実行

以下のコマンドを実行して、テストカバレッジを計測します：

```bash
pnpm test:coverage
```

コマンドを実行すると、以下のようなテキスト形式のレポートがターミナルに表示されます：

```text
 ```

 % Coverage report from v8
-----------------------|---------|----------|---------|---------|-------------------

File                   | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
-----------------------|---------|----------|---------|---------|-------------------
All files              |   85.71 |      100 |   66.67 |   85.71 |
 utils                 |   85.71 |      100 |   66.67 |   85.71 |
  calculator.ts        |     100 |      100 |     100 |     100 |
  sum.ts               |      50 |      100 |       0 |      50 | 3
-----------------------|---------|----------|---------|---------|-------------------

```
```

また、HTMLレポートはプロジェクトの`coverage`ディレクトリに生成されます。ブラウザで`coverage/index.html`を開くと、詳細なカバレッジレポートを確認できます。

## 📊 レポートの見方と分析方法

カバレッジレポートには主に以下の指標が含まれます：

### 1. 基本的な指標

- **Statements（ステートメント）**: コード内の各ステートメント（命令文）がテストされた割合
- **Branches（ブランチ）**: if文などの条件分岐がテストされた割合
- **Functions（関数）**: 関数がテストで呼び出された割合
- **Lines（行）**: コード行がテストで実行された割合

### 2. HTMLレポートの読み方

HTMLレポートを開くと、ファイルごとのカバレッジ状況が視覚的に表示されます：

1. **ディレクトリ/ファイル一覧**: プロジェクト全体のカバレッジ構造
2. **色分け**:
   - **緑**: テストでカバーされている行
   - **赤**: テストでカバーされていない行
   - **黄**: 一部だけカバーされている行（条件分岐の一部など）
3. **詳細表示**: ファイル名をクリックすると、そのファイル内のどの行がテストされているか詳細に確認できる

例えば、`utils/sum.ts`をクリックすると、以下のように行ごとのカバレッジ状況が表示されます：

```typescript
// カバレッジ結果の例
export function sum(a: number, b: number): number {
  return a + b; // この行はテストされている（緑色）
}

export function subtract(a: number, b: number): number {
  return a - b; // この行はテストされていない（赤色）
}
```

## 🎯 カバレッジ目標の設定方法

チームやプロジェクトでカバレッジ目標を設定することで、コードの品質を一定水準以上に保つことができます。

### 1. カバレッジしきい値の設定

`vitest.config.ts`にしきい値を設定できます：

```typescript
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true,
    environment: 'jsdom',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/**',
        'dist/**',
        '**/*.d.ts',
        '**/*.test.ts',
        '**/*.spec.ts',
        'test/**',
        'tests/**',
      ],
      // しきい値の設定
      thresholds: {
        statements: 80,
        branches: 80,
        functions: 80,
        lines: 80,
      },
    },
  },
})
```

この設定では、各指標が80%未満の場合、テストが失敗となります。

### 2. 効果的なカバレッジ向上の戦略

カバレッジを効果的に向上させるためのポイント：

1. **未テストの部分から優先的に**: カバレッジレポートで赤く表示されている部分から優先的にテストを追加
2. **複雑なロジックを重視**: 条件分岐が多い複雑なロジックは優先的にテスト
3. **重要なビジネスロジックに注目**: システムの中核となる機能は高いカバレッジを目指す
4. **段階的に向上**: 一度に100%を目指すのではなく、段階的に向上させる戦略も有効

## 📝 実習: カバレッジレポートの作成と分析

### 実習1: カバレッジレポートの作成

1. 必要なパッケージをインストールします：

   ```bash
   pnpm add -D @vitest/coverage-v8
   ```

2. `vitest.config.ts`を編集してカバレッジ設定を追加します（上記の設定例参照）

3. カバレッジレポートを実行します：

   ```bash
   pnpm test:coverage
   ```

4. ターミナルに表示されるテキストレポートを確認します

5. ブラウザで`coverage/index.html`を開いて詳細なレポートを確認します

### 実習2: カバレッジ改善

1. カバレッジの低いファイルを特定します

2. たとえば、`utils/date-utils.spec.ts`のカバレッジが低い場合、テストケースを追加してカバレッジを向上させます：

   ```typescript
   // utils/date-utils.spec.ts
   import { describe, it, expect } from 'vitest'
   import { formatDate, isWeekend } from './date-utils'

   describe('date-utils', () => {
     // 既存のテスト

     // カバレッジ向上のための追加テスト
     it('should correctly identify weekend days', () => {
       // 土曜日 (2023-06-03)
       expect(isWeekend(new Date(2023, 5, 3))).toBe(true)

       // 日曜日 (2023-06-04)
       expect(isWeekend(new Date(2023, 5, 4))).toBe(true)

       // 平日 (2023-06-05 月曜日)
       expect(isWeekend(new Date(2023, 5, 5))).toBe(false)
     })
   })
   ```

3. テストを再実行して、カバレッジが向上したことを確認します：

   ```bash
   pnpm test:coverage
   ```

## ⭐ チャレンジ課題

1. プロジェクト内の全ファイルのうち、カバレッジが最も低いファイルを特定し、カバレッジを80%以上に向上させましょう

2. 現在のカバレッジ状況を元に、チーム向けのカバレッジ改善計画を立ててみましょう
   - どのファイルを優先的に改善すべきか
   - どのようなテスト戦略で臨むか
   - 目標とするカバレッジ値はいくつか

## 🔍 まとめ

このセクションでは、Vitestを使用したカバレッジレポートの設定方法、実行方法、分析方法について学びました。

- カバレッジレポートは、コードの品質を定量的に測定する重要なツール
- Vitestでは簡単にカバレッジレポートを生成可能
- カバレッジ指標には、ステートメント、ブランチ、関数、行の4つの基本指標がある
- カバレッジ目標を設定することで、コード品質を一定水準以上に保つことが可能
- カバレッジ100%を目指すのではなく、重要な部分に焦点を当てるバランスが大切

カバレッジレポートを活用して、コードの信頼性を高め、持続可能な開発を進めていきましょう。

## 📚 参考資料

- [Vitest公式ドキュメント - カバレッジ](https://vitest.dev/guide/coverage.html)
- [V8カバレッジについて](https://v8.dev/blog/javascript-code-coverage)
- [コードカバレッジの考え方](https://martinfowler.com/bliki/TestCoverage.html)
