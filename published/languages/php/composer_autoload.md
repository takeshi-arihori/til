# PHPでComposerを使ったオートローダーの設定と実行方法
この記事では、PHPでComposerを使ってオートローダーを設定し、クラスを自動的にロードする方法を解説します。この記事を通して、クラスファイルの手動読み込みの手間を省き、コードをより整理された状態で管理できるようになります。

## 環境
```
OS: Ubuntu 24.04
仮想環境: VirtualBox 7.0
PHP: 8.3
Composer: 2.7.1
```

## ステップ 1: Composerのインストール
まず、Composerがインストールされていることを確認します。インストールされていない場合は、以下のコマンドを実行してインストールします：

```
sudo apt update
sudo apt install composer
```

## ステップ 2: プロジェクトのセットアップ
プロジェクトのディレクトリ構造を以下のように整理します：

```
project_root/
│
├── src/
│   ├── Interfaces/
│   │   └── Engine.php
│   ├── Engines/
│   │   ├── GasolineEngine.php
│   │   └── ElectricEngine.php
│   └── Cars/
│       ├── Car.php
│       ├── GasCar.php
│       └── ElectricCar.php
│
├── composer.json
└── main.php
```

## ステップ 3: composer.jsonの作成
プロジェクトのルートディレクトリで以下のコマンドを実行してComposerを初期化します：

```
composer init
```

質問に従って適切に回答し、作成されたcomposer.jsonに以下のautoloadセクションを追加します：

```
{
    "autoload": {
        "psr-4": {
            "": "src/"
        }
    }
}
```

## ステップ 4: クラスファイルの作成
各ファイルを以下のように作成します：

`src/Interfaces/Engine.php`

```
<?php

namespace Interfaces;

interface Engine {
    public function start(): string;
}
src/Engines/GasolineEngine.php
php
コードをコピーする
<?php

namespace Engines;

use Interfaces\Engine;

class GasolineEngine implements Engine {
    public function start(): string {
        return "Starting the gasoline engine...";
    }
}
```

`src/Engines/ElectricEngine.php`

```
<?php

namespace Engines;

use Interfaces\Engine;

class ElectricEngine implements Engine {
    public function start(): string {
        return "Starting the electric engine...";
    }
}
```


`src/Cars/Car.php`

```
<?php

namespace Cars;

use Interfaces\Engine;

abstract class Car {
    protected string $make;
    protected Engine $engine;

    public function __construct(string $make, Engine $engine) {
        $this->make = $make;
        $this->engine = $engine;
    }

    abstract public function drive(): string;

    public function start(): string {
        return $this->engine->start();
    }
}
```

`src/Cars/GasCar.php`

```
<?php

namespace Cars;

use Engines\GasolineEngine;

class GasCar extends Car {
    public function __construct(string $make) {
        parent::__construct($make, new GasolineEngine());
    }

    public function drive(): string {
        return "Driving the gas car...";
    }
}
```


`src/Cars/ElectricCar.php`

```
<?php

namespace Cars;

use Engines\ElectricEngine;

class ElectricCar extends Car {
    public function __construct(string $make) {
        parent::__construct($make, new ElectricEngine());
    }

    public function drive(): string {
        return "Driving the electric car...";
    }
}
```

## ステップ 5: オートローダーの生成
プロジェクトのルートディレクトリで以下のコマンドを実行してオートローダーを生成します：

```
composer dump-autoload
```

## ステップ 6: main.phpの作成
プロジェクトのルートディレクトリにmain.phpを以下のように作成します：

```
<?php

require 'vendor/autoload.php';

use Cars\GasCar;
use Cars\ElectricCar;

$gasCar = new GasCar('Toyota');
$electricCar = new ElectricCar('Tesla');

echo $gasCar->drive() . PHP_EOL;
echo $gasCar->start() . PHP_EOL;

echo $electricCar->drive() . PHP_EOL;
echo $electricCar->start() . PHP_EOL;
```

## ステップ 7: コードの実行
ターミナルでプロジェクトのルートディレクトリに移動し、以下のコマンドを実行してmain.phpを実行します：

```
php main.php
```

これで、main.phpが実行され、クラスが正しくオートロードされて動作するはずです。エラーメッセージが表示された場合は、エラーメッセージを確認して、問題を特定します。

## まとめ
この方法を使うことで、クラスファイルの手動読み込みの手間を省き、コードをより整理された状態で管理することができます。
