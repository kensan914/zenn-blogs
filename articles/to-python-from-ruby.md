---
title: "Ruby公式「Pythonとの違い」から学ぶ、Pythonの「Rubyとの違い」"
emoji: "🐊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "python3", "ruby"]
published: true
publication_name: open8
---

# はじめに
隔週で開催される社内勉強会で、Python の基本的な言語仕様を紹介するために Ruby 公式の「PythonからRubyへ」というドキュメントを用いた回が意外に盛り上がったので記事にします。

https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/

Ruby コミュニティのメンバーによって運営されている Ruby 公式サイトに掲載されているドキュメントで、Python 使いが Ruby と Python の言語仕様の違いを理解するのにぴったりな内容となっています。

:::message
他にも C, C++, Java, Perl, PHP 版が用意されています。
https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/
:::

今回、こちらのドキュメントの「Python との違い」の章を一部「Ruby との違い」に読み替えて、**Ruby 使いに向けた Python との言語仕様の比較**を行いたいと思います。

# Rubyと違って、Pythonは…
> Pythonと違って、Rubyは…
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)


## 文字列は不変です。
> 文字列は可変です。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)

Python では文字列がデフォルトで不変、つまりイミュータブルです。文字列オブジェクトの状態を変更することはできません。

```ruby:Rubyでは文字列がデフォルトでミュータブル.rb
text = "abc"
text[0] = "A"
text
# "Abc"
```

```python:Pythonでは文字列がデフォルトでイミュータブル.py
text = "abc"
text[0] = "A"
# Traceback (most recent call last):
#   File "<stdin>", line 1, in <module>
# TypeError: 'str' object does not support item assignment
```

## 定数がサポートされていません。
> 定数(値が変更されることを期待しない変数)をつくれます。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)

Python では、定数は慣習的な存在で、厳密には言語としてサポートされていないので再代入ができてしまいます。

```ruby:Rubyでは定数に値を再代入すると警告が出る.rb
MAX_COUNT = 10
MAX_COUNT = 0
# (irb):24: warning: already initialized constant MAX_COUNT
# (irb):23: warning: previous definition of MAX_COUNT was here
# => 0
```

```python:Pythonでは定数に値を再代入できる.py
MAX_COUNT = 10
MAX_COUNT = 0
```

PEP8 に記載されている定数の命名規約（大文字で命名しアンダースコアで単語を区切る）は、Ruby と同じです。
https://peps.python.org/pep-0008/#constants

Python3.8以上であれば `typing.Final` 型ヒントが使えるので、静的解析時に型エラーで定数への再代入を防ぐことができます。

```python:typing.Finalで定数型を定義.py
from typing import Final

MAX_COUNT: Final[int] = 10
```

<!-- ## 特に名前付けについての規約がありません。 -->
<!-- > 名前付けについての規約がいくつかあります。 たとえば、クラス名は大文字から始め、変数名は小文字で始めます。 -->
<!-- > [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/) -->


## リストコンテナは、リスト・タプル・集合など複数あります。
> リストコンテナは、配列しかありません。配列は可変です。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)

Python には組み込みコンテナとしてリスト・タプル・集合などがあります（他にも辞書がありますが、Ruby のハッシュと比較するべきなのでここでは省略します）。
リストは基本的に Ruby の配列と同様でミュータブルなコンテナです。
```python
animal_list = ["dog", "cat", "bird"]
animal_list[0] = "lion"
animal_list
# ['lion', 'cat', 'bird']
```
イミュータブルにしたい場合は、タプルを用います。
```python
animal_tuple = ("dog", "cat", "bird")
animal_tuple[0] = "lion"
# Traceback (most recent call last):
#   File "<stdin>", line 1, in <module>
# TypeError: 'tuple' object does not support item assignment
```
重複を許したくない場合は、集合を用います。
```python
animal_set = {"dog", "cat", "bird"}
animal_set.add("dog")
animal_set
# {'dog', 'bird', 'cat'}
```

