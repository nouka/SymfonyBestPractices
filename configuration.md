# 設定
設定は大抵、アプリケーションの別々の部分（例えばインフラとユーザー権限のような）と別々の環境（開発環境、本番環境）に関係しています。Symfonyはアプリケーションの設定を3つの部分に分ける事をお勧めしています。

## インフラに関係する設定
**Best Practice**
インフラに関係する設定は`app/config/parameters.yml`に定義してください。

デフォルトの`parameters.yml`はこの推奨に従っており、データベースとメールサーバに関連する設定が定義されています。

```
# app/config/parameters.yml
parameters:
    database_driver:   pdo_mysql
    database_host:     127.0.0.1
    database_port:     ~
    database_name:     symfony
    database_user:     root
    database_password: ~

    mailer_transport:  smtp
    mailer_host:       127.0.0.1
    mailer_user:       ~
    mailer_password:   ~

    # ...
```

これらの設定は`app/config/config.yml`ファイルでは定義されていません。というのも、それらはアプリケーションの振る舞いに全く関係がないからです。言い換えると、アプリケーションはデータベースが正しく設定されている限りは、データベースの場所やアクセスするための認証情報に対して関心がないのです。

## 規範的なパラメータ
**Best Practice**
アプリケーションの全てのパラメータは`app/config/parameters.yml.dist`ファイルに定義してください。

Symfonyには`parameters.yml.dist`と呼ばれる設定ファイルが含まれており、アプリケーションのための規範的な設定のリストが保存されています。

アプリケーションに新しい設定を定義したら、このファイルにも追加し変更をバージョン管理システムに送信しましょう。開発者がプロジェクトを更新したときやサーバにデプロイしたとき、Symfonyは`parameters.yml.dist`とローカルの`parameters.yml`の差分を調べます。もしそれらに差分があった場合
、Symfonyは新しいパラメータの値を指定するように要求し、ローカルの`parameters.yml`ファイルに追加します。

## アプリケーションに関連する設定
**Best Practice**
アプリケーションの振る舞いに関係する設定は`app/config/config.yml`ファイルに定義してください。

`config.yml`ファイルにはメール通知の送信先やFeature Toggleのように、アプリケーションの振る舞いを変える設定が含まれています。これらの設定を`parameters.yml`に定義してしまうと、サーバごとに変える必要のない設定までサーバごとに設定しなければならなくなります。 

`config.yml`に定義されている設定は大抵、環境によって異なります。そこでSymfonyには`app/config/config_dev.yml`と`app/config/config_prod.yml`があり、環境ごとに値を上書きできるようになっています。

## 定数 vs 設定値
アプリケーションの設定を定義するときに最もありがちなミスは、ページングの結果の件数のように、決して変わらない値を設定に作成してしまう事です。

**Best Practice**
滅多に変更しないオプションは定数として定義しましょう。

設定を定義する伝統的なアプローチのために、多くのSymfonyアプリケーションに以下のような設定が含まれています。この設定はブログに表示する投稿の数を制御します。

```
# app/config/config.yml
parameters:
    homepage.num_items: 10
```

過去にこのようなことをしていれば、その値の変更は本来必要ではなかったでしょう。変更する必要のない設定を作成する必要はありません。
If you've done something like this in the past, it's likely that you've in fact never actually needed to change that value. Creating a configuration option for a value that you are never going to configure just isn't necessary. 
→カーシャさんに聞く

このような値は定数として定義することをお薦めします。例えば、PostエンティティのNUM_ITEMS定数として定義するのです。

```
// src/AppBundle/Entity/Post.php
namespace AppBundle\Entity;

class Post
{
    const NUM_ITEMS = 10;

    // ...
}
```

定数を定義する主なメリットは、アプリケーション内のどこからでも利用できることです。パラメータを使った場合、Symfonyコンテナがアクセスできる場所でしか利用できません。

定数はconstant()関数のおかげで、Twigテンプレートでも使うことができます。

```
<p>
    Displaying the {{ constant('NUM_ITEMS', post) }} most recent results.
</p>
```

Doctrineのエンティティやrepositoriesはこれらの値に簡単にアクセスできます。一方で、コンテナのパラメータにはアクセスできません。

```
namespace AppBundle\Repository;

use Doctrine\ORM\EntityRepository;
use AppBundle\Entity\Post;

class PostRepository extends EntityRepository
{
    public function findLatest($limit = Post::NUM_ITEMS)
    {
        // ...
    }
}
```

設定に定数を使う唯一の弱点は、テストのときに簡単に値を上書きできない点です。

## セマンティックな設定（やってはいけない）
**Best Practice**
バンドルの設定をセマンティックなDI設定として定義しないでください。

「How to Load Service Configuration inside a Bundle」の記事にもあるように、Symfonyバンドルには2つの設定の方法があります。`service.yml`を通して設定する通常の方法と、特別なExtensionクラスを通して設定するセマンティックな設定です。

セマンティックな設定は強力で、設定のバリデーションのように良い機能を提供しますが、設定を定義するために必要な作業が多いため、サードパーティのバンドルとして共有しない限りは価値がありません。

## センシティブな設定をSymfonyの外に完全に移動する
データベースの接続情報のような機密性の高い設定を扱う場合、それらをSymfonyの外に保存し、環境変数を使って利用可能にすることをお薦めします。
この方法は以下の記事で学ぶ事ができます。
How to Set external Parameters in the Service Container.
