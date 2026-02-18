# Good Code, Bad Code ～持続可能な開発のためのソフトウェアエンジニア的思考～

Kindle

参考になった個所や、後程深堀していきたい内容などをメモ

P26 2.1 疑似コードでのnullの扱い方
null安全: 変数や関数の返す値が`null`である可能性を示し、最初にその値を`null`か否かをチェックしなければ、利用できないようにコンパイラーが強制するというもの。
**TypeScript**
tsconfig.jsonで以下を設定
```typescript
{
  "compilerOptions": {
    // ...他の設定
    "strict": true
  }
}
```

**他の厳格なチェックは入れたくないがNull安全性だけは確保したい場合**
```typescript
{
  "compilerOptions": {
    // ...他の設定
    "strictNullChecks": true
  }
}
```


