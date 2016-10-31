# プロジェクトの作成
## Symfonyのインストール
以前は、SymfonyプロジェクトはPHPアプリケーションの依存パッケージ管理ツールであるComposerを使って作成されていました。しかし現在は、Symfonyインストーラをプロジェクトを作成する前にインストールし使用することが推奨されています。

**Best Practice**
新しくSymfonyプロジェクトを作成するときは、Symfonyインストーラを使ってください。

Symfonyインストーラの使い方はInstalling & Setting up the Symfony Frameworkを読んでください。

## ブログアプリケーションの作成
ここまで正しく設定できていれば、Symfonyに基づいた新規プロジェクトを作成できます。コマンドコンソールでファイル作成権限があるディレクトリに移動し以下のコマンドを実行してください：

```
$ cd projects/
$ symfony new blog

# Windows
c:\> cd projects/
c:\projects\> php symfony new blog
```

> もしインストーラが動作しなかったり、何も出力されない場合は、Pharエクステンションがインストールされており有効になっていることを確認してください。

このコマンドは入手できるもっとも新しい安定板のSymfonyによる新しいプロジェクトをblogという新規ディレクトリに作成します。
加えて、インストーラはシステムがSymfonyアプリケーションを実行するために必要な技術的要件を満たしているかチェックします。もし満たしていなければ、要件を満たすために必要な変更のリストが表示されます。

> Symfonyはセキュリティの理由でデジタル署名でリリースされます。Symfonyのインストールを全て堅実に確かめたい場合は`public check sums repository`とthese stepsを参照し電子署名を確認してください。

## アプリケーションの構成
アプリケーションを作成した後に、`blog/`ディレクトリに移動するといくつかのファイルとディレクトリが自動的に生成されていることが確認できるでしょう：

```
blog/
├─ app/
│  ├─ config/
│  └─ Resources/
├─ bin
│  └─ console
├─ src/
│  └─ AppBundle/
├─ var/
│  ├─ cache/
│  ├─ logs/
│  └─ sessions/
├─ tests/
│  └─ AppBundle/
├─ vendor/
└─ web/
```

このファイルとディレクトリの階層は、アプリケーションを構築するためにSymfony によって推奨された慣習です。それぞれのディレクトリの推奨された用途は以下の通りです：

 - `app/config/`には、それぞれの環境のために定義された全ての設定を保管します;
 - `app/Resources/`には、アプリケーションのための全てのテンプレートとトランスレイションファイルを保管します;
 - `src/AppBundle/`には、Symfony特有のコード（各コントローラーや各ルート）、ドメインのコード（e.g 各Doctrineクラス）、そして全てのビジネスロジックを保管します;
 - `var/cache/`にはアプリケーションによって生成された全てのキャッシュファイルを保管します;
 - `var/logs/`には、アプリケーションによって生成された全てのログファイルを保管します;
 - `var/sessions/`にはアプリケーションによって生成された全てのセッションファイルを保管します;
 - `tests/AppBundle/`にはアプリケーションの自動テスト(e.g. ユニットテスト)を保管します;
 - `vendor/`には、Composerがインストールしたアプリケーションの依存パッケージを保管します。これらは決して変更するべきではありません;
 - `web/`には、全てのフロントコントローラーファイルとスタイルシートや JavaScript ファイル、そして画像ファイルなど全ての web assets を保管します;

## アプリケーションバンドル
Symfony 2.0がリリースされたときに、多くの開発者は論理的なモジュールにアプリケーションを分割するSymfony 1.xの流儀を自然と採用しました。これが、多くのSymfonyアプリケーションがUserBundle、ProductBundle、InvoiceBundleなどのように論理的な機能に分割したバンドルを使用している理由です。

これはバンドルは独立したソフトウェアの一片であるかのように再利用できるということを意味します。もしUserBundleが現状のままで他のSymfonyアプリケーションで利用できないのであれば、それ自体をバンドルにするべきではありません。さらに加えると、InvoiceBundleがProductBundleに依存している場合、2つの別々のバンドルにアドバンテージはありません。

**Best Practice**
アプリケーションロジックのためにAppBundleというただひとつのバンドルを作りなさい。

ひとつのAppBundleが動作することがプロジェクトを束ねるということはコードがより簡潔になり理解することも容易になるでしょう。

> このアプリケーションバンドルは決して共有されることがないため、プロジェクトで作成したvendorのAppBundleの接頭語は必要ありません。(e.g AcmeAppBundle)

> ベンダーのバンドルで何か（例えばコントローラ）をオーバーライドしているときには新しいバンドルを作成する理由になります。How to Use Bundle Inheritance to Override Parts of a Bundleを参照してください。

大体において、これらのベストプラクティスを採用する Symfony アプリケーションの 典型的なディレクトリ構成は以下のようになります:

```
blog/
├─ app/
│  ├─ config/
│  └─ Resources/
├─ bin/
│  └─ console
├─ src/
│  └─ AppBundle/
├─ tests/
│  └─ AppBundle/
├─ var/
│  ├─ cache/
│  ├─ logs/
   └─ sessions/
├─ vendor/
└─ web/
   ├─ app.php
   └─ app_dev.php
```

> もしSymfonyにAppBundleがあらかじめ生成されていない場合は、次のコマンドで手動で生成することができます:

```
$ php bin/console generate:bundle --namespace=AppBundle --dir=src --format=annotation --no-interaction
```

## ディレクトリ構成の拡張
プロジェクトや基盤構造でSymfonyのデフォルトのディレクトリ構成からいくつかの変更が必要になった場合は、`cache/`、`logs/`、`web/`といったメインのディレクトリの場所を変更することができます。
