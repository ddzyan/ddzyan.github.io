---
title: Docker-文件系统
date: 2020-09-10 09:56:11
categories:
  - docker
tag:
  - docker
---

## 开卷有益

1. docker container 内的文件系统如何实现？
   1. Mount namespace
   2. chroot --- change root file system
2. 如何在 linux 进行 UnionFS 分析？
3. container 完整的层由哪几个部分组成，每个部分的作用是什么？
   1. 在 container 内进行删除文件和添加文件，layer 会进行如何处理？
4. 除了 aufs ，docker 项目还支持哪些 UnionFS 实现？

## UnionFS

### 概念理解

docker 通过 Mount Namespace 实现容器进程对文件系统“挂载点”的认识，使容器只能看到宿主机此文件的内容，看不到其他文件，实现容器与宿主机之间的“隔离”。

那么容器进行隔离的文件夹又从哪里来的呢？这里就需要引入 docker image layer 的设计机制了。

一个 docker image 是由多个 layer 构成，而每个 layer 其实就是一个 rootfs （除了第一层为 bootsf），docker 通过 UnionFS（文件联合系统），将宿主机下的多个 layer 进行联合挂载到同一个目录下（container 最终要使用），在通过 chroot（change root file system 即 改变根目录），这样在 container 中就会看到一个完全与宿主机隔离的，由多个 layer 联合的 rootfs，由因为这个 rootfs 包含了依赖，配置等等应用需要的文件，也就完成了 docker container 部署不会出现环境，依赖等问题。

目前有多种文件系统可以当作联合文件系统，实现如上功能：

- aufs
- overlay2
- zfs
- vfs
- ...

### aufs

在 docker 前期版本中使用了 aufs ，而现在使用了 overlay2 ，由于理解比较枯草，现在我们使用 aufs 文件系统在 linux 上实现多目录联合挂载：

```shell
$ mkdir A B C

$ cd A
$ echo "aaa">a && echo "xxxa">x

$ cd .. & cd B
$ echo "bbb">b && echo "xxxb">x

$ tree A
A
├── a
└── x

$ tree B
B
├── b
└── x

$ mount -t aufs -o dirs=./A:./B none ./C

# 可以看到 C 目录，具备是由 A 和 B 目录合并而成
$ tree C
C
├── a
├── b
└── x

# 对于两个相同目录下的同名文件来说，上层会覆盖下层，这个特性很重要
$ cat C/x
aaax

# 测试完毕后，可以解除挂载
$ umount work
```

<!--more-->

### overlay2

文件夹依然使用上面的 A , B, C

```shell
$ mkdir -p tmp/test ./work

# 挂载
$ mount -t  overlay overlay -o lowerdir=A:B,upperdir=C,workdir=work ./tmp/test

# 检查挂载情况
$ tree ./tmp
tree ./tmp
./tmp
└── test
    ├── a
    ├── b
    └── x

# 对联合挂载目录进行对lower层的修改和删除操作
$ cd ./tmp/test
$ rm -rf ./a && echo "test">b

# 我们查看 A,B文件夹的时候发现文件内容没有被删除也没有被改变，而在 C文件夹会创建对应上面两个文件
# 这两个文件的作用下文会说到
$ tree ./C
./C
├── a
└── b

```

### chroot

在实现文件夹联合挂载后，我们在还需要使用 chroot 来改变程序可以看到的根目录为联合挂载目录，可以使用以下指令测试，下面操作针对的是上面 aufs 的 demo

首先创建拷贝 ls 和 bash 的依赖脚本 cp.sh

```shell
T=/root/bill/docker/temp/test
list="$(ldd /bin/ls | egrep -o '/lib.*\.[0-9]')"
for i in $list; do cp -v "$i" "${T}${i}"; done

list="$(ldd /bin/bash | egrep -o '/lib.*\.[0-9]')"
for i in $list; do cp -v "$i" "${T}${i}"; done
```

```shell
$ mkdir test

$ mkdir -p ./test/{bin,lib64,lib}

$ mkdir -p ./test/lib/x86_64-linux-gnu

$ cp -v /bin/{bash,ls} ./test/bin

$ pwd
/root/bill/docker/temp/test

$ bash cp.sh
```

再改变 bash 进程的 rootfs 目录为之前创建的 temp，采取这种方式可以进行欺骗进程，而不被发现。

```shell
$ chroot /root/bill/docker/temp/test /bin/bash
bash-4.4
$ ls
bin  lib  lib64
```

综合以上，docker 实现欺骗 container 的 rootfs 的方式要如下 2 个步骤：

1. 将 image layer 进行 UnionFS 联合挂载到一个目录下
2. 在通过 chroot 将 container 的进程一号进程 bash 的 rootfs 修改为联合挂载目录

以上基本为理论基础，我们在实际通过 docker 创建一个 container 来研究下，是否和我们想的一样

## 详解 overlay2 在 docker 中的应用

```shell
$ docker info | grep "Storage Driver"
Storage Driver: overlay2
```