<!-- > 二重引用符で囲まれた文字列は、エスケープシーケンス(\tなど)や、 式展開(いちいち+で文字列連結すること無しに、 Rubyの式を評価した結果を他の文字列に挿入可能にするしくみ)を解釈します。 一重引用符で囲まれた文字列は、Pythonでいうraw文字列と同じ扱いとなります。 -->
<!-- > [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/) -->


## Python2系とPython3系では完全な互換性がありません。
> クラスに新しいスタイル・古いスタイルといったものはありません (Python3からはこの問題はなくなりました。けれど、Python2との完全な後方互換性はありません)。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)

Python2系は2020年でサポートが終了しています。
例えば、Python2系では `print` が括弧無しで書けたのに対し、Python3系では `print` 関数になり括弧が必要になりました。
```python:Python3系でのprint記法.py
print("Hello World!!")
# Hello World!!
print "Hello World!!"
#   File "<stdin>", line 1
#     print "Hello World!!"
#     ^^^^^^^^^^^^^^^^^^^^^
# SyntaxError: Missing parentheses in call to 'print'. Did you mean print(...)?
```

コード自動変換ツールが公式で提供されています。
https://docs.python.org/ja/3/library/2to3.html


## 属性に直接アクセスできます。
> 属性には直接アクセスできません。 Rubyでは、属性へのアクセスはすべてメソッド経由になります。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)

Ruby では、属性へのアクセスはすべてメソッド経由で行いますが、Python ではオブジェクトの属性に直接ドット(`.`)演算子を使ってアクセスすることができます。

```python:Pythonでは属性に直接アクセスできる.py
class User:
    def __init__(self, name):
        self.name = name

user = User("taro")
user.name
# "taro"
```

これにより、簡潔で直感的なコードを書くことができますが、属性に直接アクセスすることが制限されないので、アンダースコアを用いてアクセス制御をするなど、ある程度考慮をする必要があります（後述）。


## メソッド（関数）呼び出しの括弧は必須です。
> メソッド呼び出しの括弧は基本的にオプションです。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)

```python:Pythonでは関数呼び出しの括弧は必須.py
def add(a, b):
  return a + b

add(1, 2)
# 3
add 1, 2
#   File "<stdin>", line 1
#     add 1, 2
#         ^
# SyntaxError: invalid syntax
```


## メソッドや属性のアクセス制御をアンダースコアの数によって実現しています。
> Pythonでアンダースコアの数によって実現しているアクセス制御は、 public、private、protectedを使って行います。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)

Python では、1つのアンダースコア（`_`）で始まるメソッドや属性は、慣習的に外部から隠蔽するプライベートであることを表します。実際にアクセス制御がされているわけではないので、アクセスすることはできてしまいます。
2つのアンダースコア（`__` 通称ダンダー:double underscore）で始まるメソッドや属性は、[ネームマングリング](https://docs.python.org/ja/3/tutorial/classes.html#private-variables)が適用されるので、厳密なプライベート化が実現できます。

```python:Pythonにおけるメソッドや変数のアクセス制御.py
class User:
    def __init__(self, name):
        self.name = name
        self._name = name
        self.__name = name

user = User("taro")
user.name
# "taro"
user._name
# "taro"
user.__name
# Traceback (most recent call last):
#   File "<stdin>", line 1, in <module>
# AttributeError: 'User' object has no attribute '__name'. Did you mean: '_name'?
```

## Mix-inの代わりに多重継承を使います。
> 多重継承の代わりにMix-inを使います。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)

Python では、Ruby の Mix-in と同様の機能を実現するために、多重継承を利用します。
```python:Pythonにおける多重継承.py
class LoggerMixin:
    def hello(self):
        print(f"Hello!! I'm {self.name}")

class BaseUser:
    def __init__(self, name):
        self.name = name

class User(BaseUser, LoggerMixin):
    pass

user = User("taro")
user.hello()
# Hello!! I'm taro
```

