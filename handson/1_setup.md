# 1. 基本セットアップ

このセクションでは、NuxtプロジェクトにVitestを導入する基本的な手順を説明します

## プロジェクトの作成

```sh
npm create nuxt@latest my-nuxt-app -t v3
```

質問回答は以下

```sh
❯ Which package manager would you like to use?
● pnpm

❯ Initialize git repository?
● No

❯ Would you like to install any of the official modules?
● No
```

プロジェクトを作成したら、そのディレクトリに移動します

```sh
cd my-nuxt-app

# 依存パッケージのビルドを許可
pnpm approve-builds


? Choose which packages to build (Press <space> to select, <a> to toggle all, <i> to invert selection) …
  ● @parcel/watcher
❯ ● esbuild

✔ The next packages will now be built: @parcel/watcher, esbuild.
Do you approve? (y/N) y
```

## Vitestインストール

必要なパッケージをインストールします

```sh
pnpm add -D vitest @vitejs/plugin-vue@latest jsdom
```

## Vitestの設定

Nuxtプロジェクト用に、プロジェクトルートに`vitest.config.ts`ファイルを新規作成し、Vitestの設定を追加します

```ts
import vue from '@vitejs/plugin-vue'
import { fileURLToPath } from 'node:url'
import { defineConfig } from 'vitest/config'

export default defineConfig({
  plugins: [vue()],
  test: {
    // Vitestの基本設定
    globals: true,        // describe, it, expect を都度インポートしなくても使えるように
    environment: 'jsdom', // DOMテスト用の環境
  },
  resolve: {
    alias: {
      '~': fileURLToPath(new URL('./', import.meta.url)),
      '@': fileURLToPath(new URL('./', import.meta.url)),
    }
  }
})
```

設定項目を詳しく知りたい場合は[公式リファレンス](https://vitest.dev/config/)を参照してください

## 最初のテストを作成

シンプルなテストファイルを作成します

1. `tests`ディレクトリを作成
2. その中に `example.spec.ts` ファイルを作成
3. 以下内容を入力

```ts
import { describe, it, expect } from 'vitest'

describe('最初のテスト', () => {
  it('trueはtrueであること', () => {
    expect(true).toBe(true)
  })

  it('1 + 1は2になること', () => {
    expect(1 + 1).toBe(2)
  })
})
```

テストコードについては後ほど解説します

## テストを実行する

以下のコマンドでテストを実行します

```sh
pnpm vitest
```

テストが成功していることを確認します

```sh
 DEV  v3.2.4 ~/practice_vitest/my-nuxt-app

 ✓ tests/example.spec.ts (2 tests) 3ms
   ✓ 最初のテスト > trueはtrueであること 1ms
   ✓ 最初のテスト > 1 + 1は2になること 0ms

 Test Files  1 passed (1)
      Tests  2 passed (2)
   Start at  14:31:55
   Duration  2.16s (transform 87ms, setup 0ms, collect 38ms, tests 3ms, environment 1.09s, prepare 299ms)

 PASS  Waiting for file changes...
       press h to show help, press q to quit
```

`q` で終了します

## package.jsonにテストスクリプトを追加

`package.json`ファイルを開いて、テスト用のスクリプトを追加します

```json
{
  "scripts": {
    "build": "nuxt build",
    "dev": "nuxt dev",
    "generate": "nuxt generate",
    "preview": "nuxt preview",
    "postinstall": "nuxt prepare",
    // ↓この行を追加
    "test": "vitest"
  }
}
```

これで、`pnpm test`コマンドでテストを実行できるようになります

---

以上で基本的なセットアップは完了です

次のセクションでは、Vitestを使った基本的なテスト方法について学びます
