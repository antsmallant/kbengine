
how to install

---

# 0. 可供参考的文档

* 编译：[建立](https://kbengine.github.io//cn/docs/build.html)

* 安装：[kbengine安装指引](https://kbengine.github.io/cn/docs/installation.html) 

* demo 安装：[kbengine_unity3d_warring安装指引](https://github.com/antsmallant/kbengine_unity3d_warring)

---

# 1. 编译服务端

注意：kbe 是自带 python3 的。  

## 1.1 wsl2 下编译（ubuntu20.04 or ubuntu22.04)

1、官网下载 kbengine     
https://github.com/kbengine/kbengine

```
git clone https://github.com/kbengine/kbengine.git
```

2、安装依赖项     
```
sudo apt install gcc
sudo apt install g++
sudo apt install make
sudo apt install autoconf
sudo apt install aptitude
sudo apt install libtool
sudo apt install libssl-dev
sudo apt install libmysqlclient-dev
```

3、修改编译选项    
修改 kbe/src/build/common.bak，在 CXXFLAG 定义的地方，补上以下两句，可以解决 mysql 和 ssl 的报错：
```
CXXFLAGS += -Wno-format-truncation
CXXFLAGS += -DOPENSSL_API_COMPAT=0x10100000L
```

4、修改一些导致编译报错的代码    
* `kbe/src/lib/entitydef/datatype.cpp` 的第 2156 行 `dataType->decRef();` 要注释掉，因为 dataType 已经是 nullptr 了；   
*  `kbe/src/lib/entitydef/datatype.cpp` 的第 2314 行 `dataType->decRef();` 要注释掉，因为 dataType 已经是 nullptr 了；

5、重新编译 curl
1）重新下载
https://curl.se/download/curl-7.61.1.tar.bz2 ，解压并替换 src/lib/dependencies 下面的整个 curl 目录

要注意，把 curl 加入 git 的时候，需要 `git add curl -f`，通过 `-f` 强制加入所有内容，否则有些内容缺失，会导致编译失败的。  

2）重新编译
```
./configure --without-libidn2 --disable-ldap --without-brotli 
make
```

3）拷贝编译出来的静态库 lib/.libs/libcurl.a 到 src/libs 目录    
```
cp ./lib/.libs/libcurl.a ../../../libs/
```

注意：curl 编译要去掉很多东西，否则链接的时候会报各种找不到 xx 符号的错，上面的 without 跟 disable 就是为了解决 idn 报错，ldap 报错，brotli 报错。  

6、在 src 目录运行 make，注意：要先 chmod 755！  
```
cd src
chmod -R 755 .
make
```

7、可能的报错
1）如果在 wsl 下直接使用 windows 磁盘里项目文件，那么可能会由于 chmod 失败，而导致编译报错，最好就直接在 wsl 自己的文件系统下编译


## 1.2 windows 下编译（可选）

1、官网下载 kbengine     
https://github.com/kbengine/kbengine

```
git clone https://github.com/kbengine/kbengine.git
```

2、用 vs2019 打开 .sln 文件，直接生成解决方案

3、可能的报错

1）vs2019 编译的时候报大量的 winnt.h 的报错，这个问题在于 visual studio installer 安装了一些不兼容的 windows sdk 的可选包，其实只要包含通用 `windows 平台开发`  默认必选的那个 `windows 10 sdk (10.0.19041.0)`  就够了，其他更低版本的 windows 10 sdk 或 windows 11 sdk 也不要勾选，而 `python 开发` 只勾选一个 python3.7.8 的 x64 版本就够了


---


# 2. 运行 demo

## 2.1 创建数据库及数据库账号

1、连接上 mysql，创建一个名为 kbe 的数据库，以及名为 kbe 的账号

```
CREATE DATABASE IF NOT EXISTS kbe DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_general_ci;

create user 'kbe'@'localhost' identified by '123456';
grant all privileges on *.* to 'kbe'@'localhost';
create user 'kbe'@'%' identified by '123456';
grant all privileges on *.* to 'kbe'@'%';
flush privileges;

```

2、可能的报错
1）可能会连不上数据库，如果 dbmgr 连不上 mysql，在 my.cnf 的 [mysqld] 下面加上 `skip-ssl` ，然后重启 mysqlserver。 
2）不要使用官方的安装工具：installer.py。  


## 2.2 下载 demo

注意1：有很多 demo，但能正常跑的就这个 kbengine_unity3d_warring，而这个 kbengine_unity3d_demo 没啥用。  
注意2：不需要单独 checkout kbengine_demos_assets，那些 demo 里都会以 submodule 的方式引用这个的

1、下载 kbengine_unity3d_warring
地址： https://github.com/kbengine/kbengine_unity3d_warring
注意 checkout 之后，把子项目也拉下来 
```
git clone https://github.com/kbengine/kbengine_unity3d_warring.git
cd kbengine_unity3d_warring
git submodule update --init
```

2、替换 sdk plugin
1）进入 kbengine_demos_assets 目录，运行 gensdk.bat 生成最新的KBE客户端插件
2）拷贝 kbengine_unity3d_plugins 到 kbengine_unity3d_warring\Assets\Plugins\kbengine\  

（如果不替换，进入游戏就会提示要版本号不一致，要更新什么的）


## 2.3 修改 demo 配置（wsl跟windows都相同的操作）

1、把拉下来的 kbengine_unity3d_warring/kbengine_demos_assets 拷贝到 kbengine 目录下

2、修改 kbengine_demos_assets/res/server/kbengine.xml
1）在`<databaseInterfaces>` 处，修改 `<default>` 的内容，从 `kbe/res/server/kbengine_defaults.xml` 拷贝 `<databaseInterfaces>` 的 `<default>` 项覆盖上去；
2）修改 mysql 相关配置：端口改为实际值，比如： 3306；填上正确的账号和密码；encrypt 改为 false；host 改为 `127.0.0.1`（因为在 wsl 上用 localhost 会连接不上）；


## 2.3 wsl2 下运行服务器

1、进入 kbengine_demos_assets，运行 `start_server_background.bat`


## 2.4 windows 下运行服务器（可选）

1、进入 kbengine_demos_assets，运行 `start_server.sh`


## 2.5 unity 运行客户端

1、这个要用 unity4.3.2f 运行，不需要像指引中说的那样用 webplayer，直接在 unity 上运行就行
（unity4.3.2f 可以在百度网盘上找到，我的网盘/software/unity/unity4.3.2）

2、build settings 用 `PC, Mac &Linux Standalone`，然后运行就 OK 了

3、可能的报错
1）如果 unity 报 **error CS1010: Newline in constant** 或者 **unexpect symbol** 等奇怪的错，将报错的文件以编码 `utf8 with bom`  另存即可。主要就两个文件：`Assets\Plugins\kbengine\kbengine_unity3d_plugins\ServerErrorDescrs.cs` ，
`Assets\Plugins\kbengine\kbengine_unity3d_plugins\EntityComponent.cs`


---

# 3. 运行 webconsole

这个要在 linux 下运行，windows 下总有问题。  

1、django 要使用很旧的版本，根据操作说明（kbe/tools/server/webconsole/WebConsole_Guide.pdf），使用 python3.x + django-1.8.9，进入 “kbe/tools/server/django_packages” 目录，并在该目录下解压 Django-1.8.9.tar.gz 文件（解压到当前目录）

2、创建一个干净的 python 环境
否则已有的新的 django site_package 会导致不使用旧的 django 包

```
conda create -n kbe python=3.8 -y
conda activate kbe
```

3、运行 sync_db.sh 完成 db 的初始化

4、运行 run_server.sh 
（如果报错，提示端口被占，就改一下 run_server.sh 里面的端口号）

5、浏览器打开 http://127.0.0.1:8001/wc

6、新建一个账号，填上相应的信息，再使用这个新账号登录，就可以看到服务器集群的信息了
注意，在 linux 下，操作系统账号填 linux 用户名，uid 填 linux 的 uid（可通过 id 命令查看）


---

# 4. 注意事项

1、官方提供的 installer.py 总会报错的，不要使用，手动配置 demo_assets 里面的数据库配置就 ok 了。  

---

# 5. 安装 mysql 

wsl 或 linux 下可以使用 docker 安装 mysql，会更方便很多。

1、在 kbengine_demos_assets 创建一个目录 env_tool

2、创建以下文件

docker-compose.yml

```
version: '3'
services:
mysql:
	image: mysql:5.7.39-debian
	ports:
	- 3306:3306
	environment:
	MYSQL_ROOT_PASSWORD: 123456
```
  
  start_env.sh
  
```bash
#!/bin/bash
docker-compose up -d
```
 
  stop_env.sh
  
```bash
#!/bin/bash
docker-compose stop
```
  
  env/mysql_init.sql
  
```sql
CREATE DATABASE IF NOT EXISTS kbe DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_general_ci;

create user 'kbe'@'localhost' identified by '123456';
grant all privileges on *.* to 'kbe'@'localhost';
create user 'kbe'@'%' identified by '123456';
grant all privileges on *.* to 'kbe'@'%';
flush privileges;
```

   init_mysql.sh
   
```bash
#!/bin/bash
mysql -h127.0.0.1 -P3306 -uroot -p123456 < ./env/mysql_init.sql
```
  
  conn_mysql_by_root.sh
```bash
#!/bin/bash
mysql -h127.0.0.1 -P3306 -uroot -p123456
```

3、使用方法

1）启动 docker 服务
```
sudo service docker start
```

2）启动数据库
```
./start_env.sh
```

3）停掉数据库
```
./stop_env.sh
```

4）首次运行时创建 kbe 数据库和账号
```
./init_mysql.sh
```

---