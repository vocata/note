# 2.3 Dockerfile定制镜像

## 镜像构建

可以通过便携dockerfile文件来自动化构建镜像，以下是一个例子

```dockerfile
FROM ubuntu:focal

RUN apt-get update && \
    apt-get install -y \
    git \
    vim

CMD ["/bin/bash"]
```

编写完dockerfile后，可以使用 **docker build** 命令来构建镜像

```bash
# 使用当前目录的dockerfile
docker build -t ubuntu:example .

# 指定dockerfiles目录下的ubuntu_dockerfile文件作为此次构建的dockerfile
docker build -t ubuntu:example -f dockerfiles/ubuntu_dockerfile .
```

## 构建上下文

docker使用的是C/S的架构，docker-client通过将需要构建的上下文（也就是构建的目录）打包发送给docker-server，docker-server收到后会已这个上下文进行镜像构建。例如 **docker build .** 这条命令指定了当前目录为构建上下文，那么doker-client将会把当前目录打包发送给docker-server。而dockerfile文件中涉及到目录的命令全部只能在这个上下文中操作，比如说复制上下文中所有文件到/tmp目录

```dockerfile
COPY . /tmp/
```

这里的 **.** 表示的就是上下文，如果想指定上下文以外的目录，比如说复制/etc中的文件到/tmp目录，构建时会报错

```dockerfile
# 构建时会报错
COPY /etc/ /tmp/
```
