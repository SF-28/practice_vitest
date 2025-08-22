# 7. CIとの統合

## 📝 概要

継続的インテグレーション（CI）は、開発者がコードを共有リポジトリに頻繁に統合することを推奨するソフトウェア開発手法です。統合のたびに自動ビルドとテストが実行され、問題を早期に発見することができます。

このセクションでは、GitHub Actionsを使って、Vitestテストを自動実行する方法を学びます：

- GitHub Actionsの基本的な理解
- Vitestテストを実行するワークフローの設定
- テストカバレッジレポートの生成と表示
- プルリクエストでのテスト自動実行

## ⚙️ GitHub Actionsとは

GitHub Actionsは、GitHubが提供するCI/CDサービスで、リポジトリで発生するイベント（プッシュ、プルリクエストなど）をトリガーにして自動化ワークフローを実行できます。

### 主な特徴

- GitHubリポジトリに統合されている
- YAMLファイルでワークフローを定義
- さまざまなランナー環境（Ubuntu, Windows, macOSなど）
- 豊富な共有アクションとマーケットプレイス
- プルリクエスト内での結果表示

## 🛠️ Vitestのワークフロー設定

### 1. ワークフローファイルの作成

GitHub Actionsのワークフローはリポジトリの`.github/workflows`ディレクトリ内のYAMLファイルで定義します。

プロジェクトのルートディレクトリに以下の構造でファイルを作成しましょう：

```bash
my-nuxt-app/
├── .github/
│   └── workflows/
│       └── vitest.yml  👈 ここを作成
```

### 2. 基本的なワークフローの定義

`vitest.yml`ファイルに以下の内容を記述します：

```yaml
name: Vitest Tests

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '23'
          cache: 'pnpm'

      - name: Install pnpm
        uses: pnpm/action-setup@v3
        with:
          version: 10
          run_install: false

      - name: Install dependencies
        run: pnpm install

      - name: Run tests
        run: pnpm test
```

### 主な要素の説明

- `name`: ワークフローの名前
- `on`: ワークフローを実行するトリガーイベント
  - `push`: 指定ブランチへのプッシュ時に実行
  - `pull_request`: 指定ブランチへのプルリクエスト時に実行
- `jobs`: 実行するジョブの定義
  - `test`: ジョブの名前
  - `runs-on`: 実行環境（ここではUbuntu最新版）
- `steps`: 順番に実行する手順
  - `actions/checkout`: リポジトリのチェックアウト
  - `actions/setup-node`: Node.js環境のセットアップ
  - `pnpm/action-setup`: pnpmのセットアップ
  - 依存関係のインストール
  - テストの実行

## 🚀 カバレッジレポートとテスト結果の追加

テストだけでなく、カバレッジレポートも生成し、プルリクエスト上でテスト結果を確認できるようにしましょう。

### 1. JUnitレポート設定の追加

まず、`vitest.config.ts`にJUnitレポート設定を追加します：

```typescript
export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true,
    environment: 'happy-dom',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      // 省略...
    },
    // JUnitレポートの追加
    outputFile: {
      json: './test-results.json',
      junit: './junit.xml'
    },
    reporters: ['default', 'json', 'junit'],
  },
  // 省略...
})
```

### 2. テスト結果を表示するワークフロー

先ほどの`vitest.yml`を以下のように拡張します：

```yaml
name: Vitest Tests

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '23'
          cache: 'pnpm'

      - name: Install pnpm
        uses: pnpm/action-setup@v3
        with:
          version: 10
          run_install: false

      - name: Install dependencies
        run: pnpm install

      - name: Run tests with coverage
        run: pnpm test:coverage

      # テスト結果をプルリクエストに表示
      - name: Test Report
        uses: dorny/test-reporter@v1
        if: success() || failure()    # テストが成功しても失敗しても実行
        with:
          name: Vitest Tests
          path: junit.xml
          reporter: jest-junit

      # カバレッジレポートをアーティファクトとして保存
      - name: Archive code coverage results
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
```

## 📊 テスト結果の詳細表示

### 1. プルリクエスト上での詳細なテスト結果

### 2. プルリクエストへのコメント追加

テスト結果をプルリクエストにコメントとして追加することで、レビュアーが結果を確認しやすくなります：

