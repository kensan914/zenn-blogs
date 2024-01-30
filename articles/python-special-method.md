---
title: "【Python】__〇〇__ メソッドの正体"
emoji: "👓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "python3"]
published: true
publication_name: open8
---

# `__〇〇__`メソッドの正体 is「特殊メソッド」
Python で開発する際、`__init__` をはじめとする `__` (アンダースコア*2) で囲まれたメソッドを目にする機会があると思います。
これらのメソッドは「特殊メソッド」と呼ばれ、Python では以下のように定義されています。
> [**special method**](https://docs.python.org/ja/3/glossary.html#term-special-method)
(特殊メソッド) ある型に特定の操作、例えば加算をするために Python から暗黙に呼び出されるメソッド。この種類のメソッドは、メソッド名の最初と最後にアンダースコア2つがついています。

つまり、独自で定義したクラス内でこれら特殊メソッドをオーバーライドすることで、特定の操作の振る舞いを変更できるようになります。特殊メソッドはそれを直接呼び出すというよりは、既存の演算子などの振る舞いを変更するというイメージです。
例えば、インスタンス同士の加算処理を実装したい時、新たに `add` メソッドを定義してそれを呼び出すようにするより、特殊メソッド `__add__` を定義し `+` 演算子を使うようにした方が [Pythonic](https://docs.python.org/ja/3/glossary.html#term-Pythonic) な書き方になり Python コードとしての可読性がグッと上がると思います。

今回は、個人的によく見る（またはよく使う）特殊メソッドをいくつかご紹介します。

# インスタンスのライフサイクル系
## `__init__`
> `object.__init__(self[, ...])`
インスタンスが (`__new__()` によって) 生成された後、それが呼び出し元に返される前に呼び出されます。
[公式Doc](https://docs.python.org/ja/3/reference/datamodel.html#object.__init__)

コンストラクタとして利用頻度は高いと思います。インスタンスの初期化処理（属性の初期化など）が主で、**インスタンスが生成された後に実行される**点が後述の`__new__`との違いです。
返り値は常に `None` を返します。

## `__new__`
> `object.__new__(cls[, ...])`
> クラス cls の新しいインスタンスを作るために呼び出されます。
[公式Doc](https://docs.python.org/ja/3/reference/datamodel.html#object.__new__)

`__init__` と違って**インスタンスが生成される前に実行され**インスタンスを生成する責務があります。そのため、インスタンスメソッドではなく静的メソッドですが、明示的に `@staticmethod` を指定する必要はありません。
返り値は、作成したインスタンス（通常は引数 `cls` のインスタンス）を返す必要があります。

`__new__` の使い所としては、**イミュータブル型（int, str, tupleなど）を継承したサブクラス**の初期化に使うことが多いようです。これらはイミュータブル故にインスタンス化した後の`__init__`で属性を変更することができないためです。

また、シングルトンパターンの実装にも使われます。
https://qiita.com/ttsubo/items/c4af71ceba15b5b213f8

## `__del__`
> `object.__del__(self)`
> インスタンスが破棄されるときに呼び出されます。 これはファイナライザや (適切ではありませんが) デストラクタとも呼ばれます。
[公式Doc](https://docs.python.org/ja/3/reference/datamodel.html#object.__del__)

`del` などでインスタンスが破棄されたタイミングで呼び出されます。あまり目立った使い所はなさそうですが、デバッグ時に GC のタイミングを把握する場面なんかに活用できそうです。

# データ型キャスト系
## `__int__`
組み込み関数 `int()` から呼び出される特殊メソッドです。`int` 型の値を返す必要があります。
関連で他にも `__float__`(浮動小数点) や `__complex__`(複素数) も用意されています。

```python
class Money:
    def __init__(self, amount: int, currency_code: Literal["JPY", "USD"]) -> None:
        self.amount = amount
        self.currency_code = currency_code

    def __int__(self) -> int:
        return self.amount

money = Money(amount=100, currency_code="JPY")
print(int(money))
# 100
```

## `__str__`
> `object.__str__(self)`
> オブジェクトの「非公式の (informal)」あるいは表示に適した文字列表現を計算するために、 str(object) と組み込み関数 format(), print() によって呼ばれます。
[公式Doc](https://docs.python.org/ja/3/reference/datamodel.html#object.__str__)

逆に「公式の (official)」文字列を表す `__repr__` 特殊メソッドも存在します。詳細な説明はここでは行いませんが、エンドユーザが読みやすい文字列を表すのが `__str__` で、開発者に有益な情報を含む文字列を表すのが `__repr__` です。
```python
from datetime import date

today = date.today()
print(repr(today))
# 'datetime.date(2024, 1, 26)'
print(str(today))
# '2024-01-26'
```

先ほどの `Money` クラスに新たに `__str__` を実装してみます。

```python
class Money:
    # __init__, __int__ 省略

    def __str__(self) -> str:
        return "{} ({})".format(self.amount, self.currency_code)

money = Money(amount=100, currency_code="JPY")
print(str(money))
# 100 (JPY)
```


# 比較演算系
## `__eq__`, `__ne__`
> `object.__eq__(self, other)`
> `object.__ne__(self, other)`
> `x==y` は `x.__eq__(y)` を呼び出します; `x!=y` は `x.__ne__(y)` を呼び出します;
[公式Doc](https://docs.python.org/ja/3/reference/datamodel.html#object.__eq__)

`__ne__` は内部で `__eq__` の否定を返してくれるので、基本的には `__eq__` のみ定義すれば問題ないです。

```python
class Money:
    # __init__, __int__, __str__ 省略

    def __eq__(self, other: 'Money') -> bool:
        if not isinstance(other, Money):
            return NotImplemented
        self_jpy_amount = self._convert_to_jpy()
        other_jpy_amount = other._convert_to_jpy()
        return self_jpy_amount == other_jpy_amount

    def _convert_to_jpy(self) -> int:
        if self.currency_code == "USD":
            # NOTE: 仮に 1$ = 100円 とする
            return self.amount * 100
        return self.amount

money1 = Money(amount=100, currency_code="JPY")

money2 = Money(amount=100, currency_code="JPY")
print(money1 == money2, money1 != money2)
# True False
money3 = Money(amount=200, currency_code="JPY")
print(money1 == money3, money1 != money3)
# False True
money4 = Money(amount=100, currency_code="USD")
print(money1 == money4, money1 != money4)
# False True
```

## `__lt__`, `__gt__`
> `object.__lt__(self, other)`
> `object.__gt__(self, other)`
> `x<y` は `x.__lt__(y)` を呼び出します; `x>y` は `x.__gt__(y)` を呼び出します;
[公式Doc](https://docs.python.org/ja/3/reference/datamodel.html#object.__lt__)

<!-- 前述した `__eq__`, `__ne__` と違って、これらの比較演算子は暗黙的に実装されることはないので本来は4つの特殊メソッド全てを定義してあげる必要があります。 -->

`__eq__`, `__ne__` 同様、`__lt__` `__gt__` もどちらか一方を定義すれば問題ありません。一方が定義されていれば、`self` と `other` を入れ替えて実行すればもう一方の結果が得られるためです。

```python
class Money:
    # __init__, __int__, __str__, __eq__ 省略

    def __lt__(self, other: 'Money') -> bool:
        if not isinstance(other, Money):
            return NotImplemented
        self_jpy_amount = self._convert_to_jpy()
        other_jpy_amount = other._convert_to_jpy()
        return self_jpy_amount < other_jpy_amount

money1 = Money(amount=100, currency_code="JPY")

money2 = Money(amount=101, currency_code="JPY")
print(money1 > money2, money1 < money2)
# False True
money3 = Money(amount=99, currency_code="JPY")
print(money1 > money3, money1 < money3)
# True False
money4 = Money(amount=2, currency_code="USD")
print(money1 > money4, money1 < money4)
# False True
money5 = Money(amount=100, currency_code="JPY")
print(money1 > money5, money1 < money5)
# False False
```

## `__le__`, `__ge__`
> `object.__le__(self, other)`
> `object.__ge__(self, other)`
> `x<=y` は `x.__le__(y)` を呼び出します; `x>=y` は `x.__ge__(y)` を呼び出します;
[公式Doc](https://docs.python.org/ja/3/reference/datamodel.html#object.__le__)

今までの `==`, `<`, `>` さえ計算可能であれば、`<=`, `>=` も計算できるので `__le__`, `__ge__` は定義不要だと思う方がいるかもしれません（`x<=y` は `(x<y or x==y)` に変換可能なので）。
しかし、残念ながらデフォルトではその変換は行ってくれません。
ですが、`@functools.total_ordering` という便利なデコレータが提供されているので、そちらを使えば `__eq__`, `__lt__` の定義だけで済みます。

> [`@functools.total_ordering`](https://docs.python.org/ja/3/library/functools.html#functools.total_ordering)
> ひとつ以上の拡張順序比較メソッド (rich comparison ordering methods) を定義したクラスを受け取り、残りを実装するクラスデコレータです。このデコレータは全ての拡張順序比較演算をサポートするための労力を軽減します
> 引数のクラスは、 `__lt__()`, `__le__()`, `__gt__()`, `__ge__()` の中からどれか1つと、 `__eq__()` メソッドを定義する必要があります。

```python
from functools import total_ordering

@total_ordering
class Money:
    # __init__, __int__, __str__, __eq__, __lt__ 省略

money1 = Money(amount=100, currency_code="JPY")

money2 = Money(amount=100, currency_code="JPY")
print(money1 >= money2, money1 <= money2)
# True True
money3 = Money(amount=99, currency_code="JPY")
print(money1 >= money3, money1 <= money3)
# True False
money4 = Money(amount=1, currency_code="USD")
print(money1 >= money4, money1 <= money4)
# True True
money5 = Money(amount=2, currency_code="USD")
print(money1 >= money5, money1 <= money5)
# False True
```


# 数値型の算術演算系
## `__add__`, `__sub__`, `__mul__`, `__floordiv__`
> 例えば x が `__add__()` メソッドのあるクラスのインスタンスである場合、式 x + y を評価すると type(x).`__add__(x, y)` が呼ばれます。
[公式Doc](https://docs.python.org/ja/3/reference/datamodel.html#object.__add__)

これらの特殊メソッドで四則演算を表現することができます。

```python
@total_ordering
class Money:
    # __init__, __int__, __str__, __eq__, __lt__ 省略

    def __add__(self, other: 'Money') -> 'Money':
        if not isinstance(other, Money):
            return NotImplemented
        if self.currency_code != other.currency_code:
            raise ValueError("currency_codeが異なります")
        return Money(self.amount + other.amount, self.currency_code)

    def __sub__(self, other) -> 'Money':
        if not isinstance(other, Money):
            return NotImplemented
        if self.currency_code != other.currency_code:
            raise ValueError("currency_codeが異なります")
        return Money(self.amount - other.amount, self.currency_code)

    def __mul__(self, other: int) -> 'Money':
        if not isinstance(other, int):
            return NotImplemented
        return Money(self.amount * other, self.currency_code)

    def __floordiv__(self, other: int) -> 'Money':
        if not isinstance(other, int):
            return NotImplemented
        return Money(self.amount // other, self.currency_code)

money1 = Money(amount=100, currency_code="JPY")
money2 = Money(amount=200, currency_code="JPY")
print(money1 + money2)
# 300 (JPY)
print(money2 - money1)
# 100 (JPY)
print(money1 * 2)
# 200 (JPY)
print(money1 / 2)
# 50 (JPY)
```

算術演算系の特殊メソッドは他にもいくつかあります。
| 特殊メソッド | 対応する演算子 |
| ---- | ---- |
| `__add__` | `+` |
| `__sub__` | `-` |
| `__mul__` | `*` |
| `__matmul__` | `@` |
| `__truediv__` | `/` |
| `__floordiv__` | `//` |
| `__mod__` | `%` |
| `__divmod__` | `divmod()` |
| `__pow__` | `pow()`, `**` |
| `__lshift__` | `<<` |
| `__rshift__` | `>>` |
| `__and__` | `&` |
| `__xor__` | `^` |
| `__or__` | `\|` |

## `__r〇〇__`
例えば、上記 Money クラスで `2 * money1` はエラーとなります（`200 (JPY)` の Money インスタンスが返ることを期待している）。`2.__mul__(money1)` に変換されることになりますが、`2` という `int` インスタンスの `__mul__` メソッドは Money 型を受け付けないためです。
この問題は、Money クラスに `__rmul__` メソッドを定義することで解決します。前述した算術演算系の特殊メソッドの前に `r` をつけると以下のような特殊メソッドになります。

> これらのメソッドを呼んで二項算術演算 (+, -, *, @, /, //, %, divmod(), pow(), **, <<, >>, &, ^, |) の、被演算子が反射した (入れ替えられた) ものを実装します。 これらの関数は、左側の被演算子が対応する演算をサポートしておらず、非演算子が異なる型の場合にのみ呼び出されます。
[公式Doc](https://docs.python.org/ja/3/reference/datamodel.html#object.__radd__)

```python
@total_ordering
class Money:
    # __init__, __int__, __str__, __eq__, __lt__, __add__, __sub__, __mul__, __floordiv__ 省略

    def __rmul__(self, other: int) -> 'Money':
        return self.__mul__(other)

money = Money(amount=100, currency_code="JPY")
print(2 * money)
# 200 (JPY)
```

# おわりに
`__〇〇__` メソッドの正体「特殊メソッド」について説明し、個人的によく見る（またはよく使う）特殊メソッドをいくつかご紹介してきました。

実際には、エディタによって `+` から `__add__` に定義元ジャンプしてくれないなどの問題があったりして、プロジェクトによっては用いるべきじゃない場面もあるとは思いますが、Pythonic な書き方であることは間違い無いですし Python コードリーディング力をつけるという意味で知っていて損はない内容かと思っています。他にもこんな特殊メソッドよく使う等の FB があれば是非コメントお願いします！

# 引用
- **Python3 公式ドキュメント 3.3. 特殊メソッド名**

https://docs.python.org/ja/3/reference/datamodel.html#special-method-names