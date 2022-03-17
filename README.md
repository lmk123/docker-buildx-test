# 尝试在 Macbook Air M1 上 build x86-64 的镜像并导入到服务器中运行

## 第一步：在 M1 上 build 镜像

这个仓库里有一个 Dockerfile，先将它 build 成一个镜像：

```bash
docker build -t arm .
```

确认镜像已经 build 出来了：

```bash
docker images
```

应该输出

```text
REPOSITORY       TAG       IMAGE ID       CREATED         SIZE
arm              latest    a1f5050666b3   6 seconds ago   5.32MB
```

运行一下这个镜像：

```bash
docker run arm
```

会输出：

```text
Linux buildkitsandbox 5.10.76-linuxkit #1 SMP PREEMPT Mon Nov 8 11:22:26 UTC 2021 aarch64 Linux
```

划重点：`aarch64`，说明这个镜像是运行在 arm64 的机器上的。

目前为止，如果用 `docker image save` 将这个镜像导出成一个 .tar 文件之后，这个 .tar 只有在同样是 arm64 的机器上导入后才能正常运行。

## build 多架构的镜像

将这个 Dockerfile 打包成 amd64 的镜像并让它出现在 `docker images` 中：

```bash
docker buildx build --platform=linux/amd64 -t amd64 -o type=image .
```

确认已经出现在了 `docker images` 中：

```text
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
amd64            latest    e2ab8ff5301e   34 seconds ago   5.58MB
arm              latest    a1f5050666b3   11 minutes ago   5.32MB
```

## 导出 build 的镜像并导入到服务器中运行

> 注意：虽然 `docker buildx build` 也有 --output 参数，但是它导出的不是一个镜像 tar，这个 tar 也许能成功导入进 docker 里，但是无法运行的。
> 至于原因，我也不清楚，暂时不深究了。

我们使用 `docker image save` 导出这个镜像：

```bash
docker image save -o amd64.tar amd64
```

然后上传到服务器中：

```bash
scp amd64.tar myserver:.
```

ssh 到服务器后，用 `ls` 确认文件已存在：

```text
amd64.tar
```

然后导入到 docker 中：

```bash
docker image load -i amd64.tar
```

用 `docker images` 确认已成功导入：

```text
REPOSITORY         TAG                  IMAGE ID       CREATED         SIZE
amd64              latest               e2ab8ff5301e   6 minutes ago   5.58MB
```

## 运行

最后，运行一下看看是不是正常的：

```bash
docker run --rm amd64
```

会输出：

```text
Linux buildkitsandbox 5.10.76-linuxkit #1 SMP PREEMPT Mon Nov 8 11:22:26 UTC 2021 x86_64 Linux
```

划重点：`x86_64`，说明这个镜像是运行在 x86_64 的机器上的。