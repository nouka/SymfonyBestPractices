# アプリケーションのビジネスロジックを整理する
コンピュータソフトウェアの世界では、データがどのように作られ、表示され、保存され、変更されるかを決める現実世界のビジネスルールを実装したプログラムをビジネスロジックあるいはドメインロジックと呼びます。（完全な定義を読む）

Symfonyのアプリケーションでは、ビジネスロジックは、フレームワーク（例えばルーティングやコントローラのような）に依存せず、自由に書くことができます。サービスとして使われるドメインのクラス、Doctrineのエンティティ、通常のPHPのクラスはビジネスロジックの一例です。

ほとんどのプロジェクトにおいて、開発者は全てをAppBundleに置くべきです。AppBundleの中では、ビジネスロジックを整理するための、どんなディレクトリでも作ることができます。

```
symfony-project/
├─ app/
├─ src/
│  └─ AppBundle/
│     └─ Utils/
│        └─ MyClass.php
├─ tests/
├─ var/
├─ vendor/
└─ web/
```

## クラスをバンドルの外に置きたいですか？
しかし、ビジネスロジックをバンドルの中に入れておく技術的な理由は何もありません。srcディレクトリの下に好きな名前空間を定義して、クラスをそこに置くこともできます。

```
symfony-project/
├─ app/
├─ src/
│  ├─ Acme/
│  │   └─ Utils/
│  │      └─ MyClass.php
│  └─ AppBundle/
├─ tests/
├─ var/
├─ vendor/
└─ web/
```

> AppBundleを使うことをお勧めするのは簡単さのためです。バンドルの中に必要なものとバンドルの外でも問題ないものがよく理解できている場合は、クラスをAppBundleの外に置いても構いません。

## サービス: 命名と形式
ブログアプリケーションに、“Hello World”のような投稿タイトルを“hello-world”のようなスラグに変換するユーティリティが必要だとします。スラグは、投稿のURLの一部として使われます。

新しいSluggerクラスを`src/AppBundle/Utils/`ディレクトリの配下に作り、下記のようなslugify()メソッドを実装してみましょう。

```
// src/AppBundle/Utils/Slugger.php
namespace AppBundle\Utils;

class Slugger
{
    public function slugify($string)
    {
        return preg_replace(
            '/[^a-z0-9]/', '-', strtolower(trim(strip_tags($string)))
        );
    }
}
```

次に、このクラスのための新しいサービスを定義します。

```
# app/config/services.yml
services:
    # keep your service names short
    app.slugger:
        class: AppBundle\Utils\Slugger
```

伝統的に、サービスの名前は、衝突を避けるためにクラス名とクラスの場所を組み合わせたものでした。そうすると、このサービスはapp.utils.sluggerと呼ばれるはずです。しかし、短い名前を使うことで、コードの読みやすさと使いやすさは向上するでしょう。

**Best Practice**
アプリケーションのサービス名は可能な限り短くしましょう。必要があれば一意なものにしてください。

これでAdminControllerのようなどんなコントローラーからでもsluggerを利用できるようになりました。

```
public function createAction(Request $request)
{
    // ...

    if ($form->isSubmitted() && $form->isValid()) {
        $slug = $this->get('app.slugger')->slugify($post->getTitle());
        $post->setSlug($slug);

        // ...
    }
}
```

## サービス定義: YAML
前項では、サービスの定義にはYAMLが使われていました。

**Best Practice**
サービスの定義にはYAMLを使ってください。

これは議論を呼ぶでしょうが、経験上、YAMLとXMLは開発者の中で均等に分布しており、ややYAMLが好まれる傾向にあります。どちらの形式も同じ機能なので、最終的には個人の好みの問題です。

YAMLをお薦めするのは、簡潔で新人にもわかりやすいためです。どちらを使っても構いません。

## サービス、クラスをパラメータにしない
前項でサービスを定義する際、クラスの名前空間をパラメータにしていないことに気づいたかもしれません。

```
# app/config/services.yml

# service definition with class namespace as parameter
parameters:
    slugger.class: AppBundle\Utils\Slugger

services:
    app.slugger:
        class: '%slugger.class%'
```

