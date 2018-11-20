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
