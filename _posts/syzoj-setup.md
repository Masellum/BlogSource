---
title: "SYZOJ 部署指南"
date: 2018-08-21 09:56:43
tags: 
  - SYZOJ
  - Linux
categories: 其他
summary: ' '
---

<!-- more -->

// 2018/10/30 更新：发现竟然有人看这篇博客……三月份写的部署指南在十月被发现全都是错漏……（哭泣）于是进行了小的修订，这次应该（暂时）没有内容上的错误了，也就是说如果没有其他诡异的意外因素，这样部署应该就是能跑起来的。

[SYZOJ](https://github.com/syzoj/) 是主要由 [Menci](https://github.com/Menci) 与 [t123yh](https://github.com/t123yh) 开发的在线评测系统，具有界面美观、后台管理可视化等优点，为许多学校用于搭建校内OJ，并被 [Libre OJ](https://loj.ac/) 使用。但由于开发者时间紧张，长期没有完善安装文档，其 GitHub 主页上的安装指南已严重过期，网上能搜索到的安装指南也要么过于老旧要么过于简略。在经历过三天痛苦的摸索后，我终于成功地部署了 SYZOJ。为了记录自己的部署历程也为了不让后来人重复这一过程，我决定依我浅见写一篇 SYZOJ 部署指南。

本文部署环境为 Ubuntu 16.04，使用的 Node.js 版本为v8.9.3，npm 版本为5.5.1，使用其他版本有可能出现不可解决的 Bug。如果想要在部署 SYZOJ 的同时不影响本机的 Node.js 版本，可以使用 [nvm](https://github.com/creationix/nvm) 管理多个 Node.js 版本。关于 SYZOJ2 在 CentOS 上的搭建，请参考 Victor Huang 的 [SYZOJ2 在 CentOS 上的搭建](https://imvictor.tech/posts/deploying-syzoj2-on-centos/)。

因为我对于许多 Linux 相关知识并不了解，本文中有很多地方是带着”能用就行“的似是而非的感觉写的，请辩证参考。本文中没有提及 SYZOJ2 的邮件验证系统的相关内容。

本文参考 t123yh 写的 [SYZOJ 搭建指南](https://blog.t123yh.xyz:2/index.php/archives/236) 和不是 t123yh 写的 [SYZOJ 中文安装指南](https://blog.csdn.net/buyaoxx/article/details/77504043)。

## 概述

SYZOJ 由 web 端 和 judge 端构成，其中 judge 端又由 daemon、frontend、runner 三个部分组成，每个部分都能同时启动多个实例。想要部署 SYZOJ 需要至少四个进程的相互配合。

### daemon

daemon 主要负责获取评测任务，并将评测任务分散为若干个较为轻量的子任务（编译和评测），这些任务再由 runner 来执行。

### runner

runner 是实际执行评测任务的部分。

### frontend

frontend 是 daemon 与 Web 端沟通的桥梁，它使用 HTTP 协议接收 Web 端传送来的评测任务，并将其推入消息队列供 daemon 部分执行。同时，它还负责向客户端推送评测进度。

## 部署步骤

### 获取代码

从 GitHub 上分别获取 web 端和 judge 端的源代码。

```shell
git clone https://github.com/syzoj/syzoj.git /var/syzoj-web
git clone https://github.com/syzoj/judge-v3.git /var/syzoj-judge-v3
```

本文的部署过程中将 web 端放在 `/var/syzoj-web` 目录，judge 端放在 `/var/syzoj-judge-v3` 目录。

从 [这个链接](https://seafile.t123yh.xyz:2/f/65f061a56f414b3db478/) 下载 sandbox-rootfs，使用 root 权限解压到根目录下。

### 安装依赖项

```shell
apt install build-essential libboost1.58-all-dev nodejs rabbitmq-server redis-server nginx p7zip-full
cd /var/syzoj-web
npm i
cd /var/syzoj-judge-v3
npm i
```

SYZOJ 使用的 simple-sandbox 需要使用 boost 库。

// 对于 Ubuntu 18.04，请将第一行中的 `libboost1.58-all-dev` 替换为 `libboost-all-dev`。

SYZOJ 使用 RabbitMQ 消息队列实现各个进程之间的通信，使用 Redis 实现多个 runner 共享一份可执行程序。

你需要分别在 web 端目录和 judge 端目录使用 npm 安装项目的依赖。npm 使用国内镜像源加速的方法不再赘述，读者可自行了解。注意使用 root 用户执行 `npm i` 时应加上 `--unsafe-perm` 参数。

### 安装与配置数据库

SYZOJ 默认使用 sqlite3 作为数据库。这里我们改用更强大也更普遍的 MySQL。MySQL 的安装不再赘述。

打开 MySQL cli，输入以下代码，注意自行将中括号中内容替换为你的自定义内容。

```mysql
CREATE DATABASE syzoj;
ALTER DATABASE syzoj CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
CREATE USER 'syzoj'@'localhost' IDENTIFIED BY '[your_password]';
GRANT ALL ON syzoj.* TO 'syzoj'@'localhost';
FLUSH PRIVILEGES;
```

这些代码创建了名为`syzoj`的数据库，创建了名为 `syzoj` 的数据库用户并更改了数据库权限。

### 启用内存控制相关的内核选项

simple-sandbox 使用了 cgroup 来进行内存用量控制，该过程需要用到 memory.memsw（控制内存和交换空间的总用量）。一些 Linux 发行版的内核在默认情况下，没有启用这个内核选项，你需要手动启用它。

检查文件 `/sys/fs/cgroup/memory/memory.memsw.usage_in_bytes` 是否存在。如果不存在，你需要使用你的 grub 启用。
打开 `/etc/default/grub`，找到其中的

```shell
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

这一行（引号内内容可能有不同），在其后添加 `cgroup_enable=memory swapaccount=1`。以上述为例，添加后该行应为

```shell
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash cgroup_enable=memory swapaccount=1"
```

修改后执行

```shell
update-grub && reboot
```

来更新 grub 配置并重启。重启后，重新检查文件 `/sys/fs/cgroup/memory/memory.memsw.usage_in_bytes` 是否存在。

### 构建源代码

SYZOJ 部分使用了 TypeScript。你需要手动将 TypeScript 编译成 JavaScript。

```shell
cd /var/syzoj-judge-v3
npm run build
```

### 复制并修改配置文件

SYZOJ 可以指定运行所用配置文件的位置。为了便于统一管理，建议将配置文件统一放在 `/etc/syzoj-config` 目录下。

SYZOJ 提供了配置文件示例，我们首先将示例复制过去。

```shell
mkdir /mnt/syzoj-data /mnt/syzoj-bin /mnt/syzoj-tmp1 /etc/syzoj-congig
cd /var/syzoj-judge-v3
cp daemon-config-example.json /etc/syzoj-config/daemon.json
cp runner-shared-config-example.json /etc/syzoj-config/runner-shared.json
cp runner-instance-config-example.json /etc/syzoj-config/runner-instance1.json
cp frontend-config-example.json /etc/syzoj-config/frontend.json
cp /var/syzoj-web/config-example.json /etc/syzoj-config/web.json
```

SYZOJ 的 `runner` 可以有多个配置不同的实例，这里我们只复制一份配置文件建立一个实例。接下来你需要按照自己的需求修改配置文件。这里给出一份示例。JSON 标准中没有注释，所以复制下面的配置文件的话请删除注释。

`config.json`

```json
{
  "title": "SYZOJ", // OJ名称
  "hostname": "127.0.0.1", // web 端 IP（也就是 web 服务器监听的 IP），对外开放请修改为对外 IP
  "port": "5283", // web 端端口
  "db": {
    // 如果使用 sqlite3 则不用修改默认配置
    // 因为我们换用了 MySQL 所以需要进行一定修改
    "database": "syzoj", // MySQL 数据库名
    "username": "syzoj", // MySQL 数据库用户名
    "password": "[your_password]", // MySQL 数据库用户密码
    "host": "localhost", // MySQL 数据库地址
    "dialect": "mysql", // 此处更改为 MySQL
    "storage": null
  },
  "register_mail": false, // 邮箱验证，自行配置，这里为了部署方便设为 false 以关闭
  "email": {
    "method": "aliyundm",
    "options": {
      "AccessKeyId": "xxxx",
      "AccessKeySecret": "xxxx",
      "AccountName": "xxxx"
    }
  },
  "upload_dir": "uploads", // 上传文件路径
  "default": { // 默认配置
    "problem": { // 题目时间限制与空间限制
      "time_limit": 1000,
      "memory_limit": 256
    },
    "user": { // 用户默认展示，初始 rating 为1500
      "show": true,
      "rating": 1500
    }
  },
  "sorting": { // 排序方法
    "ranklist": { // 排名默认为按 rating 降序排序
      "field": "rating", // 更改为 "ac_num" 以按 AC 数排序
      "order": "desc"
    },
    "problem": { // 题目默认以题目编号排序
      "field": "id",
      "order": "asc"
    }
  },
  "limit": { // 最大限制
    "time_limit": 10000,
    "memory_limit": 1024,
    "data_size": 209715200,
    "testdata": 209715200,
    "submit_code": 102400,
    "submit_answer": 10485760,
    "custom_test_input": 20971520,
    "testdata_filecount": 5
  },
  "page": { // 各个页面的显示数量限制
    "problem": 50,
    "problem_statistics": 10,
    "judge_state": 10,
    "ranklist": 20,
    "discussion": 10,
    "article_comment": 10,
    "contest": 10,
    "edit_contest_problem_list": 10,
    "edit_problem_tag_list": 10
  },
  "languages": { // 语言配置
    "cpp": {
      "show": "C++",
      "highlight": "cpp",
      "version": "GCC 5.4.0",
      "editor": "c_cpp"
    },
    "cpp11": {
      "show": "C++11",
      "highlight": "cpp",
      "version": "GCC 5.4.0",
      "editor": "c_cpp"
    },
    "csharp": {
      "show": "C#",
      "highlight": "csharp",
      "version": "MCS 4.8.0.0, Mono 4.8.0",
      "editor": "csharp"
    },
    "c": {
      "show": "C",
      "highlight": "c",
      "version": "GCC 5.4.0",
      "editor": "c_cpp"
    },
    "vala": {
      "show": "Vala",
      "highlight": "vala",
      "version": "Vala 0.30.1, GCC 5.4.0",
      "editor": "vala"
    },
    "java": {
      "show": "Java",
      "highlight": "java",
      "version": "GCC 5.4.0",
      "editor": "java"
    },
    "pascal": {
      "show": "Pascal",
      "highlight": "pascal",
      "version": "FPC 3.0.0",
      "editor": "pascal"
    },
    "lua": {
      "show": "Lua",
      "highlight": "lua",
      "version": "Lua 5.2.4",
      "editor": "lua"
    },
    "luajit": {
      "show": "LuaJIT",
      "highlight": "lua",
      "version": "LuaJIT 2.0.4",
      "editor": "lua"
    },
    "python2": {
      "show": "Python 2",
      "highlight": "python",
      "version": "CPython 2.7.12",
      "editor": "python"
    },
    "python3": {
      "show": "Python 3",
      "highlight": "python",
      "version": "CPython 3.5.2",
      "editor": "python"
    },
    "nodejs": {
      "show": "Node.js",
      "highlight": "js",
      "version": "7.7.3",
      "editor": "javascript"
    },
    "ruby": {
      "show": "Ruby",
      "highlight": "ruby",
      "version": "2.3.1",
      "editor": "ruby"
    },
    "haskell": {
      "show": "Haskell",
      "highlight": "haskell",
      "version": "GHC 7.10.3",
      "editor": "haskell"
    },
    "ocaml": {
      "show": "OCaml",
      "highlight": "ocaml",
      "version": "Ocamlbuild 4.02.3",
      "editor": "ocaml"
    },
    "vbnet": {
      "show": "Visual Basic",
      "highlight": "vbnet",
      "version": "VBNC 0.0.0.5943, Mono 4.8.0",
      "editor": "vbscript"
    }
  },
  "links": [ // 友链
    {
      "title": "LibreOJ",
      "url": "https://loj.ac/"
    }
  ],
  "session_secret": "233", // 加密 Token，自行更改，建议复杂一些
  "judge_server_addr": "http://127.0.0.1:5284", // judge 端的地址^
  "judge_token": "233", // 若要修改此项请将 judge 端的 web_token 一同修改
  "email_jwt_secret": "test" // 应该是邮件相关设置
```

*^如果想要实现用户浏览器端实时获取评测状态的功能，请将 `"judge_server_addr"` 字段改为 OJ 的对外地址。参考文首提到的[SYZOJ2 在 CentOS 上的搭建](https://imvictor.tech/posts/deploying-syzoj2-on-centos/)。*

`daemon.json`

```json
{
  "RabbitMQUrl": "amqp://localhost/", // RabbitMQ 服务器的路径
  "RedisUrl": "redis://127.0.0.1:6379", // Redis 服务器的路径
  "TestData": "/mnt/syzoj-data/uploads/testdata", // 测试数据存放目录
  "Priority": 1,
  "DataDisplayLimit": 100, // 数据显示限制
  "TempDirectory": "/tmp"
}
```

`frontend.json`

```json
{
  "RabbitMQUrl": "amqp://localhost/", // RabbitMQ 服务器地址
  "Listen": {
    "host": "127.0.0.1", "port": 5284
  }, // judge 端的地址与端口
  "RemoteUrl": "http://127.0.0.1:5283", // web 端的地址
  "Token": "233" // 与 web.json 中的 web_token 相同
}
```

`runner-shared.json`

```json
{
  "RabbitMQUrl": "amqp://localhost/",
  "RedisUrl": "redis://127.0.0.1:6379",
  "TestData": "/mnt/syzoj-data/uploads/testdata", // 与 daemon.json 中的 TestData 相同
  "Priority": 1,
  "DebugMessageDisplayLimit": 5000,
  "OutputLimit": 104857600,
  "StderrDisplayLimit": 5120,
  "DataDisplayLimit": 128,
  "CompilerMessageLimit": 50000,
  "SpjTimeLimit": 1501,
  "SpjMemoryLimit": 256, // 以上几行为评测信息显示限制
  "SandboxEnvironments": [ // sandbox 内的环境变量
    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    "HOME=/tmp"
  ],
  "SandboxUser": "nobody",
  "SandboxRoot": "/sandbox-rootfs", // sandbox 所在目录
  "BinaryDirectory": "/mnt/syzoj-bin" // runner 所要执行的可执行文件的目录
}
```

`runner-instance1.json`

```json
{
  "WorkingDirectory": "/mnt/syzoj-tmp1", // 工作目录
  "SandboxCgroup": "syzoj-1" // sandbox 相关
}
```

### 创建 systemd 系统服务

分别在 `/etc/systemd/system/` 目录下建立以下文件并写入以下内容。需要注意的是如果你是用 nvm 或其他方式安装了 Node.js，`node` 可执行文件的位置可能会有不同，请自行修改。

`syzoj-web.service`

```
[Unit]
Description=SYZOJ Online Judge
After=network.target mysql.service
Requires=mysql.service

[Service]
Type=simple
WorkingDirectory=/var/syzoj-web
User=syzoj
Group=syzoj
ExecStart=/usr/bin/node /var/syzoj-web/app.js -c /etc/syzoj-config/web.json

[Install]
WantedBy=multi-user.target
```

`syzoj-judge-daemon.service`

```
[Unit]
Description=SYZOJ Daemon
After=network.target mysql.service rabbitmq-server.service redis-server.service
Requires=mysql.service rabbitmq-server.service redis-server.service

[Service]
Type=simple
WorkingDirectory=/var/syzoj-judge-v3
User=syzoj
Group=syzoj
ExecStart=/usr/bin/node /var/syzoj-judge-v3/lib/daemon/index.js -c /etc/syzoj-config/daemon.json

[Install]
WantedBy=multi-user.target
```

`syzoj-judge-frontend.service`

```
[Unit]
Description=SYZOJ Judge Frontend
After=network.target rabbitmq-server.service
Requires=syzoj-web.service rabbitmq-server.service

[Service]
Type=simple
WorkingDirectory=/var/syzoj-judge-v3
User=syzoj
Group=syzoj
ExecStart=/usr/bin/node /var/syzoj-judge-v3/lib/frontend-syzoj/index.js -c /etc/syzoj-config/frontend.json

[Install]
WantedBy=multi-user.target
```

`syzoj-judge-runner@.service`

```
[Unit]
Description=SYZOJ Runner %I
After=network.target rabbitmq-server.service redis-server.service
Requires=rabbitmq-server.service redis-server.service

[Service]
Type=simple
WorkingDirectory=/var/syzoj-judge-v3
User=root
Group=root
ExecStart=/usr/bin/node /var/syzoj-judge-v3/lib/runner/index.js -s /etc/syzoj-config/runner-shared.json -i /etc/syzoj-config/runner-instance%i.json

[Install]
WantedBy=multi-user.target
```

欲启用系统服务，请在终端中键入以下命令：

```shell
systemctl enable syzoj-web
systemctl enable syzoj-judge-daemon
systemctl enable syzoj-judge-frontend
systemctl enable syzoj-judge-runner@1
```

### 创建 syzoj 系统用户

```shell
useradd syzoj
mkdir /home/syzoj
chown -R syzoj:syzoj /mnt/syzoj-data
chown -R syzoj:syzoj /mnt/syzoj-bin
chown -R syzoj:syzoj /mnt/syzoj-tmp1
chown -R syzoj:syzoj /var/syzoj-web
chown -R syzoj:syzoj /var/syzoj-judge-v3
chown -R syzoj:syzoj /home/syzoj
```

### 配置临时目录

SYZOJ 使用 tmpfs 挂载临时目录。你需要在 `/etc/fstab` 文件中加入以下内容。

`/etc/fstab`

```
# The following items are for SYZOJ Judge:
tmpfs /mnt/syzoj-tmp1 tmpfs nodev,nosuid,size=384M 0 2
tmpfs /mnt/syzoj-bin tmpfs nodev,nosuid,size=1280M 0 2
tmpfs /mnt/syzoj-tmp2 tmpfs nodev,nosuid,size=384M 0 2
```

`/etc/fstab` 文件会在重启后生效。如果你不想立即重启，也可以自行挂载。具体方式请自行探索，这里不再赘述。

### 配置 Nginx 反向代理

SYZOJ web 端的默认端口是5283。想要从外部网络不加端口地访问，你需要配置 Nginx 以将5283端口反向代理到80端口。

在 `/etc/nginx/sites-available` 目录下建立名为 `syzoj` 和 `syzoj-judge` 的文件。中括号中内容请自行替换，作为参考，中括号处在官方示例文件中分别为 `syzoj-demo.t123yh.xyz`与 `syzoj-demo-judge.t123yh.xyz`。这里没有配置 ssl，感兴趣的读者可以自行配置。

`syzoj`

```
server {
	listen 80;
	server_name [your_domain_or_ip];

	location / {
		proxy_set_header Host $http_host;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header X-Real-IP $remote_addr;

		proxy_pass http://127.0.0.1:5283;
	}
}
```

`syzoj-judge`

```
server {
	listen 80;
	server_name [your_judge_domain];

	location / {
		proxy_set_header Host $host;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header X-Real-IP $remote_addr;

		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";

		proxy_pass http://127.0.0.1:5284;
	}
}
```

之后，在终端中输入以下命令以启用刚刚创建的配置文件

```shell
ln -s /etc/nginx/sites-available/syzoj /etc/nginx/sites-enabled/syzoj
ln -s /etc/nginx/sites-available/syzoj-judge /etc/nginx/sites-enabled/syzoj-judge
nginx -s reload
```

### 在网站注册用户并设置为管理员

此时 SYZOJ 已部署完毕，你可以尝试启动。

启动的方式有几种。如果为了方便在部署测试中查看输出日志，你可以使用 `screen` 命令打开四个会话并分别后台运行 web、daemon、frontend 和 runner。

四个进程的启动命令分别是

```shell
node /var/syzoj-web/app.js -c /etc/syzoj-config/web.json
node /var/syzoj-judge-v3/lib/frontend-syzoj/index.js -c /etc/syzoj-config/frontend.json
node /var/syzoj-judge-v3/lib/runner/index.js -s /etc/syzoj-config/runner-shared.json -i /etc/syzoj-config/runner-instance1.json
```

`screen` 命令的使用方式不再赘述。更好的方式是使用刚才建立好的系统服务。执行

```shell
systemctl start syzoj-web
systemctl start syzoj-judge-daemon
systemctl start syzoj-judge-frontend
systemctl start syzoj-judge-runner@1
```

执行后，如果启动成功，`ps-ef | grep node` 应该可以看到你的四个进程。如果没有看到，那么请自行查看日志或考虑使用 `screen` 进行调试。

如果启动成功，那么此时访问 `http://服务器ip或域名` 应该能打开 SYZOJ 的默认界面。你应该注册一个新用户并将其设置为管理员。

注册后，进入 MySQL Cli，键入以下命令，自行替换中括号内容。

```mysql
USE syzoj;
UPDATE `user` SET `is_admin` = 1 WHERE `username` = '[your_username]';
```

刷新页面，应该能看到你的账户已经成了全站管理员。SYZOJ 的全站管理员权限只能通过更改数据库的方式添加或删除。

这时你可以加入题目并提交以测试评测端是否运行正常。如果成功评测并显示结果，那么恭喜你，你的 SYZOJ 部署成功。注意如果添加过题目又删除，那么自动增长的默认题目编号不会排除被删除的题目，所以请谨慎添加。如果确实添加错了题目，可以考虑将下一道新题目通过编辑错误题目的方式添加。

### 问题排查

**这都是我遇到过的坑**

如果浏览器显示你的服务器拒绝了请求，那么有可能是 web 端未能正常运行。考虑重启 web 端。

如果显示 502 Bad Gateway，请访问 `http://服务器地址:5283/` 。如果成功打开，那么可能是 Nginx 的反向代理设置错误。

如果在提交后评测状态始终为 Waiting，请先刷新页面。如果刷新多次后仍为 Waiting，那么可以猜测评测端出了问题。请逐个排查 daemon、frontend 与 runner。

如果运行 runner 后显示

```
WARNING: You are likely using a version of node-tar or npm that is incompatible with this version of Node.js.
```

有可能是 Node.js 的版本问题。请安装文首推荐的 Node.js 版本重试。

如果运行 frontend 后出现两行找不到文件的错误信息，请考虑 `cd` 到 `/var/syzoj-judge-v3`下后再执行

```shell
node /var/syzoj-judge-v3/lib/frontend-syzoj/index.js -c /etc/syzoj-config/frontend.json
```

### 结语

到这里，SYZOJ 的部署应该就结束了，您可以进一步自定义设置或自行定制 SYZOJ 系统本身。如果想进一步折腾，可以考虑在 docker 中部署 SYZOJ，方便重新部署、备份数据。（如果你成功了，希望你向开发者提交实现方式或者发布打包好的 image 来造福社会（

感谢 Menci 与 t123yh 开发了如此方便易用的OJ并开源。

Finita la comedia.