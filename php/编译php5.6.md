# 编译PHP5.6

## 环境

- ubuntu-20.04

## 依赖

```bash
apt-get update && apt-get install -y vim wget make libtool libtool-bin
```

## 安装openssl和curl

准备工作
```bash
# 切换到home目录
cd $HOME
# 创建build和src目录，build存放编译产物，src存放需要进行编译的代码
mkdir build src
```

### 编译&安装openssl-1.0.2u

```bash
cd $HOME/src && mkdir openssl
wget -O OpenSSL_1_0_2u.tar.gz https://github.com/openssl/openssl/archive/refs/tags/OpenSSL_1_0_2u.tar.gz
tar xfzv OpenSSL_1_0_2u.tar.gz --strip-components 1 -C ./openssl && cd openssl
./config -fPIC shared --prefix=$HOME/build
make -j$(nproc) && make install
```

### 编译&安装curl-7.85.0

```bash
cd $HOME/src && mkdir curl
wget -O curl-7.85.0.tar.gz https://github.com/curl/curl/releases/download/curl-7_85_0/curl-7.85.0.tar.gz
tar xfzv curl-7.85.0.tar.gz --strip-components 1 -C curl && cd curl
authreconf -if
./configure \
    --with-openssl=$HOME/build \
    --prefix=$HOME/build
make -j$(nproc) && make install
# 如果需要运行编译出来的curl，需要添加openssl的动态库目录，这里只用到了curl的动态链接库，所以可以忽略
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/build/lib
```

## 编译&安装php-5.6.40

```bash
cd $HOME/src && mkdir php
wget -O php-5.6.40.tar.gz https://www.php.net/distributions/php-5.6.40.tar.gz
tar xfzv php-5.6.40.tar.gz --strip-components 1 -C php && cd php
./configure \
    --prefix=$HOME/build/php \
    --with-config-file-path=$HOME/build/php/etc \
    --with-curl=$HOME/build \
    --with-openssl-dir=$HOME/build \
    --with-openssl \
    --with-pdo-mysql \
    --with-libdir=lib64
make -j$(nproc) && make install
```
