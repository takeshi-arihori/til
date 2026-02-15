# Docker上でLaravel開発中のVSCodeのTab SizeとSpaceを2に統一する方法

Docker上でLaravelを使用して開発を行っている際、VSCodeのtabサイズやspaceが自動的に4になる問題に直面しました。本記事では、この問題を解決するための具体的な手順を説明します。

## 課題

- Docker上でLaravelを使用した開発を行っている。
- VSCodeのtabのサイズが自動的に4になる。VSCodeのtab sizeを2に設定しても改善されない。
- Dockerを使用せずに純粋にPHPやTypeScriptの開発環境では、すべてtabおよびspaceは2で統一されている。
- VSCodeのステータスバーには「スペース: 2(タブサイズ: 4)」と表示される。

## プロジェクトディレクトリ構造
```
.
├── src # Laravelプロジェクトのルートディレクトリ
├── infra
│ └── docker
│ ├── mysql
│ │ ├── Dockerfile
│ │ └── my.cnf
│ ├── nginx
│ │ ├── Dockerfile
│ │ └── default.conf
│ └── php
│ ├── Dockerfile
│ ├── php-fpm.d
│ │ └── zzz-www.conf
│ └── php.ini
├── Makefile
└── docker-compose.yml
```


## 解決策

### ステップ1: VSCodeの設定を確認

1. **VSCodeの設定を開く**
    - 「設定」→左側のサイドバーで「テキストエディタ」を選択→「tab size」と検索します。

2. **Tab SizeとInsert Spacesの設定の確認**
    - `Tab Size`を`2`に設定。
    - `Insert Spaces`を`true`に設定。

### ステップ2: `.editorconfig` ファイルの作成

プロジェクトルートに `.editorconfig` ファイルを作成し、以下の内容を追加します。このファイルはプロジェクト全体で一貫したコードスタイルを保証します。

```plaintext
# .editorconfig

# EditorConfig is awesome: http://EditorConfig.org

root = true

[*]
indent_style = space
indent_size = 2

[*.php]
indent_style = space
indent_size = 2

[*.ts]
indent_style = space
indent_size = 2

[*.js]
indent_style = space
indent_size = 2

# 必要に応じて、さらにファイルの種類とその設定を追加することができます。
```
- [Editor Config](https://editorconfig.org/)
  
### ステップ3: VSCodeの拡張機能をインストール
`EditorConfig for VS Code`という拡張機能をインストールします。この拡張機能は、`.editorconfig` ファイルに基づいて設定を適用してくれます。

### ステップ4: プロジェクトの設定ファイルを確認
`settings.json` ファイルに問題がないか確認します。

- プロジェクトルートにある .vscode/settings.json ファイルを開きます。
以下の内容が含まれていることを確認します。

```
{
    "editor.tabSize": 2,
    "editor.insertSpaces": true,
    "[php]": {
        "editor.tabSize": 2,
        "editor.insertSpaces": true
    },
    "[typescript]": {
        "editor.tabSize": 2,
        "editor.insertSpaces": true
    },
    "[javascript]": {
        "editor.tabSize": 2,
        "editor.insertSpaces": true
    }
}

```
