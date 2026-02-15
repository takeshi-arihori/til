# DockerでNext.jsの環境構築をする

1. directoryの作成
```
mkdir <directory名>
cd <directory名>
```

2. Dockerfileの作成
```
ARG NODE_VERSION=20.12.0
FROM node:${NODE_VERSION}-alpine
WORKDIR /app
```

3. compose.ymlの作成
```
services:
  app:
    build:
      context: .
    tty: true
    volumes:
      - ./src:/app
    # Next.jsでホットリロードを有効にするためにWATCHPACK_POLLINGをtrueに設定
    environment:
      - WATCHPACK_POLLING=true
    command: sh -c "npm run dev"
    ports:
      - "3000:3000"
```

4. Docker コンテナを使用して Next.js アプリケーションを TypeScript で作成
```
docker compose build
docker compose run --rm app sh -c 'npx create-next-app src --typescript'
docker compose up
```

5. eslint の設定
```
docker compose run --rm app npm i --save-dev eslint eslint-config-next
```

- `.eslintrc.json`（コメントを含まない有効なJSON）
```json
{
  "extends": [
    "next/core-web-vitals",
    "prettier"
  ],
  "rules": {
    "react/react-in-jsx-scope": "off"
  }
}
```

6. prettier の設定

```
docker compose run --rm app npm i --save-dev prettier eslint-config-prettier
```
