---
title: Linux常用指令
date: 2018-09-14 01:04:57
categories:
  - linux
tags:
  - linux
---

```sh
# 树状图输出文件夹结构，需要安装 yum install -y tree
tree -L 2

# 输出内容到控制台
echo "hello word"

# 在 a.txt 结尾添加 abcd
echo "abcd" >> a.txt

# 替换文本内容，如果不存在则创建文件
echo "hello" > a.txt
```

<!--more-->

```sh
# 查看隐藏文件
ls -a

# 文件按时间排序：l：显示完整信息，t：按照时间排序，r：倒序
ls -lrt

# 查看所有.js结尾的文件
ls *.js

# 在当前目录下查找名字为 java 的文件夹
find ./ -name "stellar-core.cfg" | xargs file

# 在档期那目录下查找名字为 index 开头的文件或者文件夹
find ./ -iname "index*"

# 分页查看内容
more index.js

# 带行号的分页查看
cat -n index.js | more

# 查看2个文件的不同
diff index.js  index2.js

# 查找index.js 是否包含2  并且输出行号
cat -n index.js | egrep -0 "2"  |  egrep -n "2"  index.js

# 创建硬链接
ln cc abc

# 创建软链接
ln -s cc abc

# 找出不包含2的内容,v：不包含，i：忽略大小写，n：显示行号，
# -A:显示匹配到的字符串所在行的后几行，-B：显示匹配到的字符串所在行的前几行，-C：显示匹配到的字符串所在行的前后几行
egrep -v 2 index.js

# 包含两个关键字
egrep -in "aaa|bbb" abc.txt

# 替换文本内容
sed -i 's#1#2#g' index2.js

# 列出当前文件夹下所有文件大小
du -sh `ls`

# 打包文件，没有压缩   c打包  v 显示进度   f 使用文件
tar -cvf index.tar index.js

# 压缩文件,最后文件名称问index.tar.gz
gzip index.tar

# 解压
gunzip index.tar.gz

# 解开打包 x 解包
tar -xvf index.tar

# 查看所有 docker 进程
pgrep -l docker

# 查看端口占用状态
lsfo -i:3306

#  查看指定进程打开的文件
lsof -p 3326

# 查看所有TCP网络连接信息
lsof -i tcp

# 显示资源使用情况  M 按照占用内存排序显示 P按照CPU使用情况排序显示  i 不现实闲置进程
top

# 查看指定进程资源使用情况
top -p [pid]

# 显示完整进程信息，包括启动命令
top -c

# 查看内存使用情况
free -m

# 查看cpu使用率 idle空闲的CPU  user  CPU使用率
sar -u 2 2

#  查看所有端口
netstate -a

#  查看所有TCP接口
netstate -at

# 给所有用户授权执行index.js r读，w写
chmod a+x index.js

#
chown username dirOrFile

chown -R weber server/

# 修改环境变量
vim ~/.bash_profile
source ~/.bash_profile

# 查看test依赖的所有环境
ldd test

# 显示所有进程信息，包括命令行
ps -ef

# 显示所有在内存中的程序
ps aux

# 显示所有信息
crontab -l

# 编辑添加
crontab -e

# 重启crontab
/etc/init.d/crond restart
```
