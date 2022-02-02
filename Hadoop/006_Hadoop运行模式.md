![image-20211030213425362](images/image-20211030213425362.png)



```shell
# scp(secure copy)安全拷贝	from server1 to server2
  scp -r  $pdir/$fname      $user@$host:$pdir/$fname
# 命令 递归 要拷贝的文件路径/名称 目的地用户@主机:目的地路径/名称
```

```shell
# rsync远程同步工具
# rsync主要用于备份和镜像
# 具有速度快、避免复制相同内容和支持符号链接的优点
# rsync和scp区别：用rsync做文件的复制要比scp的速度快, rsync只对差异文件做更新 scp是把所有文件都复制过去
  rsync -av      $pdir/$fname        $user@$host:$pdir/$fname
# 命令   选项参数   要拷贝的文件路径/名称   目的地用户@主机:目的地路径/名称
# 选项参数说明	-a:归档拷贝		-v:显示复制过程
```

```shell
# 自定义脚本 xsync
# 脚本内容:
#!/bin/bash

#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi

#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
    echo ====================  $host  ====================
    #3. 遍历所有目录，挨个发送

    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e $file ]
            then
                #5. 获取父目录
                pdir=$(cd -P $(dirname $file); pwd)

                #6. 获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done
```



# 本地运行模式

```shell
# 创建一个输入内容的文件
vim word.txt

# word.txt内容如下
hadoop yarn
hadoop mapreduce

# hadoop目录下		输出目录必须不存在
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount word.txt的目录 输出目录

# 输出目录下的part-r-00000文件内容
hadoop	2
mapreduce	1
yarn	1
```



# 完全分布式运行模式