この方法は面倒で、サービスには全く必要ありません。

**Best Practice**
サービスのクラスをパラメータとして定義しないでください。

この方法はサードパーティのバンドルから間違って採用されたものです。Symfonyがサービスコンテナを導入したとき、開発者の何人かはこのテクニックを使いサービスを上書きしやすくする人もいました。しかしながら、サービス名を変更しただけでサービスを上書きするケースは非常に稀です。なぜならば多くの場合、新しいサービスは異なったコンストラクタ引数を持っているからです。

## 永続化レイヤーを利用する
SymfonyはHTTPのフレームワークで、各HTTPリクエストに対してHTTPレスポンスを生成することのみを受け持ちます。Symfonyが永続化レイヤー（データベース、外部APIなど）との通信方法を提供していないのは、このためです。あなたはどんなライブラリや戦略でも選ぶことができます。

実際には、多くのSymfonyアプリケーションはDoctrineに依存しており、エンティティやリポジトリを使いモデルを定義しています。ビジネスロジックと同じく、DoctrineのエンティティもAppBundle配下に保存することをお薦めします。

一例として、サンプルのブログアプリケーションで定義された3つのエンティティがあります。

```
symfony-project/
├─ ...
└─ src/
   └─ AppBundle/
      └─ Entity/
         ├─ Comment.php
         ├─ Post.php
         └─ User.php
```

> もちろん独自の名前空間で`src/`ディレクトリの配下に保存することもできます。

##  Doctrineのマッピング
Doctrineのエンティティはデータベースに保存することができる、プレーンなPHPオブジェクトです。Doctrineはモデルクラスに対して設定されたメタデータを通してエンティティを扱います。
DoctrineはYAML, XML, PHP, アノテーションの4つのメタデータの形式をサポートします。

**Best Practice**
Doctrineのエンティティのメタデータの定義にはアノテーションを使ってください。

アノテーションが最も便利で最速の方法だからです。

```
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use Doctrine\Common\Collections\ArrayCollection;

/**
 * @ORM\Entity
 */
class Post
{
    const NUM_ITEMS = 10;

    /**
     * @ORM\Id
     * @ORM\GeneratedValue
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string")
     */
    private $title;

    /**
     * @ORM\Column(type="string")
     */
    private $slug;

    /**
     * @ORM\Column(type="text")
     */
    private $content;

    /**
     * @ORM\Column(type="string")
     */
    private $authorEmail;

    /**
     * @ORM\Column(type="datetime")
     */
    private $publishedAt;

    /**
     * @ORM\OneToMany(
     *      targetEntity="Comment",
     *      mappedBy="post",
     *      orphanRemoval=true
     * )
     * @ORM\OrderBy({"publishedAt" = "ASC"})
     */
    private $comments;

    public function __construct()
    {
        $this->publishedAt = new \DateTime();
        $this->comments = new ArrayCollection();
    }

    // getters and setters ...
}
```

どの形式も同じ機能なので、最終的には開発者の好みの問題です。

## データフィクスチャ
Symfonyのデフォルトではフィクスチャは有効ではありません。以下のコマンドを実行して、フィクスチャバンドルをインストールしてください。

```
$ composer require "doctrine/doctrine-fixtures-bundle"
```

Then, enable the bundle in AppKernel.php, but only for the dev and test environments:

```
use Symfony\Component\HttpKernel\Kernel;

class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
            // ...
        );

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            // ...
            $bundles[] = new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle();
        }

        return $bundles;
    }

    // ...
}
```

We recommend creating just one fixture class for simplicity, though you're welcome to have more if that class gets quite large.

Assuming you have at least one fixtures class and that the database access is configured properly, you can load your fixtures by executing the following command:

```
$ php bin/console doctrine:fixtures:load

Careful, database will be purged. Do you want to continue Y/N ? Y
  > purging database
  > loading AppBundle\DataFixtures\ORM\LoadFixtures
```

## Coding Standards
The Symfony source code follows the PSR-1 and PSR-2 coding standards that were defined by the PHP community. You can learn more about the Symfony Coding standards and even use the PHP-CS-Fixer, which is a command-line utility that can fix the coding standards of an entire codebase in a matter of seconds.
