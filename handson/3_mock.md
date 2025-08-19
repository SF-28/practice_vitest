# 3. モック

このセクションでは、Vitestを使用した様々なモック技術を学びます。モックは外部依存関係をシミュレートし、テストを隔離するために重要なテクニックです。

## モックとは何か

モックとは、テスト中に本物のオブジェクトや関数の代わりに使用する、コントロール可能な模倣版です。簡単に言えば、「本物の代わりに使える偽物」です。

### なぜモックが必要なのか

実際の開発では、テスト対象のコードが以下のような外部要素に依存していることがよくあります：

1. **外部API呼び出し**: 実際のAPIを呼び出すと、テストが遅くなったり、不安定になったりします
2. **データベース操作**: 実際のデータが変更される可能性があります
3. **ファイル操作**: テスト環境によって結果が異なる可能性があります
4. **現在時刻の取得**: テスト実行のたびに異なる値になってしまいます
5. **ランダム値の生成**: 予測不可能な結果になります

これらの外部依存があると、テストが：

- **遅くなる**: 外部サービスの応答を待つ必要がある
- **不安定になる**: ネットワーク問題などで時々失敗する
- **予測不可能になる**: 実行するたびに異なる結果が得られる
- **他のテストに影響を与える**: 共有リソースを変更してしまう

### モックの役割と利点

モックを使用することで以下のような利点があります：

- **テストを高速化**: 実際のAPI呼び出しなどを省略できる
- **テストを安定化**: 外部環境に左右されない
- **特定条件のテスト**: エラー発生時など特定の状況をシミュレートできる
- **コードの分離**: 依存関係を切り離してコードの一部だけをテストできる
- **並行開発**: APIが完成していなくても、それを使うコードのテストができる

### モックの具体的な使用シーン

例えば、以下のようなシチュエーションでモックが役立ちます：

- **天気APIを使うアプリケーション**: 実際にAPIを呼び出さずに、晴れ/雨/雪などの応答をシミュレート
- **ユーザー認証機能**: データベースにアクセスせずに、ログイン成功/失敗をシミュレート
- **現在日時に依存する機能**: 「常に2023年1月1日」のように日付を固定してテスト
- **支払い処理**: 実際にクレジットカード決済を行わずに成功/失敗をシミュレート

## 関数のモック

まず、基本的な関数のモックから始めましょう。関数のモックは、最も基本的で頻繁に使用するモックの形態です。

### vi.fn() - 関数のモック

`vi.fn()`を使って関数をモックできます。これにより、以下のことが可能になります：

1. **関数呼び出しの追跡**: どのように呼び出されたか、何回呼び出されたかを記録
2. **戻り値の制御**: テストケースごとに異なる戻り値を設定可能
3. **実装の置き換え**: 関数の内部動作を完全に変更可能
4. **エラー発生のシミュレート**: 例外をスローするようにモック化

関数モックは特に「依存性の注入」パターン（関数を引数として渡す設計）と相性が良く、コールバックや高階関数のテストに役立ちます。

`utils/calculator.ts`を作成しましょう：

```ts
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export function calculate(a: number, b: number, operation: (x: number, y: number) => number): number {
  return operation(a, b);
}
```

次に、`utils/calculator.spec.ts`を作成してテストします：

```ts
import { add, calculate } from '@/utils/calculator';
import { vi } from 'vitest';

describe('関数のモック', () => {
  test('モック関数が呼び出されることを検証', () => {
    // モック関数の作成
    const mockOperation = vi.fn();

    // モック関数を引数として渡す
    calculate(5, 3, mockOperation);

    // モック関数が呼び出されたことを検証
    expect(mockOperation).toHaveBeenCalled();
    expect(mockOperation).toHaveBeenCalledWith(5, 3);
    expect(mockOperation).toHaveBeenCalledTimes(1);
  });

  test('モック関数の戻り値を指定', () => {
    // 戻り値を指定したモック関数の作成
    const mockOperation = vi.fn(() => 15);

    // モック関数を使用
    const result = calculate(5, 3, mockOperation);

    // 戻り値を検証
    expect(result).toBe(15);
  });

  test('モック関数の実装を変更', () => {
    // 内部処理を指定したモック関数の作成
    const mockOperation = vi.fn().mockImplementation((a, b) => a * b);

    // モック関数を使用
    const result = calculate(5, 3, mockOperation);

    // 戻り値を検証
    expect(result).toBe(15); // 5 * 3 = 15
  });

  test('モック関数の戻り値を連続して変更', () => {
    // 複数回の呼び出しで異なる値を返すモック
    const mockOperation = vi.fn()
      .mockReturnValueOnce(8)
      .mockReturnValueOnce(12)
      .mockReturnValue(20);

    // 1回目の呼び出し
    expect(calculate(1, 1, mockOperation)).toBe(8);

    // 2回目の呼び出し
    expect(calculate(2, 2, mockOperation)).toBe(12);

    // 3回目以降の呼び出し
    expect(calculate(3, 3, mockOperation)).toBe(20);
    expect(calculate(4, 4, mockOperation)).toBe(20);
  });
});
```

