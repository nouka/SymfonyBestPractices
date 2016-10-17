# テスト
Roughly speaking, there are two types of test. Unit testing allows you to test the input and output of specific functions. Functional testing allows you to command a "browser" where you browse to pages on your site, click links, fill out forms and assert that you see certain things on the page.

大まかに言うと、テストには2つの種類があります。ユニットテストは特定の関数の入力および出力をテストすることができます。機能テストはあなたのサイトのページをブラウザを制御し、リンクをクリックしたり、フォームに必要事項を記入したり、ページ上の特定の物事を確認できます。

## ユニットテスト
Unit tests are used to test your "business logic", which should live in classes that are independent of Symfony. For that reason, Symfony doesn't really have an opinion on what tools you use for unit testing. However, the most popular tools are PhpUnit and PhpSpec.

ユニットテストはSymfonyから独立しているクラスにあるべき、あなたの「ビジネスロジック」をテストするために使用されます。そのため、Symfonyはユニットテストのためにあなたが使用するツールについての意見を持ちません。 しかしながら、最も人気のあるツールはPhpUnitとPhpSpecです。

## 機能テスト
Creating really good functional tests can be tough so some developers skip these completely. Don't skip the functional tests! By defining some simple functional tests, you can quickly spot any big errors before you deploy them:

本当に良い機能テストを作成することは困難なため、一部の開発者は完全にこれらを省略します。機能テストを省略しないでください！いくつかのシンプルな機能テストを定義することによって、あなたはいかなる大きなエラーであってもデプロイ前に素早く見つけることができます。

**Best Practice**
Hardcode the URLs used in the functional tests instead of using the URL generator.

Consider the following functional test that uses the router service to generate the URL of the tested page:

```
public function testBlogArchives()
{
    $client = self::createClient();
    $url = $client->getContainer()->get('router')->generate('blog_archives');
    $client->request('GET', $url);

    // ...
}
```

This will work, but it has one huge drawback. If a developer mistakenly changes the path of the blog_archives route, the test will still pass, but the original (old) URL won't work! This means that any bookmarks for that URL will be broken and you'll lose any search engine page ranking.

## Testing JavaScript Functionality
The built-in functional testing client is great, but it can't be used to test any JavaScript behavior on your pages. If you need to test this, consider using the Mink library from within PHPUnit.

Of course, if you have a heavy JavaScript frontend, you should consider using pure JavaScript-based testing tools.

## Learn More about Functional Tests
Consider using the HautelookAliceBundle to generate real-looking data for your test fixtures using Faker and Alice.
