# Laravel + SQL Server で並列テスト時に発生するデッドロック対策

Laravel で SQL Server を使い、`php artisan test --parallel`（ParaTest）を実行すると、まれにデッドロック起因のフレークが発生します。  
本記事では、発生理由と `TestCase` 側での実践的な回避策をまとめます。

## 前提

- Framework: Laravel 12.x
- Database: SQL Server（`sqlsrv`）
- Testing: PHPUnit / ParaTest（並列実行）

## 何が起きるか

並列テスト中に、`assertDatabaseHas` や `assertDatabaseMissing` の検証クエリが他トランザクションと競合し、次のようなデッドロック系エラーが発生することがあります。

- SQL Server エラーコード `1205`（deadlock victim）
- SQLSTATE `40001`（serialization/deadlock 系）
- エラーメッセージに `deadlock` を含む

## なぜ起きるか

SQL Server はロックベースで整合性を保つため、並列実行時に読み取りと更新がぶつかると待機が連鎖し、タイミング次第でデッドロックになります。  
特に DB アサーションはテスト全体で実行回数が多く、競合のトリガーになりやすいポイントです。

## 対策: DB アサーションを「デッドロック時のみ」リトライする

`Tests\TestCase` にラッパーを用意し、`QueryException` のうちデッドロック判定に該当する場合だけ短時間待って再試行します。

```php
<?php

namespace Tests;

use Illuminate\Database\QueryException;
use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    protected function assertDatabaseHasWithRetry(
        string $table,
        array $data,
        int $attempts = 5,
        int $sleepMs = 100
    ): void {
        for ($i = 0; $i < $attempts; $i++) {
            try {
                $this->assertDatabaseHas($table, $data);
                return;
            } catch (QueryException $e) {
                if (! $this->isSqlServerDeadlock($e) || $i === $attempts - 1) {
                    throw $e;
                }

                usleep($sleepMs * 1000);
            }
        }
    }

    protected function assertDatabaseMissingWithRetry(
        string $table,
        array $data,
        int $attempts = 5,
        int $sleepMs = 100
    ): void {
        for ($i = 0; $i < $attempts; $i++) {
            try {
                $this->assertDatabaseMissing($table, $data);
                return;
            } catch (QueryException $e) {
                if (! $this->isSqlServerDeadlock($e) || $i === $attempts - 1) {
                    throw $e;
                }

                usleep($sleepMs * 1000);
            }
        }
    }

    private function isSqlServerDeadlock(QueryException $e): bool
    {
        $message = strtolower($e->getMessage());

        return str_contains($message, 'was deadlocked on lock resources')
            || str_contains($message, 'deadlock')
            || (string) $e->getCode() === '40001';
    }
}
```

## 運用上のポイント

1. リトライ対象はデッドロックだけに限定する。  
   制約違反や構文エラーまで握りつぶすと、原因調査が難しくなります。
2. 回数は少なめ（例: 3-5 回）にする。  
   フレーク吸収とテスト時間のバランスを取りやすくなります。
3. 乱用しない。  
   基本は競合を起こしにくいテストデータ設計を優先し、リトライは補助策として使います。

## まとめ

SQL Server 環境での並列テストでは、デッドロック由来の一時失敗は避けにくい場面があります。  
`assertDatabaseHasWithRetry` / `assertDatabaseMissingWithRetry` のような限定的リトライを入れると、CI の安定性を実用的に改善できます。
