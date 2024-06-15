
# 重新编译的流程

## 说明

kbe 原来带的 curl 编译会失败，所以需要重新用官方的 curl 包。  下载地址是： https://curl.se/download/curl-7.61.1.tar.bz2 。

## 编译

注意：curl 编译要去掉很多东西，否则链接的时候会报各种找不到 xx 符号的错，上面的 without 跟 disable 就是为了解决 idn 报错，ldap 报错，brotli 报错。

```bash
./configure --without-libidn2 --disable-ldap --without-brotli 
make
```

## 替换静态库

拷贝编译出来的静态库 lib/.libs/libcurl.a 到 src/libs 目录    

```bash
cp ./lib/.libs/libcurl.a ../../../libs/
```
  