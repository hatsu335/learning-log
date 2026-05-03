# [MySQL SSL/TLS 設定]

MySQLの接続を **SSL/TLS（暗号化通信）** にするのは、例えるなら「ハガキ（平文）」で送っていたデータを「封筒（暗号化）」に入れて送るようなものです。これにより、万が一社内ネットワーク内のパケットを盗み見られても、中身（パスワードや進捗データ）を解読できなくなります。

現在の構成（Anaconda環境のPythonからMySQLへ接続）において、SSL/TLSを導入する具体的な手順は以下の通りです。

---

## 1. MySQL サーバー側の準備
最近の MySQL（バージョン 8.0 以降など）は、インストール時にデフォルトで SSL 用の証明書を自動生成していることが多いです。

### ① SSL が有効か確認する
MySQL にログインして、以下のコマンドを打ってみてください。
```sql
SHOW VARIABLES LIKE '%ssl%';
```
`have_ssl` が `YES` になっていれば、サーバー側はすでに暗号化を受け入れる準備ができています。

### ② SSL 強制の設定（推奨）
特定のユーザーに対して、「SSL 通信でないと接続させない」というルールを設けます。
```sql
ALTER USER 'staff_user'@'%' REQUIRE SSL;
FLUSH PRIVILEGES;
```

---

## 2. Python (PySide6) 側の接続コードの修正
Python側で MySQL に接続する際（`mysql-connector-python` や `PyMySQL` を使っている場合）、接続オプションに SSL の設定を書き加えるだけで完了します。

### 例：`mysql-connector-python` を使っている場合
特別な証明書ファイルを指定しなくても、単純に「SSL を使う」という指示だけで動くケースがほとんどです。

```python
import mysql.connector

config = {
    'user': 'staff_user',
    'password': 'your_password',
    'host': '192.168.x.x',
    'database': 'honeycomb_db',
    # ここに SSL 設定を追加
    'ssl_disabled': False,
    'ssl_verify_cert': False  # 自前で証明書を管理しない場合は False でOK
}

try:
    conn = mysql.connector.connect(**config)
    print("SSL/TLS で安全に接続されました")
except mysql.connector.Error as err:
    print(f"接続失敗: {err}")
```

---

## 3. なぜ「ssl_verify_cert: False」でいいのか？
本来、本格的なセキュリティでは「証明書ファイル（.pem）」をクライアントに配布して検証させますが、社内ツールであれば **「経路の暗号化」** が第一目標です。
*   **False にした場合:** 通信は暗号化されますが、「相手が本当に自分のサーバーか」の厳密な確認をスキップします。
*   **メリット:** 証明書ファイルを各PCに配布・管理する手間が省け、運用の難易度が下がります。

---

## 4. セキュリティを高めるための全体像


SSL/TLS を導入することで、これまでの「IP制限」という **外壁** に加え、データそのものを守る **金庫** を手に入れることになります。

### 注意点：パフォーマンスへの影響
暗号化・復号の処理が加わるため、ごくわずかに通信速度が落ちることがありますが、今回のような「進捗管理システム」のテキストデータ量であれば、体感できるほどの差は出ませんので安心してください。

---

### 次のステップへのアドバイス
まずは、今の環境で `SHOW VARIABLES LIKE '%ssl%';` を実行して、サーバーの状態を確認してみてください。

もし、より厳格に「証明書ファイルを使った認証」を行いたい場合は、MySQL のデータフォルダ内にある `ca.pem` などのファイルを Python 側で読み込ませる必要があります。その手順が必要になった際も、また詳しくガイドしますね！

SQLAlchemyを使用している場合、接続文字列（URL）の末尾に**クエリパラメータとしてSSLの設定を追記**するか、`create_engine` 関数の引数に **`connect_args`** を渡すことで設定できます。

お使いのライブラリ（MySQLドライバ）によって書き方が少し異なりますが、現在主流の2つのパターンを解説します。

---

### パターン1：`mysql-connector-python` を使っている場合
Anaconda環境で標準的に使われることが多い公式ドライバです。

```python
from sqlalchemy import create_engine

# 接続情報（URL）
DATABASE_URL = "mysql+mysqlconnector://user:password@host/dbname"

# SSL設定を connect_args で渡す
engine = create_engine(
    DATABASE_URL,
    connect_args={
        "ssl_disabled": False
    }
)
```

### パターン2：`PyMySQL` を使っている場合
軽量でよく使われるドライバです。

```python
from sqlalchemy import create_engine

# 接続情報（URL）
DATABASE_URL = "mysql+pymysql://user:password@host/dbname"

# SSL設定を connect_args で渡す
engine = create_engine(
    DATABASE_URL,
    connect_args={
        "ssl": {
            "fake_config": True  # 詳細な証明書指定をしない場合のダミー設定
        }
    }
)
```

---

### 3. 「本当にSSLで繋がっているか」確認する方法
プログラム内で以下のコードを実行してみてください。現在ログインしているセッションのSSL状態を直接取得できます。

```python
from sqlalchemy import text

with engine.connect() as connection:
    result = connection.execute(text("SHOW SESSION STATUS LIKE 'Ssl_cipher'"))
    for row in result:
        print(f"暗号化方式: {row[1]}") 
        # ここに 'ECDHE-RSA-AES256-GCM-SHA384' などの文字が出ればSSL接続成功
        # 空っぽなら非SSL接続です
```

---

### 運用上のポイント：設定ファイル（YAML）との連携
以前お話しした `config.yaml` を活用して、本番環境（社内ネットワーク）では SSL を有効にする、といった切り替えも簡単です。



```yaml
# config.yaml のイメージ
database:
  host: "192.168.x.x"
  user: "staff_user"
  use_ssl: true
```

```python
# Python側での処理イメージ
ssl_args = {"ssl_disabled": False} if config['database']['use_ssl'] else {}
engine = create_engine(DATABASE_URL, connect_args=ssl_args)
```

### まとめ
SQLAlchemyを使えば、**コードの大部分を変えることなく、接続部分の設定（`connect_args`）を一行追加するだけ**で通信を暗号化できます。

まずは `SHOW SESSION STATUS LIKE 'Ssl_cipher'` を使って、現在の接続が「ハガキ（平文）」なのか「封筒（SSL）」なのかをチェックしてみることから始めるのがおすすめです。

もし社内の PC の電源を 24 時間入れている人がいて、その通信経路に不安があるなら、この SSL 化は非常に有効な「自衛策」になりますよ！