## 組み込みクラスにメソッドを追加したり、書き換えたりできません。
> 組み込みクラスにメソッドを追加したり、書き換えたりできます。 どちらの言語でも任意の時点でクラスを開いたり編集できますが、 Pythonでは組み込みクラスに対してはそれは許可されていないのに対し、 Rubyではその制限はありません。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)


## trueとfalseは、TrueとFalseになります。 また、nilの代わりはNoneになります。
> TrueとFalseは、trueとfalseになります。 また、Noneの代わりはnilになります。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)
```python
is_true = True
is_false = False
is_none = None

if is_true:
    print("This is true.")

if not is_false:
    print("This is not false.")

if is_none is None:
    print("This is None.")
```

## 真か偽かの判定では、FalseとNone以外に0、0.0、""、[]なども偽と評価されます。
> 真か偽かの判定では、falseとnilのみが偽と評価されます。 それ以外の値(0、0.0、""、[]など)はすべて真と評価されます。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)
```python
false = False
none = None
zero = 0
float_zero = 0.0
empty_string = ""
empty_list = []

if not false:
    print("Falseは偽")

if not none:
    print("Noneは偽")

if not zero:
    print("0は偽")

if not float_zero:
    print("0.0は偽")

if not empty_string:
    print("空文字列は偽")

if not empty_list:
    print("空配列は偽")
```

## elsifの代わりにelifを使います。
> elifの代わりにelsifを使います。
> [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/)

```python
x = 10

if x > 10:
    print("xは10より大きい")
elif x < 10:
    print("xは10より小さい")
else:
    print("xは10です")
```



<!-- ## requireの代わりにimportを使います。それ以外の使い方は同じです。 -->
<!-- > importの代わりにrequireを使います。それ以外の使い方は同じです。 -->
<!-- > [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/) -->


<!-- ## (docstring の代わりに)クラスやメソッドの直前に書かれた複数行のコメントは、ドキュメント生成に使われます。 -->
<!-- > (docstring の代わりに)クラスやメソッドの直前に書かれた複数行のコメントは、ドキュメント生成に使われます。 -->
<!-- > [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/) -->


<!-- ## たくさんの省略記法があります。けれど、すぐに手になじむはずです。 それらはRubyをより楽しく生産的に使えるようにするために用意されています。 -->
<!-- > たくさんの省略記法があります。けれど、すぐに手になじむはずです。 それらはRubyをより楽しく生産的に使えるようにするために用意されています。 -->
<!-- > [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/) -->


<!-- ## 一度定義した変数を、(Pythonでいうdelのように)未定義にする方法はありません。 変数をnilで設定すれば、変数に入っていた値をGCできるようにはできますが、 スコープが存在する限り変数自体はシンボルテーブルに残り続けます。 -->
<!-- > 一度定義した変数を、(Pythonでいうdelのように)未定義にする方法はありません。 変数をnilで設定すれば、変数に入っていた値をGCできるようにはできますが、 スコープが存在する限り変数自体はシンボルテーブルに残り続けます。 -->
<!-- > [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/) -->


<!-- ## yieldキーワードの振る舞いは異なります。 Pythonでは、関数呼び出しの外側のスコープへ実行結果を返します。 そのため、外側のコードは処理の再開について責任を負います。 Rubyでは、yieldは最後の引数として渡された別の関数が実行されます。 そして、実行が完了すると処理を再開します。 -->
<!-- > yieldキーワードの振る舞いは異なります。 Pythonでは、関数呼び出しの外側のスコープへ実行結果を返します。 そのため、外側のコードは処理の再開について責任を負います。 Rubyでは、yieldは最後の引数として渡された別の関数が実行されます。 そして、実行が完了すると処理を再開します。 -->
<!-- > [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/) -->


<!-- ## Pythonがサポートしている無名関数はラムダ式のみですが、 Rubyはブロック、Procオブジェクト、ラムダ式といった種類の無名関数があります。 -->
<!-- > Pythonがサポートしている無名関数はラムダ式のみですが、 Rubyはブロック、Procオブジェクト、ラムダ式といった種類の無名関数があります。 -->
<!-- > [引用: PythonからRubyへ](https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-python/) -->