テストを実行します

```sh
pnpm test utils/calculator.spec.ts
```

#### モック関数の動作解説

上記のテストコードで使用している様々なモック機能を詳しく見てみましょう：

**基本的なモック作成**：

```ts
const mockOperation = vi.fn();
```

これだけで、呼び出しを追跡できる関数が作成されます。初期状態では`undefined`を返します。

**呼び出し検証**：

```ts
expect(mockOperation).toHaveBeenCalled();       // 少なくとも1回呼び出されたか
expect(mockOperation).toHaveBeenCalledWith(5, 3); // 特定の引数で呼び出されたか
expect(mockOperation).toHaveBeenCalledTimes(1); // 呼び出し回数
```

これらのマッチャーを使って、関数が期待通りに呼び出されたかを検証できます。

**戻り値の指定**：

```ts
// 方法1: コンストラクタ引数で指定
const mockOperation = vi.fn(() => 15);

// 方法2: mockReturnValueを使用
const mockOperation = vi.fn().mockReturnValue(15);
```

どちらの方法でも、モック関数が常に15を返すように設定できます。

**実装の上書き**：

```ts
const mockOperation = vi.fn().mockImplementation((a, b) => a * b);
```

引数を受け取って処理する実装を定義できます（この例では掛け算を行う関数）。

**連続した戻り値**：

```ts
const mockOperation = vi.fn()
  .mockReturnValueOnce(8)   // 1回目の呼び出し
  .mockReturnValueOnce(12)  // 2回目の呼び出し
  .mockReturnValue(20);     // 3回目以降の呼び出し
```

複数回の呼び出しで異なる値を返すよう設定できます。

これらの機能を使うことで、テスト対象のコードが他のコードと正しく連携していることを検証できます。

### 実際の開発でのユースケース

例えば以下のようなシチュエーションで役立ちます：

1. **ユーザー入力の処理関数**：入力検証コールバックをモック化して様々なパターンをテスト
2. **イベントハンドラー**：クリックイベントハンドラーをモック化して、UI操作をシミュレート
3. **データ変換処理**：複雑な変換処理を単純な戻り値だけのモックに置き換え
4. **エラーハンドリング**：エラーコールバックをモック化して、例外処理のテスト

### 練習問題 1: 関数のモック

`utils/logger.ts`を作成してください：

```ts
export function logger(message: string): void {
  console.log(`[LOG]: ${message}`);
}

export function processData(data: string, logFn: (msg: string) => void): string {
  logFn(`Processing: ${data}`);
  return data.toUpperCase();
}
```

次に、`utils/logger.spec.ts`を作成し、以下のテストケースを実装してください：

1. `processData`関数に渡すロガー関数をモックし、呼び出されることを検証
2. モックされたロガー関数が正しい引数で呼び出されることを検証

## モジュールのモック (vi.mock)

関数単体をモックするだけでなく、モジュール全体（ファイル全体のエクスポート）をモックすることもできます。Vitestでは`vi.mock()`を使用して、これを実現できます。

### なぜモジュールをモックするのか

モジュールをモックする主な理由は以下の通りです：

1. **外部依存の隔離**: 例えばAPI呼び出しなど、外部サービスを使うモジュールをモックして隔離する
2. **実装を単純化**: 複雑なロジックを持つモジュールの動作を単純化する
3. **テストしにくい状況の再現**: データベース接続エラーなど、特定の状況をシミュレートする
4. **処理速度の向上**: 重い処理を含むモジュールを軽量なモック版に置き換える

### 実際の開発でのユースケース

