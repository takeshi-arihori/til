# spl_autoload_registerの設定時エラーと解決手順

エラー内容と解決方法
1. どんなエラーが起こるか
初期設定の `spl_autoload_register()` では、クラスファイルの自動読み込みが正しく機能せず、クラスが見つからないというエラーが発生しました。

- 自動ロードスクリプトを定義
```php
spl_autoload_extensions(".php");
spl_autoload_register();
```

- エラー内容
```
PHP Fatal error:  Uncaught Error: Class "Helpers\Settings" not found in /path/to/your/project/Database/MySQLWrapper.php:17
```

このエラーは、PHPが必要なクラスファイルを正しく見つけられなかったために発生します。

2. どうやって解決したか
`spl_autoload_register()` にカスタムコールバック関数を渡すことで、未定義のクラス名が渡された際に、そのクラスファイルを手動で読み込むように設定しました。

### spl_autoload_register に引数を渡すことで解決
```php
spl_autoload_register(function($name) {
    // __DIR__は、現在のファイルの絶対ディレクトリパスを取得します。
    $filepath = __DIR__ . "/" . str_replace('\\', '/', $name) . ".php";
    echo "\nRequiring...." . $name . " once ($filepath).\n";
    // バックスラッシュ(\)をフロントスラッシュ(/)に置き換えます。フロントスラッシュはLinuxのファイルパスで使用されます。
    require_once $filepath;
});
```
