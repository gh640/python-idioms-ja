# Python 3 イディオム集

Python 3.x のイディオム集です。

## `not` が含まれる条件式

`not` が含まれる条件式は人間が読んで読みやすい形（≒英語に近い形）で書きます。

```python
# ○:
if value is not None:
    ...

# ✕:
if not value is None:
    ...
```

```python
# ○:
if key not in adict:
    ...

# ✕:
if not key in adict:
    ...

# ✕:
if not (key in adict):
    ...
```

## 多岐分岐

他の言語にある `switch`/`case` のようなことをしたい場合は `dict` を使用します。

```python
# ○:
def printer_factory(name):
    printer_map = {
        'html': HtmlPrinter,
        'pdf': PdfPrinter,
    }

    try:
        return printer_map[name]()
    except KeyError as e:
        raise ValueError('不正な name が指定されました: {}。'.format(name))
```

## `for` ループ

`iterable` のループには原則インデックスを使いません。

```python
urls = [
    'https://www.google.co.jp',
    'https://www.facebook.com',
    'https://twitter.com',
]

# ○:
for url in urls:
    print(url)

# ✕:
for i in range(len(urls)):
    print(urls[i])
```

## インデックスが必要な `for` ループ

インデックスが必要な `for` ループには `enumerate()` を使用します。

```python
animals = [
    '子',
    '丑',
    '寅',
]

# ○:
for i, animal in enumerate(animals, start=1):
    print('順位 {:02d}: {}'.format(i, animal))
# =>
# 順位 01: 子
# 順位 02: 丑
# 順位 03: 寅
```

## 複数の `iterable` を同時にループで回す

`for` ループで複数の `iterable` を同時に回したい場合は `zip` を使用します。

```python
animals = ['猫', '馬', '河童']
animals_en = ['cat', 'horse', 'kappa']

# ○:
for animal, animal_en in zip(animals, animals_en):
    print('{} は英語で {} です。'.format(animal, animal_en))

# ✕:
for i in range(min(len(animals), len(animals_en))):
    print('{} は英語で {} です。'.format(animals[i], animals_en[i]))
```

2 つの `iterable` の長さが異なる場合、 `zip()` は短い方の長さだけループを回します。
長い方の長さにあわせたい場合は `itertools.zip_longest()` を使用します。

```python
from itertools import zip_longest

# ○:
for animal, animal_en in zip_longest(animals, animals_en):
    ...
```

## `list` のコピー

`list` のコピー（複製）には `copy()` メソッドを使用します。

```python
alist = [...]

# ○:
cloned_list = alist.copy()
```

Python 3.3 よりも前（ 3.2 以下）のバージョンでは `copy()` メソッドが提供されていないので次の書き方を使います。

```python
alist = [...]

# ○:
cloned_list = alist[:]
```

## ファイル利用

ファイルを利用する際は原則 `with` 文（コンテキストマネージャ）を使用します。

```python
path = 'sample.txt'

# ○:
with open(path) as f:
    ...

# ✕:
f = open(path)
...
f.close()
```

## 複数の戻り値

関数やメソッドに複数の戻り値を持たせたい場合は `tuple` を使用します。

```python
import requests

# ○:
def get_page(url):
    res = requests.get(url)
    return res.ok, res.content

is_ok, content = get_page('https://www.yahoo.co.jp/')
```

`tuple` ではまかなえなくなったら、 `collections.namedtuple` や独自クラスのインスタンスを使います。

## コレクション系の値の非空チェック

`list` ・ `tuple` ・ `dict` 等のコレクション系の値の非空チェックは名前を `if` 文に直接渡して行います。

```python
links = magical_func_collecting_links(url)

# ○:
if links:
    ...

# ✕:
if len(links):
    ...

# ✕:
if len(links) > 0:
    ...
```

## 引数の数が多い関数

引数の数が多い関数を使用するときは、実行側でキーワード指定で引数を渡します。

```python
def easy_to_misuse_func(file, column, cond, coerce):
    ...

# ○:
easy_to_misuse_func(file=a, column=b, cond=c, coerce=d)

# ✕:
easy_to_misuse_func(a, b, c, d)
```

関数の定義側で引数の間に `*,` を置くことで、キーワード指定での引数渡しを強制することもできます。

```python
def easy_to_misuse_func(*, file, column, cond, coerce):
    ...

easy_to_misuse_func(a, b, c, d)
# =>
# TypeError: easy_to_misuse_func() takes 0 positional arguments but 4 were given

easy_to_misuse_func(file=a, column=b, cond=c, coerce=d)
# => OK
```

> Parameters after “\*” or “\*identifier” are keyword-only parameters and may only be passed used keyword arguments.

- [Function definitions ― Compound statements — Python 3 documentation](https://docs.python.org/3/reference/compound_stmts.html#function-definitions)

## 関数のデフォルト引数

関数のデフォルト引数には原則 mutable な値は使いません。

```python
# ✕:
def merge_dicts(d1={}, d2={}):
    d1.update(d2)
    return d1

merge_dicts(d2={'a': 1})
# => {'a': 1}
merge_dicts(d2={'b': 2})
# => {'a': 1, 'b': 2}
merge_dict()
# => {'a': 1, 'b': 2}
```

## メソッドチェーン

自作のクラスでメソッドを連ねて呼び出したい場合は、メソッドが `self` （または新しいインスタンス）を返すようにします。

```python
# ○:
class Query:
    def select(self, columns):
        ...
        return self

    def where(self, column, value):
        ...
        return self


figure1 = (Query().select('figure')
        .where('rhand', 'チョキ')
        .where('lhand', 'グー'))
figure2 = (Query().select('figure')
        .where('rhand', 'パー')
        .where('lhand', 'パー'))
```

## `dict` のキーの存在チェック

`dict` に存在するかどうかわからないキーで要素にアクセスする場合は例外処理を使用します。

```python
# ○:
try:
    value = adict[key]
except KeyError:
    ...
...

# ✕:
if key in adict:
    value = adict[key]
    ...
```

このスタイルは「 EAFP 」スタイル（ Easier to Ask for Forgiveness than Permission ）と呼ばれたりします。

- [EAFP ― Glossary — Python 3.x documentation](https://docs.python.org/3/glossary.html#term-eafp)

他方の `if` 文を使ったスタイルは「 LBYL 」スタイル（ Look Before You Leap ）と呼ばれます。

- [LBYL ― Glossary — Python 3.x documentation](https://docs.python.org/3/glossary.html#term-lbyl)

## 複数の例外のキャッチ

複数の例外をキャッチする場合は、「小さい例外」（例外クラスの継承ツリーにおいて子孫側の例外）を先に、「大きい例外」を後にキャッチします。

```python
class CustomBaseError(Exception):
    pass

class InvalidValueError(CustomBaseError):
    pass

# ○:
try:
    myfunc()
except InvalidValueError as e:
    ...
except CustomBaseError as e:
    ...

# ✕:
try:
    myfunc()
except CustomBaseError as e:
    ...
except InvalidValueError as e:
    ...
```
