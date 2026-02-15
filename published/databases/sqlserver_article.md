# Docker + SQL Server for LinuxでMac上の環境作成

# SQL Serverとは

SQL ServerはMicrosoft社が開発・提供するリレーショナルデータベース管理システム（RDBMS）です。企業向けのデータベースソリューションとして広く利用されており、Windows環境を中心としたシステム開発で重要な役割を果たしています。

**主な特徴**
- 高いパフォーマンスと可用性
- 豊富な管理ツール（SQL Server Management Studio等）
- .NET Frameworkとの親和性
- ビジネスインテリジェンス機能
- クラウド対応（Azure SQL Database）

近年では、Linux版も提供されており、クロスプラットフォーム対応が進んでいます。

# ストアドプロシージャとは

ストアドプロシージャは、データベースサーバー上に保存される実行可能なSQLプログラムです。複数のSQL文をまとめて一つの処理単位として定義し、必要に応じて呼び出すことができます。

**主なメリット**
- パフォーマンスの向上（コンパイル済み実行）
- セキュリティの強化（SQLインジェクション対策）
- コードの再利用性
- ネットワークトラフィックの削減
- ビジネスロジックの集約

**注意点**
ストアドプロシージャは標準SQL（ANSI SQL）の仕様には含まれておらず、各DBMS固有の拡張機能です。そのため、異なるDBMS間での移植性は限られています。

## ストアドプロシージャの例（SQL Server）

```sql
-- 部署別従業員検索のストアドプロシージャ
CREATE PROCEDURE GetEmployeesByDepartment
    @Department NVARCHAR(50),
    @MinSalary DECIMAL(10,2) = 0
AS
BEGIN
    SET NOCOUNT ON;
    
    SELECT 
        EmployeeID,
        Name,
        Department,
        Salary,
        HireDate
    FROM Employees 
    WHERE Department = @Department 
      AND Salary >= @MinSalary
    ORDER BY Salary DESC;
END;

-- 実行例
EXEC GetEmployeesByDepartment @Department = 'IT', @MinSalary = 50000;
```

## MySQLでの例

```sql
DELIMITER //

CREATE PROCEDURE GetEmployeesByDepartment(
    IN dept_name VARCHAR(50),
    IN min_salary DECIMAL(10,2)
)
BEGIN
    SELECT 
        employee_id,
        name,
        department,
        salary,
        hire_date
    FROM employees 
    WHERE department = dept_name 
      AND salary >= min_salary
    ORDER BY salary DESC;
END //

DELIMITER ;

-- 実行例
CALL GetEmployeesByDepartment('IT', 50000);
```

**MySQLのストアドプロシージャの制限**
MySQLのストアドプロシージャは他のDBMS製品と比べると機能が限定的です。特に以下の点で劣ります：

- **デバッグ機能**: ステップ実行やブレークポイントが利用できない
- **例外処理**: エラーハンドリング機能が基本的
- **データ型**: 豊富なデータ型のサポートが限定的
- **配列・コレクション**: 複雑なデータ構造の操作が困難
- **パッケージ機能**: 関連するプロシージャをグループ化する機能がない

ストアドプロシージャに関しては、**有償であるOracle DBやSQL Serverの方が圧倒的に優れて**います。これらの製品では、PL/SQLやT-SQLといった強力な拡張言語により、複雑なビジネスロジックを効率的に実装できます。

# Mac上での学習方法

MacでSQL Serverのストアドプロシージャを学習する最も効率的な方法は、**SQL Server for Linux on Docker**を利用することです。これにより、Windowsマシンを用意することなく、本格的なSQL Server環境を構築できます。

# ステップ1: SQL Server for Linuxをインストール

まず、DockerでSQL Server 2022 Linux版を起動します。

```bash
# SQL Server 2022 Linux版を起動
docker run -e "ACCEPT_EULA=Y" \
           -e "MSSQL_SA_PASSWORD=YourStrong@Passw0rd" \
           -p 1433:1433 \
           --name sqlserver-practice \
           -d mcr.microsoft.com/mssql/server:2022-latest
```

**パラメータ説明**
- `ACCEPT_EULA=Y`: ライセンス条項に同意
- `MSSQL_SA_PASSWORD`: 管理者パスワードを設定（8文字以上で大文字・小文字・数字・記号を含む）
- `-p 1433:1433`: ポート1433でアクセス可能にする
- `--name`: コンテナ名を指定

## CLIでの操作方法

**コンテナに接続してsqlcmdを使用**

```bash
# コンテナ内のbashに入る
docker exec -it sqlserver-practice bash

# sqlcmdでSQL Serverに接続（SSL証明書エラー回避のため-Cオプション使用）
/opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P 'YourStrong@Passw0rd' -C
```

**sqlcmdの基本操作**

sqlcmdでは、SQL文を入力した後に `GO` を入力して実行します：

