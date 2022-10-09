# 编译PHP5.6

## 环境

- ubuntu-20.04

## 依赖

```bash
apt-get update && apt-get install -y vim wget make libtool-bin
```

## 安装openssl和curl

准备工作

```bash
# 切换到home目录
cd $HOME
# 创建build和src目录，.local/xxx存放编译产物，src存放需要进行编译的代码
mkdir -p .local/openssl .local/curl .local/php src
```

### 编译&安装openssl-1.0.2u

```bash
cd $HOME/src && mkdir openssl
wget -O OpenSSL_1_0_2u.tar.gz https://github.com/openssl/openssl/archive/refs/tags/OpenSSL_1_0_2u.tar.gz
tar xfzv OpenSSL_1_0_2u.tar.gz --strip-components 1 -C ./openssl && cd openssl
./config -fPIC shared --prefix=$HOME/.local/openssl
make -j$(nproc) && make install
# export PKG_CONFIG_PATH=$HOME/.local/openssl/lib/pkgconfig
export LD_LIBRARY_PATH=$HOME/.local/openssl/lib
```

### 编译&安装curl-7.85.0

```bash
cd $HOME/src && mkdir curl
wget -O curl-7.85.0.tar.gz https://github.com/curl/curl/releases/download/curl-7_85_0/curl-7.85.0.tar.gz
tar xfzv curl-7.85.0.tar.gz --strip-components 1 -C curl && cd curl
autoreconf -if
./configure \
    --with-openssl=$HOME/.local/openssl \
    --prefix=$HOME/.local/curl
make -j$(nproc) && make install
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/.local/curl/lib
```

## 编译&安装php-5.6.40

```bash
cd $HOME/src && mkdir php
wget -O php-5.6.40.tar.gz https://www.php.net/distributions/php-5.6.40.tar.gz
tar xfzv php-5.6.40.tar.gz --strip-components 1 -C php && cd php
./configure \
    --prefix=$HOME/.local/php \
    --with-config-file-path=$HOME/.local/php/etc \
    --with-curl=$HOME/.local/curl \
    --with-openssl-dir=$HOME/.local/openssl \
    --with-openssl=$HOME/.local/openssl
make -j$(nproc) && make install
```

## 清理

```bash
rm -rf $HOME/src
```
