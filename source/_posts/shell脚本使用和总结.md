---
title: shell脚本使用和总结
date: 2018-02-04 22:23:38
tags:
author: 贾艳成
---
# shell脚本使用和总结
## hello world
- 新建`test1.sh`
- 内容`echo Hello world!`

> 如果需要的话，赋予可执行权限`chmod +x test1.sh`


## 执行指令
可以在脚本中写入更多的命令，比如：启动app，编译、运行java文件，链接服务器等等；只要你能想到的，基本都可以。在此，我仅以一个简单的例子来说明：通过脚本编译运行一个java文件。

```sh
cd /Users/daniel/Documents/tmp
javac Test2.java
java Test2
```

## 匹配规则${}，##, %% , ：- ，：+， ？ 的使用（路径处理）
假设我们定义了一个变量为：`file=/dir1/dir2/dir3/my.file.txt`

- 1.可以用${ }分别替换得到不同的值：

```
${file#*/}：删掉第一个/ 及其左边的字符串：dir1/dir2/dir3/my.file.txt
${file##*/}：删掉最后一个/  及其左边的字符串：my.file.txt
${file#*.}：删掉第一个.  及其左边的字符串：file.txt
${file##*.}：删掉最后一个.  及其左边的字符串：txt
${file%/*}：删掉最后一个 /  及其右边的字符串：/dir1/dir2/dir3
${file%%/*}：删掉第一个/  及其右边的字符串：(空值)
${file%.*}：删掉最后一个 .  及其右边的字符串：/dir1/dir2/dir3/my.file
${file%%.*}：删掉第一个 .   及其右边的字符串：/dir1/dir2/dir3/my
```

- 2.记忆的方法为：

```
# 是 去掉左边（键盘上#在 $ 的左边）
%是去掉右边（键盘上% 在$ 的右边）
单一符号是最小匹配；两个符号是最大匹配
${file:0:5}：提取最左边的5 个字节：/dir1
${file:5:5}：提取第5 个字节右边的连续5个字节：/dir2
也可以对变量值里的字符串作替换：
${file/dir/path}：将第一个dir 替换为path：/path1/dir2/dir3/my.file.txt
${file//dir/path}：将全部dir 替换为path：/path1/path2/path3/my.file.txt
```

- 3.利用${ } 还可针对不同的变数状态赋值(沒设定、空值、非空值)： 

```
${file-my.file.txt} ：假如$file 沒有设定，則使用my.file.txt 作传回值。(空值及非空值時不作处理) 
${file:-my.file.txt} ：假如$file 沒有設定或為空值，則使用my.file.txt 作傳回值。(非空值時不作处理)
${file+my.file.txt} ：假如$file 設為空值或非空值，均使用my.file.txt 作傳回值。(沒設定時不作处理)
${file:+my.file.txt} ：若$file 為非空值，則使用my.file.txt 作傳回值。(沒設定及空值時不作处理)
${file=my.file.txt} ：若$file 沒設定，則使用my.file.txt 作傳回值，同時將$file 賦值為my.file.txt 。(空值及非空值時不作处理)
${file:=my.file.txt} ：若$file 沒設定或為空值，則使用my.file.txt 作傳回值，同時將$file 賦值為my.file.txt 。(非空值時不作处理)
${file?my.file.txt} ：若$file 沒設定，則將my.file.txt 輸出至STDERR。(空值及非空值時不作处理)

${file:?my.file.txt} ：若$file 没设定或为空值，则将my.file.txt 输出至STDERR。(非空值時不作处理)

${#var} 可计算出变量值的长度：

${#file} 可得到27 ，因为/dir1/dir2/dir3/my.file.txt 是27个字节
```

## 分割字符串
比如，要分割`test="aaa,bbb,cc cc,dd dd"`，可以这样:

```sh
# 1.分割
OLD_IFS="$IFS" 
IFS="-" 
arr=($filename) 
IFS="$OLD_IFS" 

# 2.获取
for x in $arr; do
  echo $x
done
echo ${arr[0]}
```

## 实例
取出照片，按照日期创建文件夹，并将文件放到指定文件夹中。

```sh
for filePath in /Users/***/DCIM/*.jpg; do
	# 1.获取文件名，eg: P70827-165946.jpg
	filename=${filePath##*/} 
	echo ${filename}

	# 2.获取日期标识部分P70827：从头开始取6位
	dirName="/Users/***/DCIM/"${filename:0:6}
	echo $dirName"\n"

	# 3.根据日期创建文件夹
	mkdir -p ${dirName} 

	# 4.将文件移入该文件夹
	mv  ${filePath}  ${dirName}
done
```

## 注意事项
- 等号`=`两边不能有空格:空格对于linux的shell是一种很典型的分隔符，所以给变量赋值的时候中间不能够有空格。


# 参考文档
- [ 求助linux下批量建立文件夹和移动文件](http://bbs.chinaunix.net/thread-4082007-1-1.html)
- [Shell 教程](http://www.runoob.com/linux/linux-shell.html)
- [shell编程--遍历目录下的文件](https://www.cnblogs.com/kaituorensheng/archive/2012/12/19/2825376.html)
- [shell中的${}，##和%%的使用](https://www.douban.com/note/498011007/?type=rec)
- [linux shell 字符串操作详解 （长度，读取，替换，截取，连接，对比，删除，位置 ）](https://www.cnblogs.com/gaochsh/p/6901809.html)









