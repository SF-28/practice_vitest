# Vitest ハンズオン

## 👋 はじめに

このリポジトリは、**Vitest**を使ったフロントエンドテストの基礎を学ぶためのハンズオン資料です。

## 🎯 対象

- Nuxt 3

## 🛠️ 前提条件

- Node.js 23.11.0
- pnpm 10.11.0
- JavaScript/TypeScriptの基本的な知識

## 🚀 Vitest (ヴィーテスト) とは？

Vitestは、Viteをベースにした高速なJavaScript/TypeScriptのテストフレームワークです。以下の特徴があります：

- **高速な実行速度**: HMR（Hot Module Replacement）をサポートし、変更されたファイルのみを再実行
- **Viteの設定との互換性**: Viteプロジェクトとシームレスに統合
- **Jest互換API**: Jestと似たAPIを持ち、既存のJestのテストコードを簡単に移行可能
- **ESM対応**: ECMAScript Modulesをネイティブサポート
- **TypeScript対応**: TypeScriptをそのままサポート
- **スナップショットテスト**: UIコンポーネントのスナップショットテストに対応

## 📋 ハンズオン概要

このハンズオンでは、以下の内容について実践的に学びます：

1. **基本セットアップ**: VitestをVueプロジェクトに導入する方法
2. **基本的なテスト**: 単純な関数のテスト方法
3. **モック**: 依存関係のモック方法
4. **コンポーネントテスト**: UIコンポーネントのテスト方法
5. **スナップショットテスト**: コンポーネントの表示をスナップショットでテストする方法
6. **カバレッジレポート**: コードカバレッジの計測方法
7. **CIとの統合**: GitHub ActionsなどのCIでVitestを使用する方法

## 📚 ハンズオン

### [1. セットアップ](./handson/1_setup.md)

### [2. 基本的なテスト](./handson/2_basic_test.md)

### [3. モック](./handson/3_mock.md)

### 4. コンポーネントテスト

- Vue Test Utilsとの統合
<!-- - React Testing Libraryとの統合 -->
<!-- - Svelte Testing Libraryとの統合 -->

### 5. スナップショットテスト

- スナップショットの作成と更新
- インラインスナップショットの使い方

### 6. カバレッジレポート

- カバレッジ設定
- レポートの見方と活用法

### 7. CIとの統合

- GitHub Actionsの設定例
<!-- - GitLab CIの設定例 -->

## 🔗 参考リソース

- [Vitest公式ドキュメント](https://vitest.dev/)
- [Vite公式ドキュメント](https://vitejs.dev/)
- [Vue Test Utils](https://vue-test-utils.vuejs.org/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Svelte Testing Library](https://testing-library.com/docs/svelte-testing-library/intro/)
