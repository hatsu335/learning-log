## [パッケージング]

### パッケージングの流れ  

- 過去にパッケージング用で作成していた仮想環境を削除
- パッケージング用のフォルダを作成して必要なファイル群をコピーして入れる
- 新たに仮想環境を作成
- 必要なモジュールをインストール
- 新な仮想環境下でスクリプトを実行できるかチェック
- パッケージング

---

### 用語の整理  

| 用語      | 意味             | ニュアンス |
| ------- | -------------- | ----- |
| EXE化    | Windows実行ファイル化 | カジュアル |
| ビルド     | 実行物生成          | 一般的   |
| パッケージング | 配布形式にまとめる      | 広い概念  |
| freeze  | Python特有       | 技術寄り  |

#### 今回とは関係ないが今後出てくる関連用語  

- デプロイ（deploy） → 実際に使う環境へ配置・公開
- ディストリビューション（distribution） → 配布物（distフォルダの語源）
- 依存関係（dependencies） → ライブラリ群

---

## [パッケージング詳細]

### 1. 仮想環境の削除は仮想環境フォルダを削除するだけでOK  
(`conda`は違うので注意)

### 2. パッケージング用のフォルダを作成、必要なスクリプトやフォルダや画像などをコピーして入れる

### 3. Powershell で仮想環境を作成

```bash
python -m venv build_env
```

### 4. 該当フォルダのパスまで移動して仮想環境に入る

```bash
build_env\scripts\activate
```

### 5. 必要なモジュールを`pip`でインストール

```bash
pip install pyinstaller sqlalchemy pymysql reportlab pyyaml pandas openpyxl pyside6 pdf2image
```

### 6. 特に必要ではないが、念の為確認

結果が返ってくれば問題なし、何も返ってこなければ`Python`の場所がわからない証拠
```bash
where.exe python
```

### 7. モジュールの確認をする必要がある場合

仮想環境内のモジュール確認
```bash
pip list
```

単体のモジュール確認
```bash
show pyside6
```

### 8. 環境が整ったらスクリプトを起動してチェック

```bash
python honeycomb_system_pyside6.py
```

### 9. パッケージング

```bash
pyinstaller --noconsole --icon=戦闘機.ico --clean --noconfirm honeycomb_system_pyside6.py
```

上記でできなければ --collect-all pyside6 を付けて試す(フルスペックにする)  

```bash
pyinstaller --noconsole --icon=戦闘機.ico --clean --noconfirm --collect-all pyside6 honeycomb_system_pyside6.py
```

- `--noconsole` コンソールが出ないようにする
- `--clean` キャッシュ削除
- `--noconfirm` 途中で上書きを聞いてくるのを自動で飛ばす
- `--collect-all` モジュールをフルスペックで入れる

### 10. `requirements.txt`を作成しておくとよい(環境等復元)

作成方法
```bash
pip freeze > requirements.txt
```
使用方法
1. venv作成
2. pip install -r requirements.txt
3. pyinstaller実行