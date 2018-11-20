# Python イディオム集

Python 3.x のイディオム集です。

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
