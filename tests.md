# テストテスト
大まかに言うと、テストには2つの種類が存在します。ユニットテストは、特定の関数の入力と出力をテストすることができます。機能テストは、あなたが閲覧するサイト上のページに対して “ブラウザ” を制御し、リンクをクリックしたり、フォームに記入を行ったり、ページ上の特定の物事を確認することができます。

## ユニットテスト
ユニットテストは、Symfonyから独立したクラス内に存在すべき、あなたの “ビジネスロジック” をテストするために使用されます。この理由により、Symfonyは実際のところ、ユニットテストのためにあなたがどのツールを使用するかについて意見を持ちません。しかしながら、最もポピュラーなツールは PhpUnit と PhpSpec です。

## 機能テスト
真に優れた機能テストを作成することは困難であるため、一部の開発者は完全にこれを省略します。機能テストを省略しないでください！いくつかの シンプルな 機能テストを定義することで、いかなる大きなエラーであれ、それらをデプロイしてしまう前にすばやく発見できます。

**Best Practice**
機能テストを定義することは、結局のところあなたのアプリケーションのページが正常にロードにされるかどうかチェックすることです。

機能テストは以下の例のように簡単です:

```
// tests/AppBundle/ApplicationAvailabilityFunctionalTest.php
namespace Tests\AppBundle;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class ApplicationAvailabilityFunctionalTest extends WebTestCase
{
    /**
     * @dataProvider urlProvider
     */
    public function testPageIsSuccessful($url)
    {
        $client = self::createClient();
        $client->request('GET', $url);

        $this->assertTrue($client->getResponse()->isSuccessful());
    }

    public function urlProvider()
    {
        return array(
            array('/'),
            array('/posts'),
            array('/post/fixture-post-1'),
            array('/blog/category/fixture-category'),
            array('/archives'),
            // ...
        );
    }
}
```

このコードは全ての与えられたURLが正しくロードできることをチェックし、それは各HTTPレスポンスのステータスコードが 200 から 299 の間であることを意味します。 これは有用に見えないかもしれませんが、かかる労力がいかに少ないかを考慮すると、あなたのアプリケーションで行う価値があります。

コンピュータソフトウェアにおいて、この種のテストは smoke testing と呼ばれ、 「将来のソフトウェアリリースを退けるほど深刻な、単純な失敗を発見にするための予備的なテスト」 で構成されます。

## 機能テスト内のハードコードされたURL
皆さんのうち一部の方は何故、上記の機能テストがURLジェネレータサービスを使用していないのかを尋ねるかもしれません：

**Best Practice**
機能テストにおいては、URLジェネレータを使用する代わりに、URLのハードコードが使用されます。

テストページのURLを生成するために router サービスを使用した、以下の機能テストについて考えてみましょう。

```
public function testBlogArchives()
{
    $client = self::createClient();
    $url = $client->getContainer()->get('router')->generate('blog_archives');
    $client->request('GET', $url);

    // ...
}
```

これは動作するでしょうが、一つ大きな欠点があります。もし開発者が blog_archives ルートのパスを誤って変更した場合、テストは未だ成功しますが、元の（古い）URLは動作しなくなるでしょう！これは、そのURLに対する全てのブックマークが壊れ、あなたが全ての検索エンジンのページランキングを失うことを意味します。

## JavaScriptの機能テスト
組み込みの機能テストクライアントは素晴らしいですが、それをあなたのページで、任意のJavaScriptの振る舞いをテストするために使用することはできません。もしあなたがこのテストを行う必要がある場合、PHPUnitの内部から、 Mink ライブラリを使用することを検討してください。

もちろん、もしもあなたが大規模なJavaScriptフロントエンドを使用している場合、純粋なJavaScriptベースのテストツールを検討すべきです。

## 機能テストについてさらに学ぶ
Faker と Alice ライブラリを使用して、あなたのテストフィクスチャのため実運用に近いデータを生成する HautelookAliceBundle を使用することを検討してください。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MDkwNTcxNDBdfQ==
-->