```sql
-- データベース一覧表示
1> SELECT name FROM sys.databases;
2> GO

-- バージョン確認
1> SELECT @@VERSION;
2> GO

-- 現在のデータベース確認
1> SELECT DB_NAME();
2> GO

-- 新しいデータベース作成
1> CREATE DATABASE SampleDB;
2> GO

-- データベース切り替え
1> USE SampleDB;
2> GO

-- sqlcmdを終了
1> EXIT
```

**CLIでの注意点**
- SQL Server 2022では新しい `mssql-tools18` が使用される
- SSL証明書の検証エラーを回避するため `-C` オプションが必要
- 各SQL文の後に必ず `GO` を入力して実行
- 複数行にわたるSQL文も入力可能

## ストアドプロシージャ練習例

接続後、以下のサンプルで練習を開始できます。

```sql
-- 1. サンプルテーブル作成
CREATE TABLE Employees (
    EmployeeID INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL,
    Department NVARCHAR(50) NOT NULL,
    Salary DECIMAL(10,2) NOT NULL,
    HireDate DATE NOT NULL
);

-- 2. サンプルデータ挿入
INSERT INTO Employees (Name, Department, Salary, HireDate) VALUES
('田中太郎', 'IT', 60000, '2020-04-01'),
('佐藤花子', 'IT', 65000, '2019-07-15'),
('鈴木一郎', '営業', 55000, '2021-01-10'),
('高橋美咲', 'IT', 70000, '2018-09-01');

-- 3. 基本的なストアドプロシージャ
CREATE PROCEDURE GetAllEmployees
AS
BEGIN
    SELECT * FROM Employees ORDER BY Salary DESC;
END;

-- 4. パラメータ付きストアドプロシージャ
CREATE PROCEDURE GetEmployeesByDepartment
    @Department NVARCHAR(50)
AS
BEGIN
    SELECT * FROM Employees 
    WHERE Department = @Department
    ORDER BY HireDate;
END;

-- 5. 複数パラメータとデフォルト値
CREATE PROCEDURE GetEmployeesBySalaryRange
    @MinSalary DECIMAL(10,2) = 0,
    @MaxSalary DECIMAL(10,2) = 999999,
    @Department NVARCHAR(50) = NULL
AS
BEGIN
    SELECT * FROM Employees 
    WHERE Salary BETWEEN @MinSalary AND @MaxSalary
      AND (@Department IS NULL OR Department = @Department)
    ORDER BY Salary;
END;

-- 実行例
EXEC GetAllEmployees;
EXEC GetEmployeesByDepartment @Department = 'IT';
EXEC GetEmployeesBySalaryRange @MinSalary = 60000, @Department = 'IT';
```

# DBeaver のインストール

DBeaver は無料のマルチプラットフォーム対応データベース管理ツールです。SQL Server にも対応しており、Mac での開発に適しています。

## インストール方法

**方法1: 公式サイトからダウンロード**
1. https://dbeaver.io/ にアクセス
2. 「Download」をクリック
3. macOS版をダウンロード
4. dmgファイルを開いてインストール

**方法2: Homebrewを使用**
```bash
brew install --cask dbeaver-community
```

## SQL Server への接続設定

1. DBeaver を起動
2. 「新しい接続」をクリック
3. 「SQL Server」を選択
4. 接続情報を入力：
   - **サーバー**: `localhost`
   - **ポート**: `1433`
   - **データベース**: `master`（初期接続時）
   - **ユーザー名**: `sa`
   - **パスワード**: `YourStrong@Passw0rd`

5. 「接続テスト」で確認後、「完了」

## 代替ツール: Azure Data Studio

Microsoft公式の無料ツールで、SQL Server に特化しています。

```bash
# Homebrewでインストール
brew install --cask azure-data-studio
```

# 学習リソース

**公式リソース**
- [Microsoft Learn](https://learn.microsoft.com/ja-jp/sql/) - 無料のオンライン学習
- [Microsoft Docs](https://docs.microsoft.com/ja-jp/sql/) - 公式ドキュメント

**その他の学習サイト**
- W3Schools SQL Server Tutorial
- SQLServerTutorial.net
- Qiita の SQL Server 関連記事

**推奨学習順序**
1. 基本構文（CREATE PROCEDURE）
2. パラメータの使い方
3. 条件分岐（IF-ELSE）
4. ループ処理（WHILE）
5. エラーハンドリング（TRY-CATCH）
6. パフォーマンス最適化

# まとめ

Mac環境でもDocker を使うことで、本格的なSQL Server 環境を簡単に構築できます。ストアドプロシージャの学習は、データベース開発スキルの向上に大きく貢献するため、実際に手を動かしながら学習することをお勧めします。

この環境があれば、企業でよく使われるSQL Server の機能を十分に学習できるでしょう。
