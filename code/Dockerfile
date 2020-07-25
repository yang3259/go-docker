FROM golang:1.14 AS builder

# 为我们的镜像设置必要的环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

ENV GOPROXY https://mirrors.aliyun.com/goproxy/

# 移动到工作目录：/build
WORKDIR /build

# 复制项目中的 go.mod 和 go.sum文件并下载依赖信息
ENV CODE_PATH .

ADD ${CODE_PATH}/discount/go.mod ./go.mod
ADD ${CODE_PATH}/discount/go.sum ./go.sum
RUN go mod download

# 将代码复制到容器中
ADD ${CODE_PATH}/discount ./discount
WORKDIR /build/discount
RUN go mod download
# 将我们的代码编译成二进制可执行文件 bubble
RUN go build -tags netgo -o bubble .

###################
# 接下来创建一个小镜像
###################
FROM debian:stretch-slim
WORKDIR /app

COPY ./wait-for-it.sh ./


# 从builder镜像中把/dist/app 拷贝到当前目录
COPY --from=builder /build/discount/bubble /app/
COPY --from=builder /build/discount/config /app/config

# 更换源
ARG CHANGE_SOURCE=true
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list && \
    sed -i 's/security-cdn.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list \
;fi

# 执行安装根证书命令
RUN apk --no-cache add ca-certificates && update-ca-certificates

RUN set -eux; \
	apt-get update; \
	apt-get install -y \
		--no-install-recommends \
		netcat; \
        chmod 755 wait-for-it.sh

# 需要运行的命令
#ENTRYPOINT ["./bubble"]