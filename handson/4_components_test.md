# 4. コンポーネントテスト

このセクションでは、Vitestを使用してVueコンポーネントのテストを行う方法を学びます。UIコンポーネントは、アプリケーションの重要な部分であり、適切なテストが必要です。

## コンポーネントテストとは

コンポーネントテストとは、Vueコンポーネントが期待通りに動作することを確認するためのテストです。具体的には以下の点を検証します：

1. **レンダリング**: コンポーネントが正しくレンダリングされるか
2. **プロパティ**: 渡したpropsが正しく処理されるか
3. **イベント**: イベントが正しく発火するか
4. **ユーザー操作**: クリックなどのユーザー操作に正しく反応するか
5. **状態変化**: コンポーネントの内部状態が正しく変化するか

## テスト環境のセットアップ

Vueコンポーネントをテストするには、ブラウザのような環境が必要です。今回は `happy-dom` を使用して実現しています。

既に `vitest.config.ts` ファイルで環境が設定されています：

```ts
test: {
  globals: true,
  environment: 'happy-dom', // DOMテスト用の環境
},
```

## 最初のコンポーネントを作成する

まずは、簡単なカウンターコンポーネントを作成しましょう。

`components/Counter.vue` ファイルを作成します：

```vue
<template>
  <div class="counter">
    <p>現在のカウント: {{ count }}</p>
    <button @click="increment">増やす</button>
    <button @click="decrement">減らす</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// カウント状態を管理
const count = ref(0)

// メソッド定義
const increment = () => count.value++
const decrement = () => count.value--
</script>
```

## テスト用のマウントヘルパー

### マウントとは

Vueコンポーネントのテストにおける「マウント」とは、テスト環境内でコンポーネントをインスタンス化して仮想DOM上にレンダリングするプロセスを指します。実際のブラウザではなく、テストの中で仮想的にコンポーネントを「取り付ける」操作です。

マウントには以下の重要な意味があります：

1. **コンポーネントの初期化**: Vueインスタンスを作成し、ライフサイクルフックを実行します
2. **レンダリング**: テンプレートをJSDOM環境（仮想DOM）上にレンダリングします
3. **状態の設定**: propsやdata、Vuexストアなど、必要な状態を設定します
4. **テスト用インターフェースの提供**: レンダリングされたコンポーネントを操作・検証するためのラッパーオブジェクトを返します

マウントすることで、テキスト内容の確認、クラスの検証、イベントのトリガー、子要素の検索など、さまざまな操作やアサーションが可能になります。実際のDOMと同様の方法でコンポーネントを検査できるため、ブラウザ環境での動作を模擬したテストが可能になります。

### マウントヘルパー関数の作成

Vueコンポーネントをテストするために、カスタムのマウントヘルパー関数を作成しましょう。これにより、テスト間で一貫した設定でコンポーネントをマウントできます。

`tests/helpers/mount.ts` ファイルを作成します：

```ts
import { mount as vueTestUtilsMount } from '@vue/test-utils'

// マウントヘルパー関数
export function mount(component, options = {}) {
  return vueTestUtilsMount(component, {
    ...options,
    global: {
      ...options.global,
    }
  })
}
```

このヘルパー関数は、Vue Test Utilsが提供する標準の `mount`関数をラップしています。将来的にプラグインやグローバルコンポーネント、モックなどの共通設定を追加する際に、この関数を拡張するだけで済むようになります。

この関数を使うために、`@vue/test-utils` パッケージをインストールしましょう：

```sh
pnpm add -D @vue/test-utils
```

## 基本的なコンポーネントテスト

`components/Counter.spec.ts` ファイルを作成して、カウンターコンポーネントをテストします：

