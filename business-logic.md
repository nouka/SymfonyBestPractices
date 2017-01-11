# ビジネスロジックを整理する
コンピュータソフトウェアでは、データがどのように生成、表示、保存、変更されるかを決定する現実世界のビジネスルールを実装したプログラムのことを、ビジネスロジックまたはドメインロジックと言います（完全な定義を読む）。

Symfonyアプリケーションでは、ビジネスロジックはフレームワーク（例えば、ルーティングやコントローラのような）に依存せず自由に書けます。サービスとして利用されるドメインのクラス、Doctrineのエンティティ、通常のPHPのクラスはビジネスロジックの良い例です。

多くのプロジェクトでは、AppBundleの中にすべてを保存するべきです。そこではあなたが整理したいことのために、どんなディレクトリでも作成できます。

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

## クラスをバンドルの外に置きたい
しかし、ビジネスロジックをバンドルの中に置く技術的な理由はありません。もしそちらの方が良ければ、好きな名前空間にsrcディレクトリを作成し、そこに置くこともできます。

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

> シンプルさのためにAppBundleディレクトリを使うことをお薦めします。もしあなたがバンドルの中に置くべきものと、そうでないものを充分に知っているならば、自由にしてください。

## サービス、命名と形式
ブログアプリケーションは投稿のタイトル（Hello World）をスラグ（hello-world）に変換するユーティリティが必要です。スラグは投稿のURLに利用されます。

Sluggerクラスを`src/AppBundle/Utils/`に作ってみましょう。そして以下のようなslugifyメソッドを追加しましょう。

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

次に、クラスに新しいサービスを定義します。

```
# app/config/services.yml
services:
    # keep your service names short
    app.slugger:
        class: AppBundle\Utils\Slugger
```

Traditionally, the naming convention for a service involved following the class name and location to avoid name collisions. Thus, the service would have been called app.utils.slugger. But by using short service names, your code will be easier to read and use.

**Best Practice**
The name of your application's services should be as short as possible, but unique enough that you can search your project for the service if you ever need to.

Now you can use the custom slugger in any controller class, such as the AdminController:

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

## Service Format: YAML
In the previous section, YAML was used to define the service.

**Best Practice**
Use the YAML format to define your own services.

This is controversial, and in our experience, YAML and XML usage is evenly distributed among developers, with a slight preference towards YAML. Both formats have the same performance, so this is ultimately a matter of personal taste.

We recommend YAML because it's friendly to newcomers and concise. You can of course use whatever format you like.

## Service: No Class Parameter
You may have noticed that the previous service definition doesn't configure the class namespace as a parameter:

```
# app/config/services.yml

# service definition with class namespace as parameter
parameters:
    slugger.class: AppBundle\Utils\Slugger

services:
    app.slugger:
        class: '%slugger.class%'
```

This practice is cumbersome and completely unnecessary for your own services.

**Best Practice**
Don't define parameters for the classes of your services.

This practice was wrongly adopted from third-party bundles. When Symfony introduced its service container, some developers used this technique to easily allow overriding services. However, overriding a service by just changing its class name is a very rare use case because, frequently, the new service has different constructor arguments.

## Using a Persistence Layer
Symfony is an HTTP framework that only cares about generating an HTTP response for each HTTP request. That's why Symfony doesn't provide a way to talk to a persistence layer (e.g. database, external API). You can choose whatever library or strategy you want for this.

In practice, many Symfony applications rely on the independent Doctrine project to define their model using entities and repositories. Just like with business logic, we recommend storing Doctrine entities in the AppBundle.

The three entities defined by our sample blog application are a good example:

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

> If you're more advanced, you can of course store them under your own namespace in `src/`.

## Doctrine Mapping Information
Doctrine entities are plain PHP objects that you store in some "database". Doctrine only knows about your entities through the mapping metadata configured for your model classes. Doctrine supports four metadata formats: YAML, XML, PHP and annotations.

**Best Practice**
Use annotations to define the mapping information of the Doctrine entities.

Annotations are by far the most convenient and agile way of setting up and looking for mapping information:

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

All formats have the same performance, so this is once again ultimately a matter of taste.

## Data Fixtures
As fixtures support is not enabled by default in Symfony, you should execute the following command to install the Doctrine fixtures bundle:

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
