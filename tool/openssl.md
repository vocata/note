# 自行签发TLS证书

> TLS协议是从SSL3.0协议演变而来的，但两者并不兼容，目前SSL协议已经逐渐被TLS所取代。

## 签发TLS/SSL证书过程

### 创建自签CA证书

1. 创建CA证书私钥

    ```bash
    # genrsa: 使用RSA算法生成私钥
    # -aes128: 一种对称加密算法，输入密码后，将RSA私钥进行了AES加密 (optional)
    # -out: 指定密钥文件名
    # 4096: RSA私钥长度
    openssl genrsa -aes128 -out ca.key 4096
    ```

2. 创建自签CA证书

    ```bash
    # req: 创建证书请求文件或者创建自签CA证书，取决于参数
    # -new: 创建请求文件
    # -x509: 创建自签CA证书，为标准的x509证书格式
    # -key: 指定私钥文件
    # -day: 指定自签CA证书有效期，单位天
    # -outform: 指定输出的CA证书格式，可选pem和der，默认为pem (optional)
    # -out: 指定CA证书文件名
    openssl req -new -x509 -key ca.key -days 365 -outform pem -out ca.crt
    ```

    如果ca.key设置了密码，这一步需要输入私钥密码

> 上述两个步骤可以合二为一
>
> ```bash
> openssl req -newkey rsa:4096 -x509 -day 365 -keyout ca.key -out ca.crt
> ```
>
> 注意：这种方式生成的ca私钥没法指定加密算法，默认使用的加密算法取决于openssl的版本。

### 使用自签CA证书签发TLS证书

1. 生成服务端私钥

    ```bash
    openssl genrsa -aes128 -out server.key 4096
    ```

    和创建CA证书私钥的过程一样

2. 创建证书请求文件

    ```bash
    openssl req -new -key server.key -out server.csr
    ```

    同样，如果server.key设置了密码，这一步需要输入私钥密码

3. 签发证书文件

```bash
# x509: 这个子命令有多种用途，这里用于签发数字证书
# -req: 表示期望输入文件类型为请求文件
# -in: 指定请求文件
# -CA: 指定用于签名的CA证书
# -CAKey: 指定用于签名的CA证书私钥，因为签名使用的是CA证书私钥
# -CAcreateserial: 生成CA序列号文件
# -out: 指定签发证书文件名
openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
```

### 校验证书是否有效

1. 查看证书信息

    ```bash
    openssl x509 -in server.crt -text -noout
    ```

2. 校验证书

    ```bash
    openssl verify -CAfile ca.crt server.crt
    ```
