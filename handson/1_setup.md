# 1. セットアップ

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
    globals: true,        // テスト用の関数を都度インポートしなくても使えるように
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

シンプルなテストを作成します

1. `utils/sum.ts`を作成し、以下コピー＆ペースト

    ```ts
    export function sum(a: number, b: number): number {
      return a + b;
    }
    ```

2. `utils/sum.spec.ts` を作成し、以下コピー＆ペースト

    ```ts
    import { sum } from '@/utils/sum'; // テスト対象をインポートする
    // import { expect, test } from 'vitest'; // vitest.config.ts で globals: false の場合はこの記述が必要になる

    test('足し算のテスト', () => {
      expect(sum(1, 2)).toBe(3); // 1 + 2 が 3 になることを確認
    });
    ```

## テストを実行する

以下のコマンドでテストを実行します

```sh
pnpm vitest
```

テストが成功していることを確認します

```sh
 DEV  v3.2.4 ~/practice_vitest/my-nuxt-app

 ✓ utils/sum.spec.ts (1 test) 4ms
   ✓ 足し算のテスト 2ms

 Test Files  1 passed (1)
      Tests  1 passed (1)
   Start at  16:43:08
   Duration  2.33s (transform 68ms, setup 0ms, collect 57ms, tests 4ms, environment 1.28s, prepare 222ms)


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

これで、`pnpm test` コマンドでテストを実行できるようになります

---

以上でセットアップは完了です

[2. 基本的なテスト](./2_basic_test.md) に進みましょう
