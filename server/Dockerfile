# 使用 Alpine Linux 作为基础镜像，保持镜像体积小巧
FROM alpine:latest

# 定义 FRP 版本和架构
ARG FRP_VERSION=0.58.0
ARG ARCH=amd64

# 设置工作目录
WORKDIR /frp

# 下载并解压 FRP
RUN wget "https://github.com/fatedier/frp/releases/download/v${FRP_VERSION}/frp_${FRP_VERSION}_linux_${ARCH}.tar.gz" -O frp.tar.gz && \
    tar -zxvf frp.tar.gz --strip-components=1 -C /frp frp_${FRP_VERSION}_linux_${ARCH}/frps && \
    rm frp.tar.gz

# 复制自定义配置文件到容器中
COPY frps.toml /frp/frps.toml

# 暴露服务端端口和 Dashboard 端口 (虽然使用 host 网络模式，但声明端口有助于理解)
EXPOSE 7000 7500

# 设置容器启动时执行的命令
ENTRYPOINT ["/frp/frps", "-c", "/frp/frps.toml"]
