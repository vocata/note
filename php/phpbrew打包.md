# phpbrw打包


## 配置环境变量

```bash
mkdir -p $HOME/.local/bin && \
echo 'export PATH=$HOME/.local/bin:$PATH' >> $HOME/.bashrc && \
echo 'export PATH=$HOME/.composer/vendor/bin:$PATH' >> $HOME/.bashrc && \
exec $SHELL -l
```

## 安装依赖

```bash
apt-get update && DEBIAN_FRONTEND=noninteractive && apt-get install -y \
build-essential \
git \
make \
zip \
php7.4 \
php7.4-mbstring \
php7.4-curl \
php7.4-xml
```

## 安装composer v2.2

```bash
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');" && \
php composer-setup.php --install-dir=$HOME/.local/bin --filename=composer --2.2 && \
php -r "unlink('composer-setup.php');"
composer -V
```

## 安装 box
```bash
composer global require humbug/box && \
box -v
```

## 打包phpbrew
```bash
git clone https://github.com/vocata/phpbrew.git && \
cd phpbrew && make
```

## 安装phpbrew
```bash
cp phpbrew $HOME/.local/bin && \
phpbrew init && \
echo "[[ -s ~/.phpbrew/bashrc ]] && source ~/.phpbrew/bashrc" >> $HOME/.bashrc && \
exec $SHELL -l && \
phpbrew update && \
phpbrew known && \
phpbrew variants && \
phpbrew list
```

## 使用phpbrew
```bash
phpbrew --debug install --jobs=8 --stdout github:php/php-src@PHP-5.6.40 as php5.6 +default -openssl && \
phpbrew --debug install --jobs=8 --stdout 7.4 as php7.4 +default && \
phpbrew --debug install --jobs=8 --stdout 8.1 as php8.1 +default && \
phpbrew clean php5.6 && \
phpbrew clean php7.4 && \
phpbrew clean php8.1
```