```yaml
- name: Comment on PR with Test Results
  if: github.event_name == 'pull_request'
  uses: actions/github-script@v7
  with:
    script: |
      const fs = require('fs');
      const testResults = JSON.parse(fs.readFileSync('./test-results.json', 'utf8'));

      const totalTests = testResults.numTotalTests;
      const passedTests = testResults.numPassedTests;
      const failedTests = testResults.numFailedTests;

      const summary = `### テスト結果 📊
      - 全テスト: ${totalTests}
      - 成功: ${passedTests}
      - 失敗: ${failedTests}

      ${failedTests > 0 ? '❌ 失敗したテストがあります' : '✅ すべてのテストが成功しました'}`;

      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: summary
      })
```

## 🔒 失敗時の対応

### 1. テスト失敗時のワークフローの設定

テストが失敗した場合のみ特定のアクションを実行したい場合は、条件式を使用します：

```yaml
- name: Notify on test failure
  if: failure()
  uses: actions/github-script@v7
  with:
    script: |
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: '❌ テストが失敗しました。詳細なレポートをご確認ください。'
      })
```

### 2. ステータスバッジの追加

リポジトリのREADMEにテストステータスバッジを追加することで、プロジェクトの健全性を簡単に確認できます：

```markdown
[![Vitest Tests](https://github.com/ユーザー名/リポジトリ名/actions/workflows/vitest.yml/badge.svg)](https://github.com/ユーザー名/リポジトリ名/actions/workflows/vitest.yml)
```

## 📝 実習: GitHub Actionsの設定

### 実習1: 基本的なワークフローを作成する

1. `.github/workflows`ディレクトリを作成します：

   ```bash
   mkdir -p .github/workflows
   ```

2. `vitest.yml`ファイルを作成し、基本的なワークフロー設定を記述します：

   ```bash
   touch .github/workflows/vitest.yml
   ```

3. ファイルに先ほどの基本的なワークフロー定義を記述します

4. 変更をコミットしてプッシュします：

   ```bash
   git add .github/workflows/vitest.yml
   git commit -m "Add GitHub Actions workflow for Vitest"
   git push
   ```

5. GitHubリポジトリの「Actions」タブで、ワークフローが実行されることを確認します

### 実習2: テスト結果とカバレッジレポートの追加

1. `vitest.config.ts`を編集してJUnitレポート設定を追加します

2. `vitest.yml`ワークフローファイルにテスト結果表示とカバレッジレポート関連の設定を追加します

3. 変更をコミットしてプッシュします：

   ```bash
   git add vitest.config.ts .github/workflows/vitest.yml
   git commit -m "Add test reporting and coverage to GitHub Actions workflow"
   git push
   ```

4. プルリクエストを作成し、テスト結果が自動的にPR上に表示されることを確認します

## ⭐ チャレンジ課題

1. ワークフローをカスタマイズして、テストの実行時間が30秒を超える場合に警告を表示するステップを追加しましょう

2. マトリックスビルドを設定して、複数のNode.jsバージョン（例：16, 18, 20）でテストを実行するようにワークフローを拡張しましょう

3. プルリクエストが作成されたときに、変更されたファイルに関連するテストのみを実行する最適化を実装してみましょう

## 🔍 まとめ

このセクションでは、GitHub Actionsを使用してVitestテストを自動化する方法について学びました：

- GitHub Actionsの基本的な概念と設定方法
- Vitestテストを自動実行するワークフローの作成方法
- テストカバレッジレポートの生成と可視化
- プルリクエストでのテスト結果の表示

CI/CDパイプラインにテストを組み込むことで、以下のメリットがあります：

- コードの品質を継続的に確保できる
- 問題を早期に発見できる
- チーム全体でのコード品質への意識が高まる
- マージ前に自動的にコードの検証ができる

GitHub Actionsを活用して、効率的な開発ワークフローを確立しましょう。

## 📚 参考資料

- [GitHub Actions公式ドキュメント](https://docs.github.com/ja/actions)
- [Vitest公式ドキュメント - CI統合](https://vitest.dev/guide/ci.html)
- [dorny/test-reporter アクション](https://github.com/dorny/test-reporter)
- [pnpm GitHub Actions](https://pnpm.io/continuous-integration#github-actions)
- [actions/setup-node](https://github.com/actions/setup-node)
