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
$ git submodule init
$ git submodule update --recursive --remote
$ cp .env-dist .env
$ docker-compose up -d
```

### 修改配置文件

```
$ docker exec api php yii generate/key true
```

根据终端输出的结果，手动修改 `.env` 文件中的 `COOKIE_VALIDATION_KEY` 和 `JWT_SECRET` 值。

然后再次重启 Docker

```
$ docker-compose stop && docker-compose up -d
```