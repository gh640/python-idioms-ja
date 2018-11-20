# Python イディオム集

Python 3.x のイディオム集です。

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

## 複数の `iterable` を同時に回す

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

`tuple` ではまかないきれなくなったら、 `collections.namedtuple` や独自クラスのインスタンスを使います。