モジュールモックが特に役立つシーンは以下のような場合です：

- **HTTPクライアント**: axios, fetchなどの通信ライブラリをモック化して、ネットワーク接続なしでテスト
- **ファイルシステム**: fs, path などのNode.jsモジュールをモック化して、実際のファイル操作なしでテスト
- **データベースクライアント**: MongoDBやMySQLのクライアントをモック化して、実DBへの接続なしでテスト
- **サードパーティーライブラリ**: 支払い処理APIなど、外部サービスのSDKをモック化

### 基本的なモジュールのモック

`utils/api.ts`を作成しましょう：

```ts
export async function fetchUserData(userId: string) {
  // 実際のAPIコールを想定
  const response = await fetch(`https://api.example.com/users/${userId}`);
  if (!response.ok) {
    throw new Error('Failed to fetch user data');
  }
  return await response.json();
}

export async function updateUserData(userId: string, data: any) {
  // 実際のAPIコールを想定
  const response = await fetch(`https://api.example.com/users/${userId}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(data),
  });
  if (!response.ok) {
    throw new Error('Failed to update user data');
  }
  return await response.json();
}
```

次に、`utils/api.spec.ts`を作成してテストします：

```ts
import { fetchUserData, updateUserData } from '@/utils/api';
import { vi, describe, it, expect, beforeEach } from 'vitest';

// モジュール全体をモック
vi.mock('@/utils/api', () => {
  return {
    fetchUserData: vi.fn(),
    updateUserData: vi.fn(),
  };
});

describe('APIモジュールのモック', () => {
  beforeEach(() => {
    // 各テスト前にモックをリセット
    vi.resetAllMocks();
  });

  it('fetchUserDataのモック', async () => {
    // モック関数の戻り値を設定
    const mockUser = { id: '123', name: 'テストユーザー' };
    (fetchUserData as any).mockResolvedValue(mockUser);

    // モック関数の呼び出し
    const result = await fetchUserData('123');

    // 検証
    expect(fetchUserData).toHaveBeenCalledWith('123');
    expect(result).toEqual(mockUser);
  });

  it('updateUserDataのモック', async () => {
    // モック関数の戻り値を設定
    const updatedUser = { id: '123', name: '更新済みユーザー' };
    (updateUserData as any).mockResolvedValue(updatedUser);

    // モック関数の呼び出し
    const userData = { name: '更新済みユーザー' };
    const result = await updateUserData('123', userData);

    // 検証
    expect(updateUserData).toHaveBeenCalledWith('123', userData);
    expect(result).toEqual(updatedUser);
  });

  it('fetchUserDataの例外をモック', async () => {
    // モック関数がエラーをスローするよう設定
    (fetchUserData as any).mockRejectedValue(new Error('API error'));

    // エラーが発生することを検証
    await expect(fetchUserData('123')).rejects.toThrow('API error');
  });
});
```

```sh
pnpm test utils/api.spec.ts
```

#### モジュールモックのポイント解説

このテストコードでは、以下の重要なテクニックを使用しています：

**モジュール全体のモック化**:

```ts
vi.mock('@/utils/api', () => {
  return {
    fetchUserData: vi.fn(),
    updateUserData: vi.fn(),
  };
});
```

この記述で、`@/utils/api`モジュールから実際にインポートする代わりに、指定したモック実装を使用します。本物のAPIにアクセスすることなくテストが可能になります。

**モックのリセット**:

```ts
beforeEach(() => {
  // 各テスト前にモックをリセット
  vi.resetAllMocks();
});
```

各テストケース間での干渉を防ぐため、テスト実行前にモックの状態をリセットします。

**非同期モックの設定**:

```ts
const mockUser = { id: '123', name: 'テストユーザー' };
(fetchUserData as any).mockResolvedValue(mockUser);
```

`mockResolvedValue`メソッドを使って、非同期関数のモックが解決する値を設定しています。これは`Promise.resolve()`で値を返すモック関数を作成します。

**エラーのモック**:

```ts
(fetchUserData as any).mockRejectedValue(new Error('API error'));
```

`mockRejectedValue`メソッドを使って、失敗するPromiseを返すようにモックしています。エラー処理のテストに役立ちます。

### 自動モックの実装置き換え

モジュール内の特定の関数の実装だけを置き換えることもできます。

`utils/user-service.ts`を作成します：

```ts
import { fetchUserData } from './api';

export async function getUserFullName(userId: string): Promise<string> {
  const userData = await fetchUserData(userId);
  return `${userData.firstName} ${userData.lastName}`;
}

export function formatUserName(user: { firstName: string; lastName: string }): string {
  return `${user.lastName}, ${user.firstName}`;
}
```

次に、`utils/user-service.spec.ts`を作成してテストします：

```ts
import { getUserFullName, formatUserName } from '@/utils/user-service';
import { fetchUserData } from '@/utils/api';
import { vi, describe, it, expect } from 'vitest';

// api.ts モジュールをモック
vi.mock('@/utils/api');

describe('UserService', () => {
  it('getUserFullNameが正しい名前を返すこと', async () => {
    // fetchUserDataのモック実装
    (fetchUserData as any).mockResolvedValue({
      firstName: '太郎',
      lastName: '山田'
    });

    // テスト対象関数の呼び出し
    const fullName = await getUserFullName('123');

    // 検証
    expect(fetchUserData).toHaveBeenCalledWith('123');
    expect(fullName).toBe('太郎 山田');
  });

  it('formatUserNameが正しい形式で名前を返すこと', () => {
    const user = { firstName: '太郎', lastName: '山田' };
    const formattedName = formatUserName(user);
    expect(formattedName).toBe('山田, 太郎');
  });
});
```

## スパイ (vi.spyOn)

`vi.spyOn`は既存のオブジェクトのメソッドを監視（スパイ）し、その呼び出しを追跡するために使用します。元の実装を置き換えることなく監視できるのが特徴です。

### スパイとモックの違い

スパイとモックの主な違いは以下の通りです：

1. **デフォルトの動作**:
   - スパイ: 元の実装をそのまま実行し、呼び出し情報だけを記録
   - モック: 元の実装を置き換え、指定した戻り値や代替実装を使用

2. **対象**:
   - スパイ: 既存のオブジェクトのメソッドやプロパティに設定
   - モック: 関数やモジュール全体を新たに作成または置き換え

3. **使用場面**:
   - スパイ: 元の実装を維持したまま呼び出し情報を確認したい場合
   - モック: 元の実装を完全に置き換えたい場合

### スパイが特に役立つ場面

スパイは以下のような場面で特に便利です：

1. **副作用の監視**: `console.log`や`localStorage.setItem`などの呼び出しを検証
2. **既存コードの非破壊的テスト**: 本番コードを変更せずに、呼び出し状況を監視
3. **オブジェクト指向設計のテスト**: クラスのメソッドが正しく他のメソッドを呼び出しているか確認
4. **一部の振る舞いだけを変更**: 元の実装はほぼそのままに、一部だけ変更したい場合

### オブジェクトメソッドの監視

`utils/math-utils.ts`を作成しましょう：

```ts
export const MathUtils = {
  add(a: number, b: number): number {
    return a + b;
  },

  multiply(a: number, b: number): number {
    return a * b;
  },

  calculateArea(radius: number): number {
    return this.multiply(Math.PI, this.multiply(radius, radius));
  }
};
```

次に、`utils/math-utils.spec.ts`を作成してテストします：

```ts
import { MathUtils } from '@/utils/math-utils';
import { vi, describe, it, expect, beforeEach } from 'vitest';

describe('MathUtils スパイ', () => {
  beforeEach(() => {
    // スパイをリセット
    vi.restoreAllMocks();
  });

  it('メソッド呼び出しを監視', () => {
    // multiplyメソッドにスパイを仕掛ける
    const multiplySpy = vi.spyOn(MathUtils, 'multiply');

    // メソッドを呼び出す
    MathUtils.calculateArea(5);

    // 検証（元の実装が実行される）
    expect(multiplySpy).toHaveBeenCalledTimes(2);
    expect(multiplySpy).toHaveBeenNthCalledWith(2, 5, 5);
  });

  it('メソッドの実装を置き換える', () => {
    // multiplyメソッドをモックに置き換える
    const multiplySpy = vi.spyOn(MathUtils, 'multiply').mockImplementation(() => 100);

    // 置き換えられたメソッドを呼び出す
    const result = MathUtils.calculateArea(5);

    // 検証
    expect(multiplySpy).toHaveBeenCalledTimes(2);
    expect(result).toBe(100); // 常に100を返すようにモックしたため
  });
});
```

### 組み込みオブジェクトのスパイ

JavaScriptの組み込みオブジェクト（例：console、Date、Math）にもスパイを仕掛けることができます。

`utils/date-utils.spec.ts`を作成しましょう：

```ts
import { vi, describe, it, expect, beforeEach } from 'vitest';

describe('組み込みオブジェクトのスパイ', () => {
  beforeEach(() => {
    // 各テスト前にスパイをリセット
    vi.restoreAllMocks();
  });

  it('console.logにスパイを仕掛ける', () => {
    // console.logにスパイを仕掛ける
    const consoleSpy = vi.spyOn(console, 'log');

    // console.logを使用
    console.log('テストメッセージ');

    // 検証
    expect(consoleSpy).toHaveBeenCalledWith('テストメッセージ');
  });

  it('現在時刻をモック', () => {
    // 2023年1月1日にDateを固定
    const mockDate = new Date(2023, 0, 1);
    vi.spyOn(Date, 'now').mockImplementation(() => mockDate.getTime());

    // 検証
    expect(new Date(Date.now()).getFullYear()).toBe(2023);
    expect(new Date(Date.now()).getMonth()).toBe(0); // 0-indexed (1月)
  });

  it('Math.randomをモック', () => {
    // Math.randomが常に0.5を返すようにモック
    vi.spyOn(Math, 'random').mockImplementation(() => 0.5);

    // 検証
    expect(Math.random()).toBe(0.5);
    expect(Math.random()).toBe(0.5);
  });
});
```

### 練習問題 2: スパイの使用

`utils/storage.ts`を作成してください：

```ts
export class Storage {
  private data: Record<string, any> = {};

  save(key: string, value: any): void {
    this.data[key] = value;
    console.log(`データを保存: ${key}`);
  }

  get(key: string): any {
    console.log(`データを取得: ${key}`);
    return this.data[key];
  }

  remove(key: string): void {
    console.log(`データを削除: ${key}`);
    delete this.data[key];
  }
}

export const storage = new Storage();
```

次に、`utils/storage.spec.ts`を作成し、以下のテストを実装してください：

1. `console.log`をスパイして、各メソッドが適切なログメッセージを出力することを検証
2. `storage.get`メソッドをスパイして、特定の値を返すように置き換える

## まとめ

このセクションでは、Vitestの主要なモック機能について学びました：

### 3種類のモックアプローチとその特徴

1. **`vi.fn()` - 関数のモック**
   - 単一関数のモック化
   - 呼び出し追跡、戻り値設定、実装置き換えが可能
   - 依存性注入されたコールバックのテストに最適

2. **`vi.mock()` - モジュールのモック**
   - モジュール全体（ファイル全体のエクスポート）のモック化
   - インポート文をそのままにして、内部実装を置き換え
   - 外部APIやデータベースアクセスなどの隔離に最適

3. **`vi.spyOn()` - スパイ**
   - オブジェクトのメソッドを非破壊的に監視
   - 元の実装を維持したまま呼び出し情報を収集
   - 必要に応じて一時的に実装を置き換え可能

### モック使用のベストプラクティス

- **必要最小限のモック**: テスト対象の境界となる部分だけをモック化し、内部実装は実際のコードを使う
- **テスト前のリセット**: `beforeEach`で`vi.resetAllMocks()`を呼び、テスト間の独立性を保つ
- **明確な検証**: 何を検証したいのかを明確にし、それに関連するアサーションだけを書く
- **実際の動作に近づける**: モックの動作は、可能な限り実際のコンポーネントの動作に近づける

### モックがもたらす効果

モックを適切に使用することで、以下の効果が得られます：

- **テストの高速化**: 外部依存を排除することで実行速度が向上
- **テストの安定性**: 外部環境の影響を受けないため、再現性が高い
- **エッジケースのテスト**: 通常発生しにくい状況（エラーなど）も容易に再現可能
- **コンポーネント単位のテスト**: 大きなシステムの一部だけを分離してテスト可能

次のセクションでは、これらのモック技術をさらに活用して、Vue.jsコンポーネントのテストに応用する方法を学びます。Vueコンポーネントのテストでは、プロップス、イベント、Vuexストア、ルーターなどのモックが必要になるケースが多く、今回学んだ技術が大いに役立ちます。
