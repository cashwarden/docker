# CashWarden Docker 部署

## 部署

### 准备环境

### 安装 Docker

中国版安装

```shell
sudo curl -sSL https://get.daocloud.io/docker | sh
sudo curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://d52bcda9.m.daocloud.io
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

换中国源

```shell
vim /etc/docker/daemon.json
```

添加代码：

```json
{
    "registry-mirrors": [
        "https://dockerhub.azk8s.cn",
        "https://docker.mirrors.ustc.edu.cn"
    ]
}
```

重启服务

```
sudo systemctl daemon-reload && sudo systemctl restart docker
```

### 准备代码

```
$ git clone https://github.com/cashwarden/docker.git cashwarden-docker
$ cd cashwarden-docker
$ git clone https://github.com/cashwarden/web.git web
$ git clone -b gh-pages https://github.com/cashwarden/web.git web/dist
$ git clone https://github.com/cashwarden/api.git api
$ cp .env-dist .env
$ docker-compose up -d
```

### 修改配置文件

```
$ docker exec api php yii generate/key true
```

根据终端输出的结果，手动修改 `.env` 文件中的 `COOKIE_VALIDATION_KEY` 和 `JWT_SECRET` 值。

⚠️ 另外

- 要修改 `.env` 中 `APP_URL` 的值，使用 Telegram 记账的时候要用到。`APP_URL=https://域名/api` ，必须使用 HTTPS 协议。
- `YII_DEBUG` 和 `YII_ENV` 的值请根据情况修改。


然后再次重启 Docker

```shell
$ docker-compose stop && docker-compose up -d
```

访问 <http://localhost:81> 就可以使用了。

### HTTPS

提供一个方案（Ubuntu 并且已经有 Nginx），可以使用 [Certbot 安装](https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx) HTTPS:

```shell
sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

配置 nginx 代理：

```shell
vim /etc/nginx/sites-available/cashwarden
```

```
server {
    charset utf-8;
    client_max_body_size 128M; ## listen for ipv4
    server_name cashwarden.com www.cashwarden.com;

    location / {
        proxy_pass http://localhost:81;
    }
}
```

全自动安装：

```shell
sudo certbot --nginx -d cashwarden.com -d www.cashwarden.com
```

配置定时任务：

```shell
sudo certbot renew --dry-run
```

### 重新 build

更新代码之后可能需要重新 build

```shell
$ docker-compose build --no-cache web api
// 或者
$ docker-compose up --build -d web api
```

### 修改配置文件

修改 `.env` 文件之后要执行：

```shell
docker-compose up -d
```

### 使用 Telegram 记账功能

```shell
docker exec api php yii init/telegram "/telegram/hook"
```
