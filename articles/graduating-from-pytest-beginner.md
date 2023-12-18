---
title: "【今日から使える】pytestを入門した人に送る実践的なTips7選"
emoji: "🦖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pytest", "python", "python3", "test"]
published: true
publication_name: open8
---

# はじめに
Python でテストを書くにあたって [pytest](https://docs.pytest.org/) は主な選択肢になってくると思います。現在では pytest に関する優れた入門記事が数多く存在しています。
今回は、それらの記事を見て入門を終えた方々に向けて、**今日から使える実践的な Tips** を7つご紹介します！
弊社でも実際に活用している Tips も交えてお伝えするので、チームで導入する際に参考にしていただければ幸いです。

:::message
今回紹介する Tips は、pytest を**単体テスト**として用いることを前提としています
:::

## 想定読者
- pytest の基本的な記法、フィクスチャあたりまで入門した方（今回、この辺りの話はしない予定です）
- pytest に入門したけど、実際のコードに落とし込むのにハードルを感じている方
- pytest のチームでの運用方法を知りたい方


# 1. テスト関数名で仕様を表現する
テスト関数名のフォーマットは、チームとして認識をそろえておきたいところです。弊社では、テスト関数にその関数またはメソッドの期待する振る舞い（仕様）を記載するようにしています。そうすることでテストコードにドキュメントとしての役割を持たせることができます。
以下は弊社で実際に採用しているフォーマットになります。

**`test_関数・メソッド名_仕様()`**

```python:test_user.py
def test_validate_バリデーションをパスした場合Trueを返すこと():
    user = User(name="太郎")

    is_valid = user.validate()

    assert is_valid is True
```

pytest がテスト関数として認識できるように prefix は `test_` で固定しています。その次にテスト対象となる関数名やクラスのメソッド名を示します。

`仕様` の部分に関しては、弊社では読みやすさ優先で日本語で書くようにしています。今のところ、特にこれといった問題はなく運用できています。

また、`仕様` には「どのような振る舞いを期待するのか」を明示することを意識しています。例えば、`test_〇〇_正常系` などは仕様が曖昧なので避けるようにPRで指摘し合っています。


# 2. parametrize を用いて様々なパターンのテストデータを1つのテスト関数で実行する
検証したいこと（表現したい仕様）は同じでも異なる複数のテストデータでテストしたい、というケースがあります。その場合、テストデータの数だけテスト関数を定義するとテストファイルが肥大化するので、`pytest.mark.parametrize` デコレータを用いてパラメータテストを行なっています。
https://docs.pytest.org/en/7.4.x/how-to/parametrize.html

```python:test_user.py
@pytest.mark.parametrize(
    ["name"],
    [
        (None,),
        (123,),
        (True,),
    ],
    ids=[
        "Noneが入力された場合",
        "数値が入力された場合",
        "bool値が入力された場合",
    ],
)
def test_validate_不正なnameが入力された場合Falseを返すこと(name):
    user = User(name=name)

    is_valid = user.validate()

    assert is_valid is False
```

`ids` には各パラメータの識別子を割り振ることができます。ここでも「どういったパラメータを入力したか」を自然言語で説明することによって、仕様を表現することができます。

テストを実行すると以下のようなテストログが出力され、1つのテスト関数で3回テストが実行されていることがわかります。（※ 表示形式のオプションを変更しているので、出力結果が異なる場合がございます。これらのオプションに関しては後述します。）

![パラメータテストのログ](/images/graduating-from-pytest-beginner/01.png)

:::message
デフォルトでは `ids` に日本語を指定すると、テストログでエスケープされてしまい期待する表示になりません。以下の設定で日本語で表示されるようになります。

```:pytest.ini
[pytest]
disable_test_id_escaping_and_forfeit_all_rights_to_community_support = True
```

pytest 公式によると、この設定により思わぬ副作用を引き起こす可能性があるとのことなので、ご注意ください。
> Keep in mind however that this might cause unwanted side effects and even bugs depending on the OS used and plugins currently installed, so use it at your own risk.
> [pytest documentation](https://docs.pytest.org/en/7.4.x/how-to/parametrize.html#:~:text=Keep%20in%20mind%20however%20that%20this%20might%20cause%20unwanted%20side%20effects%20and%20even%20bugs%20depending%20on%20the%20OS%20used%20and%20plugins%20currently%20installed%2C%20so%20use%20it%20at%20your%20own%20risk.)
:::

## フィクスチャをパラメータ化してパラメータを使い回す
パラメータすらも共通化して使い回したいケースでは、フィクスチャを使用すると便利です。フィクスチャとしてまとめることで、conftest.py や `scope` 引数を用いたスコープ制御ができるので、共通化の幅が広がります。

```python:test_user.py
@pytest.fixture(
    params=[
        None,
        123,
        True,
    ],
    ids=[
        "Noneが入力された場合",
        "数値が入力された場合",
        "bool値が入力された場合",
    ],
)
def invalid_name(request):
    return request.param

def test_validate_不正なnameが入力された場合Falseを返すこと(invalid_name):
    user = User(name=invalid_name)

    is_valid = user.validate()

    assert is_valid is False
```


# 3. マーカーを使いこなして、テスト関数を拡張する
pytest ではマーカーをデコレータとして指定し、テスト関数を拡張することができます。例えば、先ほどのパラメータで使用した `@pytest.mark.parametrize` は pytest が用意する組み込みマーカーの1つです。
https://docs.pytest.org/en/7.1.x/example/markers.html

## skip マーカーを使用して、テストをスキップする
ローカルで開発中、テストが実装途中であるためテストの実行をスキップさせたい場面があるでしょう。そんな時、組み込みマーカーの `@pytest.mark.skip` デコレータを使用してテストをスキップさせることができます。

```python:test_user.py
@pytest.mark.skip(reason="テストを書いてる途中")
def test_validate_不正なageが入力された場合Falseを返すこと():
    ...
```

他にも組み込みマーカーとして、条件付きでテストをスキップさせる `@pytest.mark.skipif` や、失敗すると分かっているテストに対して使用する `@pytest.mark.xfail` などが用意されているようです。
以下の公式ドキュメントが非常によくまとまっているので、参考にしてみてください。
https://docs.pytest.org/en/7.1.x/how-to/skipping.html

逆に、一部のテストのみ実行したい（それ以外の全てのテストを実行したくない）場合、カスタムマーカーを使うと便利です。
まず、実行したいテストに任意のカスタムマーカーをつけてあげます。

```python:test_user.py
@pytest.mark.todo # 任意のカスタムマーカー（今回は`todo`というカスタムマーカーを定義）
def test_validate_不正なageが入力された場合Falseを返すこと():
    ...
```

テスト実行時に `-m` オプションに定義したカスタムマーカーを指定すると、そのカスタムマーカーがついているテスト関数のみ実行されます。

```sh
$ pytest test_user.py -m todo
```

具体的な活用方法として、修正対象のテスト関数すべてにあらかじめ `todo` マーカーをつけておくと、それ以外のテストは実行されないのでテスト実行時間の短縮になったりします。修正が完了したテスト関数の `todo` マーカーを外していって、最終的に `todo` マーカーを0にする、みたいな開発フローを個人的によくします。

## カスタムマーカーを使用して、フィクスチャの振る舞いを変更する
たまにフィクスチャの振る舞いを呼び出し元で柔軟に変更したいケースに遭遇します。そんな時はカスタムマーカーを使うと便利です。
フィクスチャ内で `request.node.get_closest_marker("<任意のマーカー名>")` で `Mark` オブジェクトが取得できるので、その有無で任意のマーカーがフィクスチャ呼び出し元で指定されているか、が判定できます。それを用いて処理を分岐することで、フィクスチャの振る舞いを呼び出し元で変更することができます。

```python:test_user.py
@pytest.fixture
def db_client(request):
    # `use_inmemorydb`カスタムマーカーが指定されている場合はインメモリDBを使用する
    if request.node.get_closest_marker("use_inmemorydb"):
        _db_client = InMemoryDBClient()
    else:
        _db_client = DBClient()
    yield _db_client
    _db_client.close()


@pytest.mark.use_inmemorydb
def test_save_インスタンスの状態がDBに保存されること(db_client):
    ... # 省略
```

<!-- TODO: ## カスタムマーカー -->

# 4. breakpoint() を記述してデバッグを開始する
テストが失敗したら（コードの誤りを教えてくれたら）、Python 標準のデバッガ [pdb](https://docs.python.org/ja/3/library/pdb.html) を用いてデバッグします。
pdb を起動するには、デバッグしたい箇所に `breakpoint()` を記述して pytest を実行します。
```diff python:test_user.py
def test_validate_バリデーションをパスした場合Trueを返すこと():
    user = User(name="Taro")
+   breakpoint() # Python3.7以前の場合は `import pdb; pdb.set_trace()`
    is_valid = user.validate()

    assert is_valid is True
```
ここでは、個人的に頻繁に使うデバッガコマンドをいくつかご紹介します。
**`l(ist)`**
> 現在のファイルのソースコードを表示します。引数を指定しないと、現在の行の前後 11 行分を表示するか、直前の表示を続行します。引数に . を指定すると、現在の行の前後 11 行分を表示します。
> https://docs.python.org/ja/3/library/pdb.html#pdbcommand-list

**`n(ext)`**
> 現在の関数の次の行に達するか、あるいは関数が返るまで実行を継続します。
> https://docs.python.org/ja/3/library/pdb.html#pdbcommand-next

**`c(ont(inue))`**
> ブレークポイントに出会うまで、実行を継続します。
> https://docs.python.org/ja/3/library/pdb.html#pdbcommand-continue

**`q(uit)`**
> デバッガーを終了します。実行しているプログラムは中断されます。
> https://docs.python.org/ja/3/library/pdb.html#pdbcommand-quit

# 5. オプションを用いて、テストログの出力をカスタマイズする
pytest では数多くのオプションが提供されていますが、ここではテストログの出力をカスタマイズしテスト体験を向上させるようなオプションをいくつか紹介いたします。

個人的にオススメなオプションは以下の3つです。
- `-vv`: デフォルトだと出力エラーが省略される場合があるので、不自由ないくらいに出力されるよう設定しています
- `--tb=short`: デフォルトだとエラートレースが長すぎるので1段階短く設定しています
- `-s`: デフォルトだと print 等で標準出力した内容が表示されないので、表示されるよう設定しています

より詳しく知りたい方は、以下の公式ドキュメントを参考にしてみてください。
https://docs.pytest.org/en/7.1.x/how-to/output.html#managing-pytest-s-output

また、通常これらの毎回実行したいオプションは設定ファイルに記述してしまい、テスト実行時のオプション指定を省略します。これについては次の章で紹介します。

# 6. 最低限の pytest 設定を行い、チーム全体の効率化を図る
pytest の設定は python プロジェクトの設定ファイルに記述して、チームで共通の設定を使用するようにしています。例えば、pytest では以下のような設定ファイルがサポートされています。
- pytest.ini
- pyproject.toml
- tox.ini
- setup.cfg

https://docs.pytest.org/en/stable/reference/customize.html

個人的には、最低限以下の設定をしておけば、テスト実行の効率化を図るという側面においては十分かと思います。これらは一部省略・変更をしていますが、実際に弊社で採用している設定になります。弊社では pyproject.toml を使用しているので、pyproject.toml の例を示します。

```toml:pyproject.toml
[tool.pytest.ini_options]
addopts = "-vv --tb=short -s"
pythonpath = ["src"]
testpaths = ["tests"]
python_files = ["test_*.py"]
disable_test_id_escaping_and_forfeit_all_rights_to_community_support = true
```

前章で紹介したオプション `-vv --tb=short -s` は `addopts` に指定して、テスト実行時にオプションを省略できるようにしています^[[addopts - pytest documentation](https://docs.pytest.org/en/7.1.x/reference/reference.html?highlight=pythonpath#confval-addopts)]。ただ、チーム全体に影響が出るので、必要最低限のオプションのみ記述することが望ましいと思います。

`pythonpath` には、プロダクションコードのディレクトリを指定しています^[[pythonpath - pytest documentation](https://docs.pytest.org/en/7.1.x/reference/reference.html?highlight=pythonpath#confval-pythonpath)]。pytest が import されたモジュールの検索にこの設定値を参照するので、テスト実行時間の短縮が期待できます。

`testpaths` にはテストコードのディレクトリを^[[testpaths - pytest documentation](https://docs.pytest.org/en/7.1.x/reference/reference.html?highlight=pythonpath#confval-testpaths)]、`python_files` にはテストモジュールの命名パターンを指定しています^[[python_files - pytest documentation](https://docs.pytest.org/en/7.1.x/reference/reference.html?highlight=pythonpath#confval-python_files)]。pytest がテストモジュールを検索する際にこの設定値を参照するので、テスト実行時間の短縮が期待できます。


# 7. pytest プラグインを導入して、生産性を爆上げする
pytest プラグインは、パッケージをインストールするだけで pytest の機能を簡単に拡張することができて、そのうえ生産性が大きく向上するのでオススメです。
その種類は非常に多く、公式がリストアップしているものだけで **1007** 件もの pytest プラグインが存在しています（2023/12/15 現在）。
https://docs.pytest.org/en/7.1.x/reference/plugin_list.html

今回は、その中から個人的に入れておきたい pytest プラグインをピックアップしてご紹介します（一部弊社でも使用しています）。

## pytest-mock
Python 標準モックライブラリ [unittest.mock](https://docs.python.org/ja/3/library/unittest.mock.html) の 薄いラッパーである `mocker` フィクスチャを提供します。
https://github.com/pytest-dev/pytest-mock

## pytest-clarity
テストが失敗した時、「期待していた値」と「実際の値」の差分をハイライト表示してくれるプラグインです。デフォルトだと赤文字のみで出力されるので特に大きいディクショナリの場合差分がわかりにくかったりするのですが、ハイライトで直感的に差分がわかるようになるので重宝します。
![pytest-clarityのデモ](/images/graduating-from-pytest-beginner/02.png)

詳しくは、下記の READEME を参照いただければと思うのですが、`-vv` オプションを設定する必要があるのでご注意ください。
https://github.com/darrenburns/pytest-clarity

## pytest-randomly
各テストの実行順序をランダムにしてくれるプラグインです。テストは互いに独立すべきですが、予期しないテスト間の依存を発見するのに役立ちます。
https://github.com/pytest-dev/pytest-randomly

## pytest-freezegun
datetime モジュールをモックして、時間を固定することができます。実行する時間によってテスト結果が変わる flaky なテストに対して有効です。
https://github.com/ktosiek/pytest-freezegun

## faker
こちらは pytest プラグインではありませんがご紹介します。テストで使用するダミーデータを作成するための Python パッケージです。
https://github.com/joke2k/faker

## ipdb
こちらも pytest プラグインではありませんがご紹介します。pdb デバッガについて触れましたが、それを拡張したデバッガで以下の特徴があります。

- シンタックスハイライトがかかる
- タブ補完が効くようになる（下記添付画像）

![タブ補完](/images/graduating-from-pytest-beginner/03.png)

パッケージをインストールし、pytest の実行時に `--pdbcls=IPython.terminal.debugger:Pdb` オプションをつけると利用できるようになります。導入した際は、設定ファイルの `addopts` に記述してしまうのがよいかと思います。詳しくは、下記の READEME をご参照ください。

https://github.com/gotcha/ipdb
