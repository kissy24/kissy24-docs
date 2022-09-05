# GitHub Actions から Jira を操作する

## 1. はじめに

JiraとGitHubのタスクが紐づいていない事象を解決するためにしたことを忘備録がてら残しています。
目的はJira と GitHub を連携させ、2 重管理の手間を簡略化させることです。

---

## 2. Jira と GitHub を連携させる

Jira→GitHub と GitHub→Jira の 2 パターンがある。  
Jira には事前に GitHub のブランチ登録が必要。  
いずれも Jira の課題キーを利用する必要がある。(GitHub Actions の操作では一部不要)

### 2-1. Smart Commit

Git ブランチのコミットに Jira の課題キーを Prefix として記載することで、  
Jira と GitHub の特定のブランチを紐付ける事ができる。
コミットで他の Prefix を利用していると競合する可能性がある。

### 2-2. WorkFlows

Jira プロジェクトのワークフローのトリガーに Git の操作フラグを割り当てる。  
2021/07 時点では、チーム管理対象プロジェクトは利用できない。

### 2-3. Project automation

Jira プロジェクトの プロジェクト > プロジェクト名 > プロジェクトの管理 > アプリ で自動化ルールを作成できる。  
ルールの「いつ」等で GitHub 上のブランチやコミットを指定すること可能。

### 2-4. Jira のプラグイン

GitHub 連携用のプラグインがある。  
導入時に管理者権限がいるため、利用の障壁がそれなりにある。

### 2-5. GitHub Actions (これを採用)

GitHub Actions で Jira API を利用して Jira を編集する。  
GitHub ドリブンな場合は恩恵が大きい。  
他のCI含め、GitHub Actions の利用上限にかかる可能性はある。

---

## 3. 事前準備

### 3-1. Jira API キーの取得

1. `https://id.atlassian.com/manage/api-tokens` にログインする。
2. 「API トークンを作成する」を押す。
3. トークン(お好きなラベル名で)を登録する。
4. トークンを他の場所にコピペして保管する。

### 3-2. GitHub Secrets の登録

リポジトリ > Settings > Secrets を開き、以下を新規追加する

- JIRA_API_TOKEN: 3-1.で取得した API キー
- JIRA_BASE_URL: `https://ドメイン名.atlassian.net`
- JIRA_USER_EMAIL: API キー作成者のメールアドレス

---

## 4. GitHub Actions WorkFlow の書き方

基礎的な書き方は省略します。  
今回必要な知識をピックアップして記載します。

### 4-1. WorkFlow の書き方

#### 4-1-1. on > issues

types で issue が hogehoge されたとき、のような指定が出来ます。  
デフォルトブランチでしか動作しません。
複数指定も可能です。

- opened: issue が作成されたとき
- edited: issue が編集されたとき
- milestoned: issue のマイルストーンが設定されたとき
- assigned: issue に開発者がアサインされたとき
- close: issue が閉じられたとき

> 参考: [events-that-trigger-workflows](https://docs.github.com/ja/actions/reference/events-that-trigger-workflows#discussion)

他にも、`issue_comment` や `push`、`pull_request` は Jira の自動化に使える良トリガーです。

#### 4-1-2. github.event.issue

この辺りの属性を使ってissue内容をいい感じに取得します。

- title: issue のタイトル
- body: issue の本文
- html_url: issue の URL

### 4-2. atlassian/gajira の使い方

今回の主役。gajira を利用して jira を操作する。

#### 4-2-1. Login

Jira にログインします。
3-2. で登録した Secrets は `${{ secrets.Hoge }}`で指定可能です。

```yml
- name: Login
      uses: atlassian/gajira-login@master
      env:
        JIRA_BASE_URL: ${{ secrets.JIRA_BASE_URL }}
        JIRA_USER_EMAIL: ${{ secrets.JIRA_USER_EMAIL }}
        JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}
```

#### 4-2-2. Find issue Key

Jira の課題キー(Jira タスクの下あたりにある`HOGE-XXX`)を探して取得します。  
`string`にサーチしたい文字列を与えると探して課題キーを取得します。  
取得した課題キーは`${{ steps.create.outputs.issue }}`で使用できます。

```yml
- name: Find in commit messages
  uses: atlassian/gajira-find-issue-key@master
  with:
    string: ${{ github.event.ref }}
```

#### 4-2-3. Create

Jira の新しい課題を作成します。
作成した課題の課題キーは`${{ steps.create.outputs.issue }}`で使用できます。  
`issuetype`: Task, Story, Epic, Build 等課題のタイプ(日本語も使えるかも？)

```yml
- name: Create
  id: create
  uses: atlassian/gajira-create@master
  with:
    project: プロジェクトの略(ACとかHGとか)
    issuetype: Task
    summary: 書きたいタイトル
    description: 書きたい説明文
    fields: '{"フィールド名": "hoge"}'

- name: Log created issue
  run: echo "Issue ${{ steps.create.outputs.issue }} was created"
```

#### 4-2-4. Transition

Jira のプロジェクトボードのタスクのステータスを変更します。  
`transition`: In progress, Review, Done などの移行先を指定します。(日本語も可みたいです)

```yml
- name: Transition issue
  uses: atlassian/gajira-transition@master
  with:
    issue: HOGE-XXX
    transition: "In progress"
```

> 参考: [gajira のリポジトリ](https://github.com/atlassian/gajira)

### 4-3. WorkFlow を作成する

#### 例. GitHub Issue から Jira の課題を自動作成する

```yml
name: Create Jira Task

on:
  issues:
    types: [opened]

jobs:
  build:
    runs-on: ubuntu-latest
    name: Create Jira Task
    if: github.event.action == 'opened'
    steps:
      - name: Login
        uses: atlassian/gajira-login@master
        env:
          JIRA_BASE_URL: ${{ secrets.JIRA_BASE_URL }}
          JIRA_USER_EMAIL: ${{ secrets.JIRA_USER_EMAIL }}
          JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}

      - name: Create Task
        uses: atlassian/gajira-create@v2.0.1
        with:
          project: プロジェクトキー
          issuetype: Task
          summary: ${{ github.event.issue.title}}
          description: ${{ github.event.issue.body}}

      - name: Echo Jira Issue Key
        run: echo "${{ steps.create.outputs.issue }} was created"
```

---

## 5. おわりに

弊社だと`pre-commit`を採用していて、PrefixにJira側のトリガー入れづらいんですよね。。  
今後、Jira 側からの連携がもう少し融通聞くと嬉しいですね。
