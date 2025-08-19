# 2. 基本的なテスト

このセクションでは、Vitestを使用した基本的なテスト手法を学びます

## 基本構文

`utils/sum.spec.ts`ファイルを例に見ていきましょう

```ts
import { sum } from '@/utils/sum'; // テスト対象をインポートする

test('足し算のテスト', () => {
  expect(sum(1, 2)).toBe(3); // 1 + 2 が 3 になることを確認
});
```

- test : テストケースを定義します。`it` と書くこともできます
- expect : アサーションを定義します
- toBe : 厳密等価性を確認するマッチャーです(他のマッチャーは後述)

意味としては以下のような形になる

```ts
test(<テストケース名>, () => {
  expect(<検証したい処理>).toBe(<期待する値>);
})
```

---

## テストファイルの命名規則

Vitestでは、デフォルトで以下のパターンのファイルを自動的にテストファイルとして認識します

- `*.spec.ts`: 例えば `sum.spec.ts`
- `*.test.ts`: 例えば `sum.test.ts`

テストファイルは通常、テスト対象のファイルと同じ名前にテスト用の拡張子（.spec または .test）を付けて命名します。これにより、テストとテスト対象のコードの関連性が明確になります。

---

## テストファイルの置き場所

テスト対象と同じディレクトリか、`tests/`ディレクトリに配置するのが一般的です。

例えば、`utils/sum.ts`のテストは `utils/sum.spec.ts` もしくは `tests/utils/sum.spec.ts`に配置します。今回のハンズオンでは、テスト対象と同じディレクトリに配置します。

---

## テストを実行するコマンド

```bash
# 全てのテストを実行
pnpm test

# 特定のファイルのテストのみ実行
pnpm test utils/sum.spec.ts
```

---

### 練習問題 1

以下のファイルに対するテストを作成して実行しましょう

`utils/subtract.ts`

```ts
export function subtract(a: number, b: number): number {
  return a - b;
}
```

---

## テストスイート

テストスイートは、関連するテストケースをグループ化するためのものです。`describe`ブロックを使用して定義します。

```ts
import { sum } from '@/utils/sum';
import { subtract } from '@/utils/subtract';

describe('数学演算', () => {
  test('足し算のテスト', () => {
    expect(sum(1, 2)).toBe(3);
  });

  test('引き算のテスト', () => {
    expect(subtract(5, 2)).toBe(3);
  });
});
```

### テストスイートのネスト

テストスイート（`describe`ブロック）は、ネストして使うことができます。

```ts
import { sum } from '@/utils/sum';
import { subtract } from '@/utils/subtract';

describe('数学演算', () => {
  describe('足し算', () => {
    test('1 + 2 は 3 になる', () => {
      expect(1 + 2).toBe(3);
    });

    test('2 + 3 は 5 になる', () => {
      expect(2 + 3).toBe(5);
    });
  });

  describe('引き算', () => {
    test('5 - 2 は 3 になる', () => {
      expect(5 - 2).toBe(3);
    });

    test('10 - 4 は 6 になる', () => {
      expect(10 - 4).toBe(6);
    });
  });
});
```

---

## アサーション（expect）の使い方

Vitestには、様々な状況でのアサーションをサポートするマッチャーが豊富に用意されています。

`tests/assertions.spec.ts`を作成して、よく使われるマッチャーを試してみましょう：

```ts
describe('様々なアサーション', () => {
  // 等価性テスト
  test('等価性テスト', () => {
    // プリミティブ値の比較
    expect(2 + 2).toBe(4); // 厳密等価 (===)

    // オブジェクトの比較
    const obj = { name: 'テスト' };
    expect(obj).toEqual({ name: 'テスト' }); // 構造的等価
    expect(obj).not.toBe({ name: 'テスト' }); // 異なるオブジェクト参照
  });

  // 真偽値テスト
  test('真偽値テスト', () => {
    expect(true).toBeTruthy();
    expect(false).toBeFalsy();
    expect(null).toBeNull();
    expect(undefined).toBeUndefined();
    expect('値あり').toBeDefined();
  });

  // 数値比較
  test('数値比較', () => {
    expect(10).toBeGreaterThan(5);
    expect(5).toBeLessThan(10);
    expect(10).toBeGreaterThanOrEqual(10);
    expect(10).toBeLessThanOrEqual(10);
  });

  // 配列のテスト
  test('配列テスト', () => {
    const array = [1, 2, 3];
    expect(array).toContain(2);
    expect(array).toHaveLength(3);
  });

  // オブジェクトのテスト
  test('オブジェクトテスト', () => {
    const obj = { name: 'テスト', age: 30 };
    expect(obj).toHaveProperty('name');
    expect(obj).toHaveProperty('age', 30);
  });

  // 例外のテスト
  test('例外テスト', () => {
    const throwError = () => { throw new Error('テストエラー'); };
    expect(throwError).toThrow();
    expect(throwError).toThrow('テストエラー');
  });
});
```

