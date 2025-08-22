# 5. スナップショットテスト

## 👀 スナップショットテストとは

スナップショットテストは、UIコンポーネントの出力（レンダリング結果）をキャプチャし、将来の変更に対して比較するテスト手法です。主に以下のような目的で使用します：

- UIコンポーネントの意図しない変更を検出する
- コンポーネントの表示が一貫していることを確認する
- リグレッション（機能後退）を防ぐ

## 🌟 スナップショットテストの仕組み

1. 初回実行時：テスト対象のコンポーネントをレンダリングし、その結果を「スナップショット」としてファイルに保存
2. 2回目以降：新しいレンダリング結果と保存されたスナップショットを比較
3. 差異がある場合：テストが失敗し、意図的な変更であれば更新が必要

## 🔍 基本的なスナップショットテスト

まずは簡単なコンポーネントに対してスナップショットテストを作成してみましょう。

### 1. シンプルなコンポーネントの作成

まずはテスト用の簡単なコンポーネント `Greeting.vue` を作成します：

```bash
touch my-nuxt-app/components/Greeting.vue
```

`Greeting.vue` に以下のコードを追加します：

```vue
<template>
  <div class="greeting">
    <h1>{{ title }}</h1>
    <p>{{ message }}</p>
  </div>
</template>

<script setup>
defineProps({
  title: {
    type: String,
    default: 'こんにちは'
  },
  message: {
    type: String,
    default: 'Vitestへようこそ！'
  }
})
</script>

<style scoped>
.greeting {
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}
h1 {
  color: #41b883;
}
</style>
```

### 2. スナップショットテストの作成

次に、このコンポーネントのスナップショットテスト `Greeting.spec.ts` を作成します：

```bash
touch my-nuxt-app/components/Greeting.spec.ts
```

`Greeting.spec.ts` に以下のコードを追加します：

```typescript
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import Greeting from './Greeting.vue'

describe('Greeting', () => {
  it('デフォルト表示が正しいこと', () => {
    const wrapper = mount(Greeting)
    expect(wrapper.html()).toMatchSnapshot()
  })

  it('プロパティを渡した場合に表示が変わること', () => {
    const wrapper = mount(Greeting, {
      props: {
        title: 'こんばんは',
        message: 'スナップショットテストを学びましょう！'
      }
    })
    expect(wrapper.html()).toMatchSnapshot()
  })
})
```

### 3. スナップショットテストの実行

テストを実行します：

```bash
cd my-nuxt-app
pnpm vitest run components/Greeting.spec.ts
```

初回実行時、Vitestはスナップショットを生成し保存します。以下のような出力が表示されるでしょう：

```sh
 ✓ components/Greeting.spec.ts (2 tests) 34ms
   ✓ Greeting > デフォルト表示が正しいこと 28ms
   ✓ Greeting > プロパティを渡した場合に表示が変わること 4ms

  Snapshots  2 written
 Test Files  1 passed (1)
      Tests  2 passed (2)
```

### 4. スナップショットファイルの確認

Vitestは自動的に `__snapshots__` ディレクトリを作成し、その中にスナップショットファイルを保存します。

```bash
ls components/__snapshots__
# Greeting.spec.ts.snap が表示されるはずです
```

スナップショットファイルを開いて中身を確認してみましょう：

```javascript
// components/__snapshots__/Greeting.spec.ts.snap
exports[`Greeting > デフォルト表示が正しいこと 1`] = `
"<div class=\\"greeting\\">
  <h1>こんにちは</h1>
  <p>Vitestへようこそ！</p>
</div>"
`;

exports[`Greeting > プロパティを渡した場合に表示が変わること 1`] = `
"<div class=\\"greeting\\">
  <h1>こんばんは</h1>
  <p>スナップショットテストを学びましょう！</p>
</div>"
`;
```

## 🔄 スナップショットの更新

コンポーネントを意図的に変更した場合、スナップショットも更新する必要があります。

### 1. コンポーネントの変更

`Greeting.vue` を変更してみましょう：

```vue
<template>
  <div class="greeting">
    <h1>{{ title }}</h1>
    <p>{{ message }}</p>
    <!-- 新しい要素を追加 -->
    <small>Powered by Vitest</small>
  </div>
</template>

<!-- 残りのコードは変更なし -->
```

### 2. テストの実行

テストを再実行すると、スナップショットの差異により失敗します：

```bash
cd my-nuxt-app
pnpm vitest run components/Greeting.spec.ts
```

差異の詳細が表示されます：

```diff
Error: Snapshot `Greeting > プロパティを渡した場合に表示が変わること 1` mismatched

- Expected
+ Received

  "<div data-v-259a7336="" class="greeting">
    <h1 data-v-259a7336="">こんばんは</h1>
-   <p data-v-259a7336="">スナップショットテストを学びましょう！</p>
+   <p data-v-259a7336="">スナップショットテストを学びましょう！</p><!-- 新しい要素を追加 --><small data-v-259a7336="">Powered by Vitest</small>
  </div>"
```

### 3. スナップショットの更新

この変更が意図的なものである場合、スナップショットを更新します：

```bash
cd my-nuxt-app
pnpm vitest run components/Greeting.spec.ts -u
```

`-u` または `--update` フラグを使うことで、スナップショットが更新されます。

## 📸 インラインスナップショット

別ファイルではなく、テストファイル内に直接スナップショットを保存することもできます。

### インラインスナップショットの例

`Greeting.inline.spec.ts` という新しいテストファイルを作成しましょう：

