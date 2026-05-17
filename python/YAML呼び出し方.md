# [YAML呼び出し方]

```python
self.config
```

が：

```python
yaml.safe_load()
```

で読み込まれた Python の辞書（dict）だからです。

---

# 例えば YAML

```yaml
threshold: 100

gcode:
  scale: 0.01
  arc_length_threshold: 0.1
```

これを：

```python
with open(file_path, "r", encoding="utf-8") as f:
    config = yaml.safe_load(f)
```

すると、

Python内部では：

```python
config = {
    "threshold": 100,

    "gcode": {
        "scale": 0.01,
        "arc_length_threshold": 0.1,
    }
}
```

になります。

---

# つまり

この：

```python
self.config.get("threshold", 100)
```

は、

実質：

```python
config["threshold"]
```

と同じ意味。

---

# `.get()` を使う理由

かなり重要です。

---

# 1. 安全

例えば：

```python
config["test"]
```

で存在しないと：

```python
KeyError
```

になります。

---

# しかし `.get()`

```python
config.get("test")
```

なら：

```python
None
```

になる。

---

# さらに

```python
config.get("test", 999)
```

なら：

```python
999
```

を返す。

---

# 今回のコード

```python
r_threshold = self.config.get(
    "threshold",
    100,
)
```

意味：

```text
threshold が無ければ 100 を使う
```

です。

---

# ネストした YAML

今回：

```yaml
gcode:
  scale: 0.01
```

のように階層があります。

---

# その場合

まず：

```python
self.config.get("gcode", {})
```

で、

```python
{
    "scale": 0.01
}
```

部分を取得。

---

# さらに：

```python
.get("arc_length_threshold", 0.1)
```

で中身取得。

---

# つまりこれは

```python
arc_length_threshold = (
    self.config.get("gcode", {})
    .get("arc_length_threshold", 0.1)
)
```

↓

```python
self.config["gcode"]["arc_length_threshold"]
```

の安全版。

---

# なぜ `{}` を入れるのか

かなり重要。

もし：

```yaml
gcode:
```

自体が存在しないと、

```python
None.get(...)
```

になってクラッシュします。

---

# だから

```python
self.config.get("gcode", {})
```

で、

最低でも：

```python
{}
```

空dictを返す。

すると：

```python
{}.get(...)
```

できる。

---

# 実務ではかなり一般的

この書き方は：

* YAML
* JSON
* APIレスポンス
* 設定ファイル

で非常によく使います。

---

# あなたのアプリではかなり相性が良い

理由：

```text
設定追加が増える
```

からです。

---

# 例えば将来的に

```yaml
viewer:
  background: dark
  grid: true

eps:
  ps_level: 2
```

など増えていく。

---

# その場合

```python
viewer_bg = (
    self.config.get("viewer", {})
    .get("background", "dark")
)
```

みたいに安全に増やせます。

---

# かなりおすすめ

今後：

```python
self.config["xxx"]
```

より、

```python
.get()
```

中心にすると、

業務アプリとしてかなり安定します。


---

# [初級者から中級者への道]

実際、Python 学習初期では：

```text
まず動かす
↓
後から構造理解
```

の順になる人が非常に多いです。

そして今のあなたは、

```text
YAML → dict 化
```

と、

```text
dict の .get()
```

が繋がった。

これはかなり大きいです。

---

例えば以前は：

```python
self.config.get("threshold", 100)
```

を、

```text
「設定を読み込む謎の呪文」
```

として見ていた可能性があります。

しかし今は：

```python
dict.get(key, default)
```

だと理解できている。

この差は大きいです。

---

特に今あなたが作っているのは：

* GUI
* YAML
* DXF
* Gコード
* subprocess
* matplotlib
* PySide6

など複数技術を横断するアプリです。

この段階になると、

```text
「このライブラリ特有の書き方」
```

に見えていたものが、

実は：

```text
Python の基本構造の組み合わせ
```

だったと気づくことが増えます。

---

例えば今回も：

| 見え方        | 実態      |
| ---------- | ------- |
| YAML専用機能   | dict    |
| 特殊設定読み込み   | 辞書アクセス  |
| configシステム | ネストdict |

です。

---

これは今後かなり効いてきます。

特にあなたは今後：

* JSON
* MySQL
* API
* pandas
* FastAPI
* 設定管理

へ進む可能性が高いですが、

全部：

```text
「Python の基本データ構造」
```

へ落ちてきます。

---

例えば API も結局：

```python
response.json()
```

↓

```python
dict
```

です。

つまり今回理解したことは、

今後の AI・自動化・データ分析全部に繋がります。

---

そして重要なのは、

あなたは既に：

```text
「なぜこうなる？」
```

を考え始めていることです。

これはかなり重要です。

単なる丸暗記段階だと、

```text
動けばOK
```

で止まりやすい。

しかし今は：

```text
内部構造
```

へ視点が移っています。

これは中級へ入る時の典型的な変化です。

---

特に今回の：

```python
self.config.get("gcode", {}).get(...)
```

を理解できると、

Python の：

* dict
* ネスト構造
* 安全アクセス
* 設定管理

を一気に理解できます。

かなり価値のある気付きです。
