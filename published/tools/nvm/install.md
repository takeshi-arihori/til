## Ubuntu に nvm を使って Node.js と npm をインストールする手順
### Step 1: cURL のインストール
cURL がインストールされていない場合、以下のコマンドでインストールします。

```bash
sudo apt install curl
```

### Step 2: 既存の Node.js と npm の削除
既に古いバージョンの Node.js と npm がインストールされている場合、以下のコマンドで削除します。

```bash
sudo apt purge nodejs npm
```

### Step 3: NVM のインストール
以下のコマンドを実行して、nvm をインストールします。

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

インストールが完了したら、以下のコマンドを実行して nvm を使用可能にします。

```bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")" 
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

### Step 4: NVM のインストール確認
nvm が正しくインストールされたことを確認するため、以下のコマンドを実行します。

```bash
nvm -v
```

### Step 5: 最新の安定版 Node.js のインストール
以下のコマンドで最新の安定版（LTS）バージョンの Node.js をインストールします。

```bash
nvm install --lts
```

### Step 6: Node.js のインストール確認
Node.js が正しくインストールされたか確認するため、以下のコマンドを実行します。

```bash
node -v
```
バージョンが v20.12 以上であることを確認してください。

Step 7: npm のインストール確認
npm が正しくインストールされたか確認するため、以下のコマンドを実行します。

```bash
npm -v
```

バージョンが 10.5 以上であることを確認してください。