通过上面指令，可以看到现在 docker 使用的文件联合系统是 overlay2 文件系统，它是一个类似于 aufs 的现代的联合文件系统，并且更快

架构图

![img](https://img2018.cnblogs.com/blog/1075473/201903/1075473-20190306140759563-2087063609.png)

overlay2 包括`lowerdir`，`upperdir`和`merged`三个层次，其中：

- lowerdir：是只读的 image layer 也就是 rootfs
- upperdir：在容器创建的时候会进行创建，所有对容器的数据更改都发生在这层
- merged：是 lowerdir 和 upperdir 合并后的联合挂载点，最终暴露给用户的统一视角
- workdir：用来存放挂载后的临时文件与间接文件。

如何工作：

- 读：
  - 如果文件在容器层（upperdir），直接读取文件；
  - 如果文件不在容器层（upperdir），则从镜像层（lowerdir）读取；
- 修改（和 aufs 一致）
  - 文件不存在 upperdir ：会从 lowerdir 拷贝到 upperdir （），后续对同一文件的操作都是针对这个新创建的副本
  - 删除文件和目录：会在 upperdir 创建一个 whiteout 文件，这个文件会阻止 lowerdir 文件显示。

解析来我们下载镜像和构建容器来观察 layer 是如何进行联合挂载的，第一步先观察联合挂载点目录结构，通过下面指令，你就会发现是一个完整的操作系统目录。

```shell
$ docker run -d ubuntu:latest sleep 3600
4bd09af88fc74e366e662ffb4d23787205138f02dfe5b83948a4f235fef125a7

$ docker images | grep "ubuntu"
ubuntu                                                                    latest              4e2eef94cd6b        2 weeks ago         73.9MB

# 查看此镜像的 layer rootfs 构成
$ docker inspect --format=‘{{.RootFS.Layers}}’ 4e2eef94cd6b
‘[
sha256:2ce3c188c38d7ad46d2df5e6af7e7aed846bc3321bdd89706d5262fefd6a3390 sha256:ad44aa179b334bbf4aeb61ecef978c3c77a3bb27cb28bcb727f5566d7f085b31 sha256:35a91a75d24be7ff9c68ce618dcc933f89fef502a59becac8510dbc3bf7a4a05 sha256:a4399aeb9a0e1ddf9da712ef222fd66f707a8c7205ed2607c9c8aac0dbabe882
]’

# 查看容器的联合挂载点
$ docker inspect  --format="{{.GraphDriver.Data}}" 4bd09af88fc7
[
LowerDir:/var/lib/docker/overlay2/1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe-init/diff:/var/lib/docker/overlay2/d9b9481347eb58204f0923869bf28cea17e623f3c12ef736f89cd71e119d8fcb/diff:/var/lib/docker/overlay2/7a7901540e2ac0be290f94e842ef780c50d4176f7353158b4f491cb9ff88ce71/diff:/var/lib/docker/overlay2/6757fd72782648c40760498a35bd77d32b37ce292f359ea7475ec034aa2336f6/diff:/var/lib/docker/overlay2/3470ebd95720c14e4b3fbdccd3e091bf8fb67e07acfb64575a1605834a9c65b3/diff
MergedDir:/var/lib/docker/overlay2/1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe/merged
UpperDir:/var/lib/docker/overlay2/1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe/diff
WorkDir:/var/lib/docker/overlay2/1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe/work
]

# 进入 merged 联合挂载目录会发现是一个完整的操作系统目录
$ cd /var/lib/docker/overlay2/1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe/merged && ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# 查看 merged 是否哪些文件夹联合挂载
$ mount | grep overlay | grep "1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe"
overlay on /var/lib/docker/overlay2/1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe/merged type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/5IDNF3PDJO5Q2DZ42P4XSJTWMX:/var/lib/docker/overlay2/l/M4Y7PQAQDSK3PAT4VE2UZEF4Y7:/var/lib/docker/overlay2/l/DIVP3TVK6KVMLZY5HC6U2JCLEF:/var/lib/docker/overlay2/l/DLFCKMGZN45GKE2OLSKJBBZDMR:/var/lib/docker/overlay2/l/XZZXVJL7FUPHVONEFKZCOOOXJ7,upperdir=/var/lib/docker/overlay2/1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe/diff,workdir=/var/lib/docker/overlay2/1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe/work)
```

我们在 container 进行创建文件，验证 1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe 是否会产生

```shell
# 进入 container ，创建一个临时文件
$ docker exec -it 4bd09af88fc7 bash
$ cd temp && echo "aaa">a.text

# 退出在宿主机内进行验证,发现与预期效果一致，并且文件还会 diff 文件夹中也保存一份
$ cd /var/lib/docker/overlay2/1d1212e6e17e6432954578ec4d3a6bead1b747e9bad03802f94165f09eb4ffbe
$ cat ./tmp/a.text
aaa

# diff 目录存放的是当前层的镜像内容
$ cat ./diff/a.text

```