```ts
import { mount } from '@vue/test-utils'
import Counter from '@/components/Counter.vue'

describe('Counter.vue', () => {
  it('初期状態のレンダリングが正しいこと', () => {
    // コンポーネントをマウント
    const wrapper = mount(Counter)

    // テキストコンテンツの検証
    expect(wrapper.text()).toContain('現在のカウント: 0')

    // 要素の存在を検証
    expect(wrapper.find('button').exists()).toBe(true)
    expect(wrapper.findAll('button').length).toBe(2)
  })

  it('増やすボタンをクリックするとカウントが増加すること', async () => {
    const wrapper = mount(Counter)

    // 初期値の確認
    expect(wrapper.text()).toContain('現在のカウント: 0')

    // 増やすボタンをクリック
    await wrapper.findAll('button')[0].trigger('click')

    // カウントが増加したことを確認
    expect(wrapper.text()).toContain('現在のカウント: 1')
  })

  it('減らすボタンをクリックするとカウントが減少すること', async () => {
    const wrapper = mount(Counter)

    // 初期値の確認
    expect(wrapper.text()).toContain('現在のカウント: 0')

    // 減らすボタンをクリック
    await wrapper.findAll('button')[1].trigger('click')

    // カウントが減少したことを確認
    expect(wrapper.text()).toContain('現在のカウント: -1')
  })
})
```

## 要素の選択とアサーション

Vue Test Utilsは、コンポーネントの要素を選択するための様々なメソッドを提供しています：

- `find(selector)`: CSSセレクタ、コンポーネント、またはオブジェクトに一致する最初の要素を見つける
- `findAll(selector)`: セレクタに一致するすべての要素の配列を返す
- `findComponent(component)`: 子コンポーネントを見つける

よく使うアサーション：

- `wrapper.text()`: テキストコンテンツ全体を取得
- `wrapper.html()`: HTML全体を取得
- `wrapper.classes()`: 要素のクラス一覧を取得
- `wrapper.attributes()`: 要素の属性を取得
- `wrapper.exists()`: 要素が存在するかどうかを確認
- `wrapper.isVisible()`: 要素が表示されているかどうかを確認

## ユーザー操作のシミュレーション

ユーザーのインタラクションをテストするための主なメソッド：

- `trigger('event')`: イベントをトリガーする（例：click, input, submit）
- `setValue(value)`: input, textarea, selectの値を設定する

```ts
// クリックイベントの例
await wrapper.find('button').trigger('click')

// 入力の例
await wrapper.find('input').setValue('新しい値')
```

## プロパティとイベントのテスト

Vue.jsアプリケーションでは、コンポーネント間のデータの受け渡し（props）と通信（イベント）が重要な役割を果たします。これらを適切にテストすることで、コンポーネント間の連携が正しく動作することを保証できます。

### プロパティとイベントのテストの重要性

1. **プロパティ（Props）のテスト**:

   - 親コンポーネントから子コンポーネントへのデータ受け渡しが正しく行われるか確認
   - デフォルト値が期待通りに機能するか検証
   - 型チェックやバリデーションが想定通りに動作するか検証
2. **イベント（Events）のテスト**:

   - 子コンポーネントから親コンポーネントへの通知が正しく行われるか確認
   - イベントが適切なタイミングで発火されるか検証
   - イベントに適切なデータが含まれているか検証

### 実装とテスト例

次に、プロパティを受け取り、イベントを発火するコンポーネントを作成してテストしましょう。

`components/UserForm.vue` ファイルを作成します：

```vue
<template>
  <form @submit.prevent="submitForm">
    <div>
      <label for="name">名前：</label>
      <input
        id="name"
        v-model="formData.name"
        :placeholder="placeholder"
        data-testid="name-input"
      />
    </div>
    <div>
      <label for="email">メール：</label>
      <input
        id="email"
        v-model="formData.email"
        type="email"
        data-testid="email-input"
      />
    </div>
    <button type="submit">送信</button>
  </form>
</template>

<script setup>
import { reactive, defineProps, defineEmits } from 'vue'

const props = defineProps({
  placeholder: {
    type: String,
    default: '名前を入力'
  },
  initialData: {
    type: Object,
    default: () => ({ name: '', email: '' })
  }
})

const emit = defineEmits(['submit'])

// フォームデータの初期化
const formData = reactive({ ...props.initialData })

// フォーム送信処理
const submitForm = () => {
  emit('submit', { ...formData })
}
</script>
```

