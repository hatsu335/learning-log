## [Pandas早見表]

## 1. データ読み込み・書き出し  
- import pandas as pd

### CSV読み込み  
- df = pd.read_csv("data.csv")

### Excel読み込み  
- df = pd.read_excel("data.xlsx")

### CSV書き出し  
- df.to_csv("output.csv", index=False)


## 2. データ確認
- df.head()        # 先頭5行
- df.tail(3)       # 末尾3行
- df.info()        # 列名・型・欠損数
- df.describe()    # 基本統計量
- df.shape         # (行数, 列数)
- df.columns       # 列名一覧
- df.dtypes        # 各列のデータ型


## 3. データ選択・抽出
- df["col"]                # 列選択
- df[["col1", "col2"]]     # 複数列選択
- df.loc[0, "col"]         # 行・列指定（ラベルベース）
- df.iloc[0, 1]            # 行・列指定（位置ベース）
- df[df["col"] > 10]       # 条件抽出
- df.query("col > 10")     # クエリ式で抽出


## 4. データ加工
- df.rename(columns={"old": "new"}, inplace=True)  # 列名変更
- df.drop("col", axis=1, inplace=True)             # 列削除
- df.dropna()                                       # 欠損値削除
- df.fillna(0)                                      # 欠損値補完
- df.isnull()                                       # 欠損値検出
- df["new_col"] = df["a"] + df["b"]                 # 新列作成
- df.astype({"col": int})                           # 型変換
- df["new_column"] = [1, 2, 3]                      # 列の追加


## 5. 集計・統計
- df["col"].mean()           # 平均
- df["col"].sum()            # 合計
- df["col"].value_counts()   # 値の出現回数
- df.groupby("category")["value"].mean()  # グループごとの平均
- df.pivot_table(index="A", columns="B", values="C", aggfunc="sum")  # ピボット集計
- df["col"].std()            # 標準偏差
- df["col"].corr()           # 相関係数
- df["col"].median()         # 中央値
- df["col"].max()            # 最大値
- df["col"].min()            # 最小値


## 6. 並び替え
- df.sort_values("col")                # 昇順
- df.sort_values("col", ascending=False)  # 降順


## 7. 結合・マージ
- pd.concat([df1, df2], axis=0)  # 縦結合
- pd.merge(df1, df2, on="key")   # 内部結合


## 8. 日付処理
- df["date"] = pd.to_datetime(df["date"])
- df["year"] = df["date"].dt.year
- df["month"] = df["date"].dt.month


## ポイント

- head(), info(), describe() はデータ確認の基本セット
- loc と iloc の違いを理解すると抽出がスムーズ
- 集計は groupby と pivot_table が強力

## 結合例

サンプルデータ作成
```python
import pandas as pd

# 2つのサンプルデータフレームを作成
df1 = pd.DataFrame({
    'key': ['A', 'B', 'C', 'D'],
    'value1': [1, 2, 3, 4]
})

df2 = pd.DataFrame({
    'key': ['C', 'D', 'E', 'F'],
    'value2': [5, 6, 7, 8]
})
```

結果
```bash
  key  value1
0   A       1
1   B       2
2   C       3
3   D       4

  key  value2
0   C       5
1   D       6
2   E       7
3   F       8
```

結合
```python
# inner merge
print(pd.merge(df1, df2, on='key', how='inner'))

# left merge
print(pd.merge(df1, df2, on='key', how='left'))

# right merge
print(pd.merge(df1, df2, on='key', how='right'))

# outer merge
print(pd.merge(df1, df2, on='key', how='outer'))
```

各種結果
```bash
  key  value1  value2
0   C       3       5
1   D       4       6

  key  value1  value2
0   A       1     NaN
1   B       2     NaN
2   C       3     5.0
3   D       4     6.0

  key  value1  value2
0   C     3.0       5
1   D     4.0       6
2   E     NaN       7
3   F     NaN       8

  key  value1  value2
0   A     1.0     NaN
1   B     2.0     NaN
2   C     3.0     5.0
3   D     4.0     6.0
4   E     NaN     7.0
5   F     NaN     8.0
```

引数 cross の解説
```python
import pandas as pd

df1 = pd.DataFrame({
    'A': ['A0', 'A1', 'A2'],
})

df2 = pd.DataFrame({
    'B': ['B0', 'B1'],
})

# cross結合を使用してカーテシアン積を計算
result = pd.merge(df1, df2, how='cross')

print(result)
```

結果
```bash
   A   B
0  A0  B0
1  A0  B1
2  A1  B0
3  A1  B1
4  A2  B0
5  A2  B1
```