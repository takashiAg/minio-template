# 使い方

S3 をローカルで建てる用
[github](https://github.com/takashiAg/minio-template)を読んでくれ

## 経緯(TL;DR)

S3 が世界標準になりつつある。
互換のあるシステムがたくさん出てきた。
ローカルで建てれるものが必要になる機会も増えた。
OSS の minio を使うことが増えた
そこでテンプレ化しましょう
[minio](https://min.io/)を呼んでください

## 依存

依存してるライブラリは以下です

- docker-compose
- docker

## 結論

以下の構成で動かすのが最小構成

```tree:tree
.
├── .env
└── docker-compose.yml
```

### 設定ファイル

以下のようにすると良い

```yaml:docker-compose.yml
version: "3"
services:
  minio:
    image: minio/minio:latest
    ports:
      - ${MINIO_PORT:-9000}:9000
      - ${MINIO_CONSOLE_PORT:-9001}:9001
    volumes:
      - ./.data/minio/data:/export
      - ./.data/minio/config:/root/.minio
    environment:
      MINIO_ACCESS_KEY: ${MINIO_ACCESS_KEY:-minio}
      MINIO_SECRET_KEY: ${MINIO_SECRET_KEY:-minio123}
    command: server /export --console-address ":9001"
  createbuckets:
    image: minio/mc
    depends_on:
      - minio
    entrypoint: >
      /bin/sh -c "
      until (/usr/bin/mc config host add myminio http://minio:9000 ${MINIO_ACCESS_KEY:-minio} ${MINIO_SECRET_KEY:-minio123}) do echo '...waiting...' && sleep 1; done;
      /usr/bin/mc mb myminio/${MINIO_BUCKET_NAME:-mybucket};
      /usr/bin/mc policy download myminio/${MINIO_BUCKET_NAME:-mybucket};
      exit 0;
      "

```

### 環境変数

.env ファイルを用意しましょう。
`MINIO_ACCESS_KEY` `MINIO_SECRET_KEY` は秘匿情報なので、適宜変えてください。
[password generator](https://www.graviness.com/app/pwg/)とかでも良いので。

```.env:.env
MINIO_ACCESS_KEY=bbFJoygjrP5BdqnQ
MINIO_SECRET_KEY=CAMHACH4bFlr0R2E
MINIO_BUCKET_NAME=mybucket
MINIO_PORT=9000
MINIO_CONSOLE_PORT=9001
```

## 起動

裏で実行する場合は`-d`をつけて！

```bash:bash
## cd /path/to/app

docker-compose up
## macの場合は以下
## docker compose up

## 裏で実行する場合は-d をつけて！
## docker-compose up -d
```
