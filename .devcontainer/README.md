# 開発用コンテナ

## ビルド

### Linux, Mac

docker compose build --build-arg UID=$(id -u) --build-arg GID=$(id -g)

### Windows

docker compose build --build-arg UID=1000 --build-arg GID=1000

## 起動

docker compose up -d
