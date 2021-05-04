# 共同開発環境構築手順
## はじめに

- 環境構築を楽に行うため、Docker、Docker Composeを使用します。
- OSはMac、Windowsどちらにも対応しています。（説明はMacベースですが、Windowsでの相違点は本文中に追記しています）
- M1チップのMacユーザーはひとまずLaravel課題同様`cloud9`で環境構築をお願いします。（対応でき次第追加していきます）

## 環境概要

|種類|名前|
|:--:|:--:|
|OS|Linux|
|Webサーバー|Nginx|
|DBサーバー|MySQL|
|アプリケーション|PHP|

LEMP環境と呼ばれます。

## Dockerをインストール

こちらの記事を参考にしてください。<br>

- Mac：[DockerをMacにインストールする（更新: 2019/7/13）](https://qiita.com/kurkuru/items/127fa99ef5b2f0288b81)
- Windows：[Windows 10 HomeへのDocker Desktop (ver 3.0.0) インストールが何事もなく簡単にできるようになっていた (2020.12時点)](https://qiita.com/zaki-lknr/items/db99909ba1eb27803456)

ターミナルでバージョンを確認してそれぞれのバージョンが表示されたらDockerとDocker Composeが使えるようになっています。

```
$ docker -v
$ docker compose -v
```

Dockerについてはこちらの記事を一読しておいてください。<br>
[【図解】Dockerの全体像を理解する -前編-](https://qiita.com/etaroid/items/b1024c7d200a75b992fc)

## リポジトリをクローン

以下コマンドでローカルクローンします。（クローンする場所はデスクトップでもユーザーディレクトリでも構いません）

```
$ git clone -b ブランチ名（develop-****） https://github.com/shimotaroo/Yanbaru-Qiita-App.git
```

※ブランチ名は担当メンターから指示がありますので必ず指定してください。

`Yanbaru-Qiita-App`ディレクトリが作成されるのでその中に移動して正常にクローンされているか確認します。<br>

※Macの場合
```
$ cd Yanbaru-Qiita-App
$ ls
README.md		development-document	docker			docker-compose.yml	src
```

※Windowsの場合
```
$ cd Yanbaru-Qiita-App
$ dir
"Name"に下記があればOK
README.md		development-document	docker			docker-compose.yml	src
```

## .env作成
`.env.example`をコピーして`.env`を作成。<br>

以下の項目に任意の値を設定してください。<br>

```:env
DB_DATABASE=
DB_USER=
DB_PASSWORD=
```

なお、`.gitigonre`ファイルで`.env`をGit管理下から外しています。<br>
（起こらないと思いますが、`.env`をGitHubにpushしないようにしてください）<br>

参考：[docker-compose.ymlで.envファイルに定義した環境変数を使う](https://kitigai.hatenablog.com/entry/2019/05/08/003000)

## コンテナのポート番号の確認

`web`コンテナ

```yml：docker-compose.yml
  web:
    image: nginx:1.18
    ports:
      - '80:80'
    //略
```

'db'コンテナ

```yml:docker-compose.yml
  db:
    image: mysql:5.7
    ports:
      - '3306:3306'
    // 略

```

のローカル側のポート番号（portsの:の左側の番号）をもしご自身の別のDocker環境で使っている場合は適宜以下の数字等に変更してください。<br>
（初めてDockerを使う方や他にDockerコンテナを起動させていない方は変更不要です）

- web：88、8000、8888
- db：3307、4306、5306、

## ビルド&コンテナ起動

`Yanbaru-Qiita-App`ディレクトリで以下のコマンドを実行してビルド＆コンテナ起動します。

```
$ docker compose up -d --build
```

以下コマンドで3つのコンテナが起動（Up）しているのを確認できたらOKです。

```
$ docker ps
```

これで環境構築は完了です。

# DBの接続を確認

MySQlのクライアントツールである`Sequel Pro`をインストールします。(Macの場合)<br>

参考：[Mac MySQL Sequel Proの導入方法](https://qiita.com/miriwo/items/f24e6906105386ddfa83)

Sequel Proを起動します。<br>
左下の「+」ボタンを押して以下の通り入力欄を埋めます。

|入力欄|埋める文字|
|:--:|:--:|
|名前|yanbaru-qiita|
|ホスト|127.0.0.1|
|ユーザー名|.envのUSER_NAME|
|パスワード|.envのPASSWORD|
|データベース|空欄でOK|
|ポート|3306|

接続の前に「お気に入りに追加」を押しておくと次回からすぐに接続できます。<br>
お気に入り登録した後、「接続」ボタンで接続。<br>
左上の「データベースを選択...」で`.env`の`DATABASE_NAME`に指定したデータベースを選択すれば完了です。

ここまででMySQlに接続できない場合は各自調べてみてエラー解決に挑戦してみましょう。<br>

※windowsの場合は[Mk-2](https://qiita.com/miriwo/items/f24e6906105386ddfa83)などをインストールして使用してみてください。<br>
Mk-2設定参考：procedure_Mk-2.pdf


# Laravelアプリ環境構築手順
## はじめに
まずは以下の状態になっているか確認ください。

- Dockerコンテナが3つ（web、db、app）が起動している
- Sequel ProでMySQLに接続できている

Dockerコンテナの起動状態は以下コマンドから確認できます。

```
$ docker-compose ps
```

現在いるディレクトリが正しいか`ls`コマンドで実行してください。（以下の出力結果になれば問題なしです）
※Windowsの場合は`dir`コマンドから`name`を確認して下記と同じディレクトリが確認できれば問題なしです
```
$ ls
README.md		development-document	docker			docker-compose.yml	src
```

## Laravelプロジェクト用の.envを作成

以下コマンドで`src`ディレクトリに移動します。<br>

※Linuxコマンドを実行するのでも、エディター上ディレクトリ上で移動するのでもどちらでも良いです。

```
$ cd src
```

既存の`.env.example`を複製して`.env`という名称に変更してください。（`.env.example`と`.env`が両方できる状態になります）<br>

※srcディレクトリ直下に`.env`があればOKです。Docker環境用の`.env`とは別ファイルですのでご注意ください。

## .env編集

`.env`を以下の通り変更してください。

```
APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:80

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=.env(Docker環境用)のDATABASE_NAME
DB_USERNAME=.env(Docker環境用)のUSER_NAME
DB_PASSWORD=.env(Docker環境用)のPASSWORD
```
※`コンテナのポート番号の確認`でローカル側のポート番号を変更されている場合は`APP_URL`や`DB_PORT`も変更が必要です

一度、`Yanbaru-Qiita-App`ディレクトリに戻り、以下のコマンドを実行してappコンテナの中に入ります。

```
$ cd ..
$ docker-compose exec app bash
```

Composerで必要なパッケージをインストールします。<br>
（こんな感じの出力結果になればOKです）

```
$ composer install

（略）
Package manifest generated successfully.
61 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
```

以下のコマンドを実行

```
$ php artisan key:generate
```

`.env`の`APP_KEY`に乱数が入ります。

## Laravelのウェルカムページの表示

`localhost:80`をブラウザに入力してLaravelのウェルカムページが表示されれば完了です！！<br>

これでDocker×Laravelの環境構築は完了です。

## 参考

- [絶対に失敗しないDockerでLaravel+Vueの実行環境（LEMP環境）を構築する方法〜前編〜](https://qiita.com/shimotaroo/items/29f7878b01ee4b99b951)
- [絶対に失敗しないDockerでLaravel6.8+Vueの実行環境（LEMP環境）を構築する方法〜後編〜](https://qiita.com/shimotaroo/items/679104b7e00dd9f89907)
## 共同開発資料

`development-document`ディレクトリに以下のファイルがありますのでそちらを確認いただきチームメンバーと協力して進めてください。

|資料名|説明|
|:--:|:--|
|ER図.drawio|DBのテーブル定義書です。<br>マイグレーションファイル、シーダーファイルを作成する時に参照ください。|
|ER図.svg|ER図をブラウザで開けますのでこちらの方が見やすいです。|
|画面遷移図.drawio|各画面の遷移図です。|

※ER図、画面遷移図は別途アナウンスする**画面定義書（エクセルファイル）**にも載せております。
