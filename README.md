
# 1、概述


　　Docker 作为一款广受欢迎的容器化技术，为开发者提供了极大的便利。它能够将应用程序以及其全部的依赖项整合并打包，形成一个标准化的独立单元 —— 镜像。对 Docker 镜像进行优化意义非凡，一方面可以显著降低镜像的存储空间占用，进而大幅提升其下载与部署的速率；另一方面，还能有效强化系统的安全性，为应用的稳定运行保驾护航。


　　在本文中，我们将探讨一系列切实可行的 Docker 镜像优化技巧，助力开发者轻松掌握这一关键技能，为构建高效、可靠的容器化应用奠定坚实基础，让您的开发与部署流程更加顺畅无阻，全面提升应用的性能表现与安全保障。


# 2、Docker镜像优化技巧


## 2\.1 使用合适的基础镜像


选择合适的基础镜像是优化 Docker 镜像的重要一步。尽量选择体积小、功能简单的基础镜像。


* 比如 alpine，它是一个轻量级的 Linux 发行版，适合用作 Docker 镜像的基础。
* 比如 distroless，安全性高，详细介绍参见《[distroless 镜像介绍及调试基于distroless 镜像的容器](https://github.com/zhangmingcheng/p/15795201.html "发布于 2022-01-12 20:34")》这篇博文。


示例，以下是一个使用 alpine 作为基础镜像的 Dockerfile 示例：



[?](https://github.com)

| 1234567891011121314151617 | `# 使用 alpine 作为基础镜像``FROM alpine:3.15` `# 安装必要的依赖``RUN apk add --no-cache python3 py3-pip` `# 设置工作目录``WORKDIR /app` `# 复制应用文件``COPY . .` `# 安装 Python 依赖``RUN pip install -r requirements.txt` `# 启动应用``CMD ["python3", "app.py"]` |
| --- | --- |



## 2\.2 合并 RUN 指令


每个 RUN 指令都会创建一个新的层，增加镜像的大小。通过合并多个 RUN 指令，可以减少镜像的层数，进而减少镜像的体积。


示例：



[?](https://github.com)

| 123 | `# 不推荐的做法：多层构建``RUN apk add --no-cache git``RUN apk add --no-cache curl` |
| --- | --- |



\# 推荐的做法：合并 RUN 指令



[?](https://github.com)

| 1 | `RUN apk add --no-cache git curl` |
| --- | --- |



## 2\.3 清理不必要的文件


在构建镜像的过程中，可能会生成一些不必要的文件或缓存。可以在 Dockerfile 中使用清理命令来减小镜像体积。


示例：



[?](https://github.com)

| 12 | `# 安装依赖并清理缓存``RUN apk add --no-cache git curl && rm -rf /var/cache/apk/*` |
| --- | --- |



## 2\.4 使用 .dockerignore 文件


与 .gitignore 文件类似，.dockerignore 文件可以用来排除不必要的文件和目录，避免将它们包含在镜像中。这有助于减小镜像体积，并加快构建速度。


示例，在项目根目录下创建一个 .dockerignore 文件，内容如下：



[?](https://github.com):[豆荚加速器](https://yirou.org)

| 123 | `# 排除不必要的文件和目录``*.log``node_modules/` |
| --- | --- |



## 2\.5 使用多阶段构建


多阶段构建允许你在一个 Dockerfile 中使用多个 FROM 指令，从而只将所需的文件复制到最终镜像中。这种方法可以显著减小镜像体积。详细介绍参见《[Docker中Dockerfile多From 指令存在的意义](https://github.com/zhangmingcheng/p/13991412.html)》这篇博文。


示例：



[?](https://github.com)

| 12345678910111213141516171819202122232425262728293031 | `# 第一阶段：构建应用``FROM golang:1.22 AS builder` `#RUN #go env -w GOPROXY=https://goproxy.cn,direct``# Run this with docker build --build_arg $(go env GOPROXY) to override the goproxy``#ARG goproxy=https://goproxy.cn,direct``#ENV GOPROXY=$goproxy` `WORKDIR /workspace` `COPY ../../apis apis/``COPY ../../clients clients/``COPY ../../cmd cmd/``COPY ../../pkg/controllers controllers/``COPY ../../pkg pkg/``COPY ../../pkg/constant constant/``COPY ../../swaggerDocs swaggerDocs/``COPY ../../go.mod go.mod``COPY ../../go.sum go.sum``RUN go mod tidy && go mod vendor<``br``>``# Do not force rebuild of up-to-date packages (do not use -a) and use the compiler cache folder``RUN CGO_ENABLED=0 GOOS=linux GOARCH=${TARGETPLATFORM} go build   -o /workspace/ke ./cmd/ke` `# 第二阶段：创建最终镜像``FROM alpine:3.19` `WORKDIR /` `COPY --from=builder /workspace/ke /bin/ke` `ENTRYPOINT ["/bin/ke"]` |
| --- | --- |



## 2\.6 使用标签和版本控制


给镜像打上标签，可以方便地进行版本控制和管理，避免混淆不同版本的镜像。


示例：



[?](https://github.com)

| 12 | `# 构建镜像并打标签``docker build -t helloworld:1.0 .` |
| --- | --- |



# 3、总结



　　在容器化应用的浪潮中，优化 Docker 镜像对于提升应用性能和安全性意义非凡。选择合适基础镜像是关键起点，如同搭建稳固根基，能有效控制镜像的规模与功能范畴。巧妙合并相关指令，可避免镜像层不必要的增多，切实缩减体积。构建过程中，及时清理冗余文件，剔除无实际作用的元素，为镜像 “瘦身”。合理运用.dockerignore 文件，精准排除无关文件和目录，加速构建并减少无用负载。采用多阶段构建策略，分步筛选，仅保留最终运行的必要组件，降低复杂性并提升效率。


　　遵循 “**容器镜像应仅含应用程序及其运行时依赖项**” 的原则，运用这些优化技巧，让 Docker 镜像更加精简高效，为容器化应用的平稳、快速部署保驾护航，助力开发者充分发挥 Docker 技术优势，推动应用开发与部署向高质量发展迈进。



参考：《[distroless 镜像介绍及调试基于distroless 镜像的容器](https://github.com/zhangmingcheng/p/15795201.html "发布于 2022-01-12 20:34")》


参考：《[Docker中Dockerfile多From 指令存在的意义](https://github.com/zhangmingcheng/p/13991412.html)》


