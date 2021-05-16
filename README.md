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

## Dockerでの環境構築

### Dockerをインストール

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

### リポジトリをクローン

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

### .env作成
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

### コンテナのポート番号の確認

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

## M1 Macの方の作業

M1版のDockerでは現在mysql:5.7のイメージが対応できていないので、以下の通り修正してください。

```diff

+ image: mariadb:10.3
- image: mysql:5.7

```

### ビルド&コンテナ起動

`Yanbaru-Qiita-App`ディレクトリで以下のコマンドを実行してビルド＆コンテナ起動します。

```
$ docker compose up -d --build
```

以下コマンドで3つのコンテナが起動（Up）しているのを確認できたらOKです。

```
$ docker ps
```

これで環境構築は完了です。

## DBと接続

### Mac

[こちらの記事](https://qiita.com/miriwo/items/f24e6906105386ddfa83)を参考にMySQLのクライアントツール`Sequel Pro`をインストール。<br>

Sequel Proを起動します。<br>

左下の「+」ボタンを押して以下の通り入力欄を埋めます。

|入力欄|入力値|
|:--:|:--:|
|名前|yanbaru-qiita|
|ホスト|127.0.0.1|
|ユーザー名|.envのDB_USER|
|パスワード|.envのDB_PASSWORD|
|データベース|空欄でOK|
|ポート|3306|

接続の前に「お気に入りに追加」を押しておくと次回からすぐに接続できます。<br>
お気に入り登録した後、「接続」ボタンで接続。<br>
左上の「データベースを選択...」で`.env`の`DB_DATABASE`に指定したデータベースを選択し、接続できれば完了です。

※接続できない場合は各自調べてみてエラー解決に挑戦してみましょう。<br>

### Windows

[Mk-2](https://qiita.com/miriwo/items/f24e6906105386ddfa83)などをインストールして使用してみてください。<br>
Mk-2設定参考：procedure_Mk-2.pdf


## Laravelの設定

### 前提
まずは以下の状態になっているか確認ください。

- Dockerコンテナが3つ（web、db、app）が起動している
- Sequel ProでMySQLに接続できている

Dockerコンテナの起動状態は以下コマンドから確認できます。

```
$ docker ps
```

起動してない場合は以下コマンドで起動してください。

```
$ docker compose up -d
```

### Laravel用の.env作成

`src`ディレクトリに移動。<br>
※`cd src`を実行するのでも、エディター上で移動するのでもどちらでも良いです。<br>

既存の`.env.example`をコピーして`.env`を作成してください。（`.env.example`と`.env`が両方できる状態になります）<br>

※srcディレクトリ直下に`.env`があればOKです。Docker環境用の`.env`とは別ファイルですのでご注意ください。

### パッケージのインストールとAPP_KEYの発行

一度、`Yanbaru-Qiita-App`ディレクトリに戻り、以下のコマンドを実行してappコンテナの中に入ります。

```
$ docker compose exec app bash
```

Composerで必要なパッケージをインストールします。<br>

```
$ composer install

（略）
Package manifest generated successfully.
61 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
```

続けて以下のコマンドを実行

```
$ php artisan key:generate
```

`.env`の`APP_KEY`に乱数が入ります。<br>

このケースのようにDocker環境の場合、`php artisan`コマンドはappコンテナの中で実行しますので覚えておきましょう。


### Laravelのウェルカムページの表示

`localhost:80`をブラウザに入力してLaravelのウェルカムページが表示されれば完了です！！<br>

これで環境構築は完了です！これから共同開発を頑張っていきましょう！

# 参考記事

- [【導入編】絶対に失敗しないDockerでLaravel + Vue.jsの開発環境（LEMP環境）を構築する方法〜MacOS Intel Chip対応〜](https://yutaro-blog.net/2021/04/28/docker-laravel-vuejs-intel-1/)
- [【前編】絶対に失敗しないDockerでLaravel + Vue.jsの開発環境（LEMP環境）を構築する方法〜MacOS Intel Chip対応〜](https://yutaro-blog.net/2021/04/28/docker-laravel-vuejs-intel-2/)
- [【後編】絶対に失敗しないDockerでLaravel + Vue.jsの開発環境（LEMP環境）を構築する方法〜MacOS Intel Chip対応〜](https://yutaro-blog.net/2021/04/28/docker-laravel-vuejs-intel-3/)

# 共同開発資料

`development-document`ディレクトリに以下のファイルがありますのでそちらを確認いただきチームメンバーと協力して進めてください。

|資料名|説明|
|:--:|:--|
|ER図.drawio|DBのテーブル定義書です。<br>マイグレーションファイル、シーダーファイルを作成する時に参照ください。|
|ER図.svg|ER図をブラウザで開けますのでこちらの方が見やすいです。|
|画面遷移図.drawio|各画面の遷移図です。|

※ER図、画面遷移図は別途配布する**画面定義書（エクセルファイル）**にも載せております。
