# moodle-docker

Docker Compose で Moodle をローカル起動するための開発用基盤リポジトリです。

このリポジトリ自体は Moodle 本体を管理しません。PHP 8.3 + Apache + Xdebug を使う `web` コンテナと、MariaDB 11.8 を使う `db` コンテナを用意し、任意の Moodle ソースコードを `src/` に配置するだけですぐに開発を始められる状態を提供します。

`src/` と `moodledata/` は `.gitignore` に含まれており、利用者ごとのローカル作業領域として扱う前提です。

## このリポジトリでできること

- 任意のバージョンの Moodle を `src/` にクローンしてすぐに起動する
- `http://localhost:8080` で Moodle を確認する
- `src/` を直接編集してローカル開発する
- Xdebug を使って PHP デバッグする
- `moodledata/` と Docker volume で状態を保持する

## 構成

- `docker-compose.yml`: `web` と `db` の定義
- `docker/Dockerfile`: PHP 8.3 / Apache / 必要な拡張 / Xdebug のセットアップ
- `docker/php.ini`: PHP 設定
- `src/`: 利用者が任意の Moodle 本体をクローンするディレクトリ
- `src/public/`: Moodle 側の Apache ドキュメントルート
- `moodledata/`: Moodle のデータディレクトリ

## 前提条件

- Docker
- Docker Compose
- ホストの `8080` 番ポートが空いていること
- Xdebug を使う場合は `9003` 番ポートが空いていること

## 起動手順

1. 開発したい Moodle を `src/` にクローンします。

公式リポジトリを使う場合の例:

```bash
git clone --branch <branch-or-tag> https://github.com/moodle/moodle.git src
```

自分の fork を使う場合は、URL を fork 側に置き換えてください。

2. `moodledata/` がまだ無い場合は作成します。

```bash
mkdir -p moodledata
```

3. `moodledata/` がコンテナ内の `www-data` ユーザーから書き込める状態であることを確認します。

```bash
sudo chown -R 33:33 moodledata
```

権限エラーを避けるためのローカル開発向けの簡易対応として、必要なら次でも動作します。

```bash
chmod -R 0777 moodledata
```

4. コンテナをビルドして起動します。

```bash
docker compose up -d --build
```

5. ブラウザで以下にアクセスします。

```text
http://localhost:8080
```

6. データベース volume が空の初回起動では、必要に応じて Moodle のインストーラを進めてください。

`src/config.php` がまだ無い場合は、Web インストーラで生成するか、`src/config-dist.php` をもとに作成してください。

## デフォルト接続情報

| 項目 | 値 |
| --- | --- |
| Moodle URL | `http://localhost:8080` |
| DB ホスト | `db` |
| DB 名 | `moodle` |
| DB ユーザー | `moodle` |
| DB パスワード | `moodlepass` |
| MariaDB root パスワード | `rootpass` |
| Moodle dataroot | `/var/www/moodledata` |
| Xdebug ポート | `9003` |

## よく使うコマンド

起動:

```bash
docker compose up -d
```

停止:

```bash
docker compose stop
```

停止してコンテナを削除:

```bash
docker compose down
```

Web コンテナのログ確認:

```bash
docker compose logs -f web
```

DB コンテナのログ確認:

```bash
docker compose logs -f db
```

Web コンテナに入る:

```bash
docker compose exec web bash
```

cron を手動実行する:

```bash
docker compose exec web php /var/www/moodle/admin/cli/cron.php
```

キャッシュを削除する:

```bash
docker compose exec web php /var/www/moodle/admin/cli/purge_caches.php
```

## デバッグ

- `web` コンテナでは Xdebug が有効です
- `XDEBUG_MODE=debug` が設定されています
- `client_host=host.docker.internal`、`client_port=9003` を使います

IDE 側は `9003` を待ち受ける設定にしてください。

## 注意点

- Apache のドキュメントルートは `src/public/` です
- `src/` はこのリポジトリでは管理していないため、Moodle のバージョン切り替えは `src/` の中身を差し替えて行います
- ホスト側でポート番号を変更した場合は、Moodle 側の `config.php` の `wwwroot` も同じ URL に合わせて更新してください

## トラブルシューティング

`moodledata` の書き込みエラーが出る場合:

- `moodledata/` の所有者または権限を見直してください
- コンテナ内では `www-data` が書き込みを行います

ポート `8080` が使用中の場合:

- `docker-compose.yml` のポート割り当てを変更してください
- 変更後は Moodle 側の `config.php` の `wwwroot` も同じ URL に合わせてください

環境を作り直したい場合:

```bash
docker compose down -v
docker compose up -d --build
```

`down -v` は DB volume を削除するため、保存済みデータは失われます。