```bash
touch my-nuxt-app/components/Greeting.inline.spec.ts
```

`Greeting.inline.spec.ts` に以下のコードを追加します：

```typescript
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import Greeting from './Greeting.vue'

describe('Greeting (インラインスナップショット)', () => {
  it('デフォルト表示が正しいこと', () => {
    const wrapper = mount(Greeting)
    expect(wrapper.html()).toMatchInlineSnapshot()
  })

  it('プロパティを渡した場合に表示が変わること', () => {
    const wrapper = mount(Greeting, {
      props: {
        title: 'おはよう',
        message: 'インラインスナップショットのデモです'
      }
    })
    expect(wrapper.html()).toMatchInlineSnapshot()
  })
})
```

### インラインスナップショットの実行と更新

テストを実行すると、Vitestはスナップショットをインラインで追加します：

```bash
cd my-nuxt-app
pnpm vitest run components/Greeting.inline.spec.ts
```

テストファイルが自動的に更新され、以下のようになります：

```typescript
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import Greeting from './Greeting.vue'

describe('Greeting (インラインスナップショット)', () => {
  it('デフォルト表示が正しいこと', () => {
    const wrapper = mount(Greeting)
    expect(wrapper.html()).toMatchInlineSnapshot(`
      "<div class=\\"greeting\\">
        <h1>こんにちは</h1>
        <p>Vitestへようこそ！</p>
        <small>Powered by Vitest</small>
      </div>"
    `)
  })

  it('プロパティを渡した場合に表示が変わること', () => {
    const wrapper = mount(Greeting, {
      props: {
        title: 'おはよう',
        message: 'インラインスナップショットのデモです'
      }
    })
    expect(wrapper.html()).toMatchInlineSnapshot(`
      "<div class=\\"greeting\\">
        <h1>おはよう</h1>
        <p>インラインスナップショットのデモです</p>
        <small>Powered by Vitest</small>
      </div>"
    `)
  })
})
```

## 🧪 部分的なスナップショットテスト

時にはコンポーネント全体ではなく、特定の部分だけをスナップショットとして保存したい場合があります。

### 部分スナップショットの例

`Greeting.partial.spec.ts` という新しいテストファイルを作成しましょう：

```bash
touch my-nuxt-app/components/Greeting.partial.spec.ts
```

`Greeting.partial.spec.ts` に以下のコードを追加します：

```typescript
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import Greeting from './Greeting.vue'

describe('Greeting (部分スナップショット)', () => {
  it('タイトル部分のみをスナップショットとして保存', () => {
    const wrapper = mount(Greeting)
    // h1要素の内容だけをスナップショット
    expect(wrapper.find('h1').html()).toMatchSnapshot()
  })

  it('メッセージ部分のみをスナップショットとして保存', () => {
    const wrapper = mount(Greeting, {
      props: {
        message: '部分スナップショットのテストです'
      }
    })
    // p要素の内容だけをスナップショット
    expect(wrapper.find('p').html()).toMatchSnapshot()
  })
})
```

同様に実行すると、部分的なスナップショットが生成されます。

## 🔧 スナップショットテストのベストプラクティス

1. **変更の頻度が低い要素に使用する**
   - 頻繁に変更される部分には使わない（メンテナンスコストが高くなる）

2. **適切な粒度を選ぶ**
   - 大きすぎるコンポーネントのスナップショットは読みにくく、メンテナンスも難しい
   - 必要に応じて部分スナップショットを活用する

3. **動的なデータには注意**
   - 日付や乱数など、実行ごとに変わる値はモックするか、スナップショットから除外する

4. **スナップショットの更新は慎重に**
   - `-u` フラグで更新する前に、差分を確認し意図した変更かどうかを確認する

5. **テストの説明を明確に**
   - スナップショットが何を検証しているのか、テスト名で明確にする

## 🚨 スナップショットテストの注意点

1. **スタイルの変更も検知される**
   - CSSの変更もHTMLに影響するため、純粋な構造だけをテストしたい場合は注意

2. **大きすぎるスナップショット**
   - 巨大なスナップショットは可読性が低下し、メンテナンスが困難になる

3. **偽陽性と偽陰性**
   - 小さな変更でも失敗するため、本当に重要な変更を見逃す可能性がある

4. **スナップショットに依存しすぎない**
   - 他のタイプのテスト（ユニットテスト、インタラクションテスト）と組み合わせる

## 📝 練習問題

1. 以下の要件を満たす `UserCard.vue` コンポーネントを作成し、スナップショットテストを書いてみましょう：
   - ユーザーの名前、メール、役職を表示
   - プロパティとしてユーザー情報を受け取る
   - デフォルト値も設定する

2. 作成したコンポーネントに新しい要素（例：アバター画像）を追加し、スナップショットを更新する練習をしてみましょう。

## 🎯 まとめ

スナップショットテストは、UIコンポーネントの変更を効率的に検出するための強力なツールです。適切に使用することで、以下のメリットが得られます：

- UI変更の視覚的な確認が容易になる
- 意図しない変更を早期に発見できる
- コンポーネントの動作を文書化できる

一方で、メンテナンスコストや偽陽性の問題もあるため、他のテスト手法と組み合わせて使用することが重要です。

## 📚 参考リソース

- [Vitest スナップショット公式ドキュメント](https://vitest.dev/guide/snapshot.html)
- [Vue Test Utils を使ったテスト](https://test-utils.vuejs.org/)