---

### 練習問題 2

以下の関数をテストするコードを書いてください：

`utils/formatUserName.ts`

```ts
export function formatUserName(name: string | null | undefined): string {
  if (name === null || name === undefined) {
    return 'ゲスト';
  }

  if (name.length === 0) {
    return 'ゲスト';
  }

  return name.trim();
}
```

以下の条件でテストを作成します：

- nullが渡された場合は「ゲスト」を返す
- undefinedが渡された場合は「ゲスト」を返す
- 空文字が渡された場合は「ゲスト」を返す
- 前後に空白がある文字列の場合は、それをトリムして返す
- 通常の文字列はそのまま返す

---

## setup(前処理) / teardown(後処理)

テストの前後に特定の処理を行いたいケースがあります

- テスト前に初期化して毎回同じ状態でテストが走るようにする
- テスト後にリソースを解放・削除する、など

そのためには以下の関数を使用します

- `beforeEach`: 各テストの前に実行される
- `afterEach`: 各テストの後に実行される
- `beforeAll`: すべてのテストの前に1回だけ実行される
- `afterAll`: すべてのテストの後に1回だけ実行される

`tests/setup.spec.ts`を作成し、以下のコードを試してみましょう：

```ts
describe('セットアップとクリーンアップ', () => {
  let testValue: number;

  // すべてのテストの前に1回だけ実行
  beforeAll(() => {
    console.log('テストスイートの開始');
  });

  // すべてのテストの後に1回だけ実行
  afterAll(() => {
    console.log('テストスイートの終了');
  });

  // 各テストの前に実行
  beforeEach(() => {
    testValue = 1;
    console.log('テストの開始', testValue);
  });

  // 各テストの後に実行
  afterEach(() => {
    console.log('テストの終了', testValue);
  });

  test('テスト1', () => {
    testValue += 1;
    expect(testValue).toBe(2);
  });

  test('テスト2', () => {
    // 各テストの前にbeforeEachが実行されるため、testValueは1にリセットされている
    testValue += 2;
    expect(testValue).toBe(3);
  });
});
```

このテストを実行すると、コンソールに各フックの実行順序が表示されます：

```bash
pnpm test tests/setup.spec.ts
```

### ネストしたdescribeでのセットアップ

`beforeEach`や`afterEach`などのフックは、スコープに従って実行されます。

親の`describe`ブロックのフックも、子のテストに適用されます。

`tests/nested-setup.spec.ts`を作成して、以下のコードを試してみましょう：

```ts
describe('親テストスイート', () => {
  let counter = 0;

  beforeEach(() => {
    counter += 1;
    console.log('親テストスイートで加算');
  });

  test('親のテスト', () => {
    expect(counter).toBe(1);
    console.log('親のテスト', counter);
  });

  describe('子テストスイート', () => {
    beforeEach(() => {
      counter += 1;
        console.log('子テストスイートで加算');
    });

    test('子のテスト', () => {
      // 親のbeforeEach (+1) と子のbeforeEach (+1) が実行される
      expect(counter).toBe(3);
      console.log('子のテスト', counter);
      console.log('テスト実行は2回なのにカウンターが2でなく3になっている');
    });
  });
});
```

このテストを実行すると、親の`beforeEach`と子の`beforeEach`がどのように実行されるかがわかります。

```bash
pnpm test tests/nested-setup.spec.ts
```

---

### 練習問題 3

シンプルな買い物カートのテストを書いてみましょう。

`utils/shoppingCart.ts`

```ts
// ShoppingCart クラス
export default class ShoppingCart {
  items: string[] = [];

  addItem(item: string): void {
    this.items.push(item);
  }

  removeItem(item: string): void {
    const index = this.items.indexOf(item);
    if (index !== -1) {
      this.items.splice(index, 1);
    }
  }

  getItemCount(): number {
    return this.items.length;
  }

  clearCart(): void {
    this.items = [];
  }
}
```

以下の条件を満たすテストを作成してください：

- 各テストの前に新しいShoppingCartインスタンスを作成する
- カートに商品を追加するテスト
- カートから商品を削除するテスト
- カートを空にするテスト
- カートの商品数を取得するテスト

---

## まとめ

この章では、以下の内容を学びました：

- テストの基本構文
- テストファイルの命名規則
- テストファイルの置き場所
- テストを実行するコマンド
- テストスイート
- アサーションの使い方
- setup/teardownの使い方

次の章では、外部の依存関係をモックする方法について学んでいきます。