次に、このコンポーネントをテストするファイル `components/UserForm.spec.ts` を作成します：

```ts
import UserForm from '@/components/UserForm.vue'
import { mount } from '@vue/test-utils'

describe('UserForm.vue', () => {
  it('デフォルトのプレースホルダーが表示されること', () => {
    const wrapper = mount(UserForm)
    expect(wrapper.find('[data-testid="name-input"]').attributes('placeholder')).toBe('名前を入力')
  })

  it('カスタムプレースホルダーが正しく表示されること', () => {
    const placeholder = 'フルネームを入力'
    const wrapper = mount(UserForm, {
      props: {
        placeholder
      }
    })
    expect(wrapper.find('[data-testid="name-input"]').attributes('placeholder')).toBe(placeholder)
  })

  it('初期データが正しく設定されること', async () => {
    const initialData = { name: 'テスト太郎', email: 'test@example.com' }
    const wrapper = mount(UserForm, {
      props: {
        initialData
      }
    })

    // 非同期更新を待つ
    await wrapper.vm.$nextTick()

    // input要素の値を検証
    expect(wrapper.find('[data-testid="name-input"]').element.value).toBe(initialData.name)
    expect(wrapper.find('[data-testid="email-input"]').element.value).toBe(initialData.email)
  })

  it('フォーム送信時にsubmitイベントが発火すること', async () => {
    const wrapper = mount(UserForm)

    // フォームデータを入力
    await wrapper.find('[data-testid="name-input"]').setValue('テスト花子')
    await wrapper.find('[data-testid="email-input"]').setValue('hanako@example.com')

    // フォームを送信
    await wrapper.find('form').trigger('submit')

    // イベントが発火したことを確認
    expect(wrapper.emitted()).toHaveProperty('submit')

    // イベントデータを検証
    const submitEvent = wrapper.emitted('submit')
    expect(submitEvent[0][0]).toEqual({
      name: 'テスト花子',
      email: 'hanako@example.com'
    })
  })
})

```

### テスト内容の詳細解説

このテストコードでは、UserFormコンポーネントの4つの重要な側面をテストしています：

#### デフォルトプロパティのテスト

```ts
it('デフォルトのプレースホルダーが表示されること', () => {
  const wrapper = mount(UserForm)
  expect(wrapper.find('[data-testid="name-input"]').attributes('placeholder')).toBe('名前を入力')
})
```

- このテストでは、propsが提供されなかった場合に、デフォルト値（`'名前を入力'`）が正しく適用されるかを確認しています
- `data-testid`属性を使用して要素を選択し、その`placeholder`属性の値を検証しています
- `data-testid`は、CSSクラスやIDが変更されても影響を受けないため、テストの安定性を向上させます

#### カスタムプロパティのテスト

```ts
it('カスタムプレースホルダーが正しく表示されること', () => {
  const placeholder = 'フルネームを入力'
  const wrapper = mount(UserForm, {
    props: {
      placeholder
    }
  })
  expect(wrapper.find('[data-testid="name-input"]').attributes('placeholder')).toBe(placeholder)
})
```

- mount関数の`props`オプションを使用して、テスト時にプロパティを渡しています
- これにより、親コンポーネントからプロパティが渡された場合の動作をシミュレートしています
- 渡したプロパティ値が実際にDOM要素に反映されるかを検証しています

#### 複雑なプロパティと非同期更新のテスト

```ts
it('初期データが正しく設定されること', async () => {
  const initialData = { name: 'テスト太郎', email: 'test@example.com' }
  const wrapper = mount(UserForm, {
    props: {
      initialData
    }
  })

  // 非同期更新を待つ
  await wrapper.vm.$nextTick()

  // input要素の値を検証
  expect(wrapper.find('[data-testid="name-input"]').element.value).toBe(initialData.name)
  expect(wrapper.find('[data-testid="email-input"]').element.value).toBe(initialData.email)
})
```

- オブジェクト型のプロパティを渡し、それが正しく処理されるかをテストしています
- `vm.$nextTick()`を使用して、Vueの非同期更新サイクルが完了するのを待っています
- これは、プロパティ変更後にDOMの更新が反映されるのを保証するために重要です
- input要素の`.element.value`プロパティにアクセスして、実際のDOM要素の値を検証しています

#### イベントの発火と送信データのテスト

```ts
it('フォーム送信時にsubmitイベントが発火すること', async () => {
  const wrapper = mount(UserForm)

  // フォームデータを入力
  await wrapper.find('[data-testid="name-input"]').setValue('テスト花子')
  await wrapper.find('[data-testid="email-input"]').setValue('hanako@example.com')

  // フォームを送信
  await wrapper.find('form').trigger('submit')

  // イベントが発火したことを確認
  expect(wrapper.emitted()).toHaveProperty('submit')

  // イベントデータを検証
  const submitEvent = wrapper.emitted('submit')
  expect(submitEvent[0][0]).toEqual({
    name: 'テスト花子',
    email: 'hanako@example.com'
  })
})
```

- `setValue()`メソッドを使用して、フォームの入力フィールドにデータを設定しています
- `trigger('submit')`を使用して、フォームの送信イベントをシミュレートしています
- `wrapper.emitted()`を使用して、コンポーネントから発火されたすべてのイベントを取得しています
- `toHaveProperty('submit')`で、`submit`イベントが発火されたことを確認しています
- `submitEvent[0][0]`で最初の`submit`イベントの最初の引数にアクセスし、送信されたデータが期待通りの形式と値であることを検証しています

このように、プロパティとイベントのテストでは、コンポーネントへの入力（プロパティ）と出力（イベント）の両方を検証することで、コンポーネントが期待通りに動作することを確認します。

## 非同期コンポーネントのテスト

APIからデータを取得するコンポーネントをテストしましょう。`components/UserList.vue` ファイルを作成します：

```vue
<template>
  <div>
    <h2>ユーザー一覧</h2>
    <div v-if="loading">読み込み中...</div>
    <div v-else-if="error">エラーが発生しました: {{ error }}</div>
    <ul v-else>
      <li v-for="user in users" :key="user.id" data-testid="user-item">
        {{ user.name }} ({{ user.email }})
      </li>
    </ul>
    <button @click="fetchUsers">更新</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { fetchUserData } from '@/utils/api'

// 状態
const users = ref([])
const loading = ref(false)
const error = ref(null)

// ユーザーデータの取得
const fetchUsers = async () => {
  loading.value = true
  error.value = null

  try {
    const data = await fetchUserData('all')
    users.value = data
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

// コンポーネントマウント時にデータを取得
onMounted(fetchUsers)
</script>
```

このコンポーネントをテストするファイル `components/UserList.spec.ts` を作成します：

```ts
import UserList from '@/components/UserList.vue'
import { fetchUserData } from '@/utils/api'
import { mount } from '@vue/test-utils'

// APIのモック
vi.mock('@/utils/api', () => ({
  fetchUserData: vi.fn()
}))

describe('UserList.vue', () => {
  beforeEach(() => {
    vi.resetAllMocks()
  })

  it('ユーザーデータを正しく表示すること', async () => {
    // モックデータ
    const mockUsers = [
      { id: 1, name: '山田太郎', email: 'taro@example.com' },
      { id: 2, name: '佐藤花子', email: 'hanako@example.com' }
    ]

    // モック関数の応答を設定
    fetchUserData.mockResolvedValue(mockUsers)

    // コンポーネントをマウント
    const wrapper = mount(UserList)

    // 非同期処理の完了を待つ
    await vi.waitFor(() => {
      return !wrapper.vm.loading
    })
    await wrapper.vm.$nextTick() // DOM更新を待つ

    // ユーザーリストが表示されていることを確認
    const items = wrapper.findAll('[data-testid="user-item"]')
    expect(items.length).toBe(2)
    expect(items[0].text()).toContain('山田太郎')
    expect(items[1].text()).toContain('佐藤花子')
  })

  it('データ取得中にローディングが表示されること', async () => {
    // 解決されないPromiseを返すモック
    fetchUserData.mockImplementation(() => new Promise(() => {}))

    const wrapper = mount(UserList)
    await wrapper.vm.$nextTick() // DOM更新を待つ

    // ローディングメッセージが表示されていることを確認
    expect(wrapper.text()).toContain('読み込み中...')
  })

  it('エラー発生時にエラーメッセージが表示されること', async () => {
    // エラーを返すモック
    fetchUserData.mockRejectedValue(new Error('APIエラー'))

    const wrapper = mount(UserList)

    // 非同期処理の完了を待つ
    await vi.waitFor(() => {
      return !wrapper.vm.loading
    })
    await wrapper.vm.$nextTick() // DOM更新を待つ

    // エラーメッセージが表示されていることを確認
    expect(wrapper.text()).toContain('エラーが発生しました')
    expect(wrapper.text()).toContain('APIエラー')
  })

  it('更新ボタンをクリックするとデータが再取得されること', async () => {
    // 初回のモックデータ
    const initialUsers = [{ id: 1, name: '初期ユーザー' }]
    fetchUserData.mockResolvedValue(initialUsers)

    const wrapper = mount(UserList)

    // 初回データ取得の完了を待つ
    await vi.waitFor(() => !wrapper.vm.loading)
    await wrapper.vm.$nextTick() // DOM更新を待つ

    // モックを更新
    const updatedUsers = [
      { id: 1, name: '更新ユーザー1' },
      { id: 2, name: '更新ユーザー2' }
    ]
    fetchUserData.mockResolvedValue(updatedUsers)

    // 更新ボタンをクリック
    await wrapper.find('button').trigger('click')

    // データ取得の完了を待つ
    await vi.waitFor(() => !wrapper.vm.loading)
    await wrapper.vm.$nextTick() // DOM更新を待つ

    // APIが再度呼ばれたことを確認
    expect(fetchUserData).toHaveBeenCalledTimes(2)

    // 更新されたデータが表示されていることを確認
    const items = wrapper.findAll('[data-testid="user-item"]')
    expect(items.length).toBe(2)
    expect(items[0].text()).toContain('更新ユーザー1')
  })
})

```

## コンポーザブルを使ったコンポーネントのテスト

最後に、Vueのコンポーザブル（Composables）を使ったコンポーネントのテストを見ていきましょう。

まず、簡単なコンポーザブルを作成します。`composables/useCounter.ts` ファイルを作成します：

```ts
import { ref } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)

  const increment = () => count.value++
  const decrement = () => count.value--
  const reset = () => count.value = initialValue

  return {
    count,
    increment,
    decrement,
    reset
  }
}
```

この `useCounter` コンポーザブルを使ったコンポーネントを作成します。`components/ComposableCounter.vue` ファイルを作成します：

```vue
<template>
  <div>
    <h2>コンポーザブル利用カウンター</h2>
    <p>カウント: {{ count }}</p>
    <button @click="increment">増やす</button>
    <button @click="decrement">減らす</button>
    <button @click="reset">リセット</button>
  </div>
</template>

<script setup>
import { useCounter } from '@/composables/useCounter'

// コンポーザブルを利用
const { count, increment, decrement, reset } = useCounter(10) // 初期値10で開始
</script>
```

