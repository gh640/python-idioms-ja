# Python 3 イディオム集

Python 3.x のイディオム集です。

## 目次

- [コレクション系の値の名前](#コレクション系の値の名前)
- [コレクション系の値の非空チェック](#コレクション系の値の非空チェック)
- [`not` が含まれる条件式](#not-が含まれる条件式)
- [多岐分岐](#多岐分岐)
- [`for` ループ](#for-ループ)
- [インデックスが必要な `for` ループ](#インデックスが必要な-for-ループ)
- [複数の iterable の `for` ループ](#複数の-iterable-の-for-ループ)
- [`list` のコピー](#list-のコピー)
- [`dict` のサブセットの取得](#dict-のサブセットの取得)
- [ファイル利用](#ファイル利用)
- [複数の戻り値](#複数の戻り値)
- [引数の数が多い関数](#引数の数が多い関数)
- [関数のデフォルト引数](#関数のデフォルト引数)
- [メソッドチェーン](#メソッドチェーン)
- [`dict` のキーの存在チェック](#dict-のキーの存在チェック)
- [複数の `dict` のマージ](#複数の-dict-のマージ)
- [バージョンによって場所が異なる関数やクラスの `import`](#バージョンによって場所が異なる関数やクラスの-import)
- [行の長さ制限を守るための改行](#行の長さ制限を守るための改行)
- [複数の例外のキャッチ](#複数の例外のキャッチ)

## コレクション系の値の名前

`list` ・ `tuple` 等のコレクション系の値の名前には単語の複数形を使います。

```python
# ○:
numbers = [1, 3, 5, ...]
urls = (
    'https://www.google.co.jp',
    'https://www.facebook.com',
    'https://twitter.com',
)
menu_items = (
    ('/about/', '○○とは'),
    ('/services/', 'サービス'),
    ('/contact/', '問い合わせ'),
)

# ✕:
number = [...]
url = (
    ...,
)
menu_item = (
    ...,
)
```

`for` ループや内包表記でその要素を取り出すときは単数形を使います。
スコープが狭く意味が明白な場合は 1 文字変数を使っても大丈夫です。

```python
# ○:
for n in numbers:
    ...

for url in urls:
    ...

for i in menu_items:
    ...
```

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

## `not` が含まれる条件式

`not` が含まれる条件式は人間が読んで読みやすい形（≒英語の語順に近い形）で書きます。

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
        'toml': TomlPrinter,
        'dsv': DsvPrinter,
    }

    try:
        return printer_map[name]()
    except KeyError as e:
        raise ValueError('不正な name が指定されました: {}。'.format(name))
```

## `for` ループ

iterable のループには原則インデックスを使いません。

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

## 複数の iterable の `for` ループ

`for` ループで複数の iterable を同時に回したい場合は `zip` を使用します。

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

2 つの iterable の長さが異なる場合、 `zip()` は短い方の長さだけループを回します。
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

Python 3.3 未満（ 3.2 以下）のバージョンでは `copy()` メソッドが提供されていないので次の書き方を使います。

```python
alist = [...]

# ○:
cloned_list = alist[:]
```

## `dict` のサブセットの取得

`dict` のサブセットの取得には `{}` を使った `dict` の内包表記を使用します。

```python
adict = {'tokyo': 'T', 'hokkaido': 'H', 'okinawa': 'O', 'kagoshima': 'K'}
targets = ['tokyo', 'kagoshima']

# ○:
subdict = {k: adict[k] for k in targets}
# {'tokyo': 'T', 'kagoshima': 'K'}

# ○:
subdict = {k: adict[k] for k in adict.keys() & targets}
# {'kagoshima': 'K', 'tokyo': 'T'}
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
# ○:
def merge_dicts(d1=None, d2=None):
    d1 = d1 or {}
    d2 = d2 or {}

    d1.update(d2)
    return d1

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


figure1 = (Query()
           .select(['figure'])
           .where('rhand', 'チョキ')
           .where('lhand', 'グー'))
figure2 = (Query()
           .select(['figure'])
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

## 複数の `dict` のマージ

複数の `dict` を組み合わせて新たな `dict` を作りたいときは `dict` のアンパック演算を使用します。

```python
dict1 = {...}
dict2 = {...}
dict3 = {...}

merged_dict = {**dict1, **dict2, **dict3}
```

キーの衝突が発生した場合は後に方に記述された `dict` の値が残ります:

```python
items1 = {'コーラ': 'ペプシ', 'ジンジャーエール': 'ウィルキンソン'}
items2 = {'ジンジャーエール': 'カナダドライ'}

merged = {**items1, **items2}
# => {'コーラ': 'ペプシ', 'ジンジャーエール': 'カナダドライ'}
```

`dict` の数が不定の場合は `dict` の内容表記（ dict comprehension ）を使うとシンプルに書けます。

```python
many_dicts = [dict1, dict2, ..., dictn]

merged_dict = {k: v for d in many_dicts for k, v in d.items()}
```

## バージョンによって場所が異なる関数やクラスの `import`

モジュールのバージョンによって場所が異なる関数やクラスの `import` には `try` `except` を使用します。

```python
# ○:
try:
    from urllib.parse import urlparse
except:
    from urlparse import urlparse
```

## 行の長さ制限を守るための改行

コードをそのまま書くと 80 文字や 100 文字等の行長さ制限にひっかかる場合は、 `()` を使用したり式を分割したりして制限以内に収めます。

関数・メソッド呼び出し:

```python
# ○:
class Student(models.Model):
    year_in_school = models.CharField(
        max_length=2,
        choices=YEAR_IN_SCHOOL_CHOICES,
        default=FRESHMAN,
        db_index=True,
    )

# ✕
class Student(models.Model):
    year_in_school = models.CharField(max_length=2, choices=YEAR_IN_SCHOOL_CHOICES, default=FRESHMAN, db_index=True)

```

メソッドチェーン:

```python
# ○:
class PublishedBookRelatedManager(models.Manager):
    def get_queryset(self):
        books_published = Book.published_objects.all()
        return (
            super()
            .get_queryset()
            .distinct()
            .filter(book__in=books_published)
        )

# ✕:
class PublishedBookRelatedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().distinct().filter(book__in=Book.published_objects.all())

# ✕:
class PublishedBookRelatedManager(models.Manager):
    def get_queryset(self):
        books_published = Book.published_objects.all()
        return super().get_queryset() \
            .distinct() \
            .filter(book__in=books_published)
```

文字列:

```python
class MyFormView(FormView):
    def form_valid(self, form):
        # この形で書かれた文字列リテラルは自動的に連結されるので `+` 演算子は不要です
        message = (
            'お問い合わせいただきありがとうございます。'
            'ご入力いただいたメールアドレスに 3 営業日以内にご連絡差し上げます。'
            '連絡が無い場合は大変お手数ですが再度お問い合わせいただきますようお願いいたします。'
        )
        messages.success(message)
        return super().form_valid(form)
```

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
