2020/2/7-8
## 発生した問題
- MySQLにアクセスできなかった

## 原因の特定
- はまったエラーは「アクセス拒否エラー(Access denied for user ‘root’@’localhost’)」
- まず「MySQLにアクセス」の意味からわからなかった。今もよくわかってない。
- rootユーザーの権限設定がされていなかったことが原因

## 問題の解決
- 「Access denied for user ‘root’@’localhost’」でとにかくググった。
- 権限なしでアクセスできる以下のコマンドを実行した→アクセスできず
```ruby
mysqld_safe --skip-grant-tables &
````
- systemctlをいじる。[このページ](https://teratail.com/questions/53026)を参照して以下のコマンドを実行→アクセスできた
```ruby
systemctl stop mysqld
systemctl set-environment MYSQLD_OPTS="--skip-grant-tables"
systemctl start mysqld
````
- セキュリティのため、rootユーザーに権限設定する。[このページ](https://www.aiship.jp/knowhow/archives/28257#outline__4)を参照して以下のコマンドを実行→解決
MySQLにログインしないと権限設定はできない。
```ruby
mysql> flush privileges;　#キャッシュの削除
mysql> grant all privileges on *.* to root@localhost identified by 'パスワード' with grant option;　#rootユーザーに全権限を付与してパスワードを設定
mysql> flush privileges;　#キャッシュの削除
mysql> quit
Bye
````
これでrootユーザーへの権限設定は完了。
```ruby
mysql -u root -p
````
設定したパスワードを入力すればMySQLにログインできる。