このコンポーネントをテストするファイル `components/ComposableCounter.spec.ts` を作成します：

```ts
import { mount } from '@vue/test-utils'
import ComposableCounter from '@/components/ComposableCounter.vue'

describe('ComposableCounter.vue', () => {
  it('コンポーザブルとの統合テスト', async () => {
    // コンポーネントをマウント
    const wrapper = mount(ComposableCounter)

    // 初期値の検証
    expect(wrapper.text()).toContain('カウント: 10')

    // 増やすボタンのクリック
    await wrapper.findAll('button')[0].trigger('click')
    expect(wrapper.text()).toContain('カウント: 11')

    // 減らすボタンのクリック
    await wrapper.findAll('button')[1].trigger('click')
    expect(wrapper.text()).toContain('カウント: 10')

    // リセットボタンのクリック
    await wrapper.findAll('button')[0].trigger('click') // 11にする
    await wrapper.findAll('button')[0].trigger('click') // 12にする
    await wrapper.findAll('button')[2].trigger('click') // リセット
    expect(wrapper.text()).toContain('カウント: 10')
  })
})
```

mockを使うと、コンポーザブルの関数呼び出しを監視できます

`components/ComposableCounter.mock.spec.ts`

```ts
import { mount } from '@vue/test-utils'
import ComposableCounter from '@/components/ComposableCounter.vue'
import { useCounter } from '@/composables/useCounter'

// useCounter のモックを定義
vi.mock('@/composables/useCounter')

describe('ComposableCounter.vue', () => {
  // モック用の関数を定義
  const incrementMock = vi.fn()
  const decrementMock = vi.fn()
  const resetMock = vi.fn()

  beforeEach(() => {
    // 各テストの前にモックの状態をクリア
    vi.clearAllMocks()

    // useCounterがモックされた値を返すように設定
    // as は型推論を補助するために使用
    vi.mocked(useCounter).mockReturnValue({
      count: 10,
      increment: incrementMock,
      decrement: decrementMock,
      reset: resetMock,
    })
  })

  it('コンポーザブルが正しく使用されること', () => {
    // コンポーネントをマウント
    const wrapper = mount(ComposableCounter)

    // 初期値の検証
    expect(wrapper.text()).toContain('カウント: 10')
    expect(useCounter).toHaveBeenCalled() // useCounterが呼び出されたことを確認

    // ボタンクリックのテスト
    const buttons = wrapper.findAll('button')

    buttons[0].trigger('click') // 増やす
    expect(incrementMock).toHaveBeenCalled()

    buttons[1].trigger('click') // 減らす
    expect(decrementMock).toHaveBeenCalled()

    buttons[2].trigger('click') // リセット
    expect(resetMock).toHaveBeenCalled()
  })
})
```

## まとめ

このセクションでは、Vitestを使ったVueコンポーネントのテストの基本を学びました：

1. **セットアップ**: テスト環境の設定方法
2. **基本的なレンダリングテスト**: コンポーネントが正しくレンダリングされるかの確認
3. **ユーザー操作のテスト**: クリックや入力などのユーザー操作のシミュレーション
4. **プロパティとイベントのテスト**: propsの処理とイベントの発火の検証
5. **非同期処理のテスト**: API呼び出しなどの非同期処理を含むコンポーネントのテスト
6. **コンポーザブルを使ったコンポーネントのテスト**: コンポーザブルのモックと統合テスト

これらの技術を組み合わせることで、Vueアプリケーションの品質と信頼性を高めることができます。コンポーネントテストは、ユニットテストとE2Eテストの中間に位置し、開発速度と信頼性のバランスがとれたテスト戦略を実現します。

次のステップとして、さらに複雑なコンポーネントや、Vuexストアとの連携、ルーティングを含むコンポーネントのテストに進むことができます。
z
