

1、根据日期生成文件夹的简单shell脚本
#!bin/sh
cd /share_eip/gtp/files/eip/snd/gt
num=0
while((num<76))
do
dateStr=`perl -e "print sprintf '%04d%02d%02d',(localtime(time()+86400*$num))[5]+1900,(localtime(time()+86400*$num))[4]+1,(localtime(time()+86400*$num))[3]"`
mkdir $dateStr
chmod -R 777 $dateStr
num=`expr $num + 1`
done

2、echo和printf的区别
echo会自动换行，printf不会，printf换行符 \n  printf：printf format [arguments]

3、vi编辑器
输入
	A：在行尾输入
	I：在当前行前输入
	o：在当前行后输入新一行
	O：在当前行前输入新一行

移动光标
	j：向下移动一行
	k：向上移动一行
	b：移动到当前单词的开始  
	e：移动到当前单词的结尾 
	w：移动到下个单词的首字母
	$：到行尾
	0：到行首
	G：文档尾部  nG：文档第几行
	L：屏幕尾部  H：屏幕首部

删除
	x：字符删除
	d$或D：从当前位置删除到行尾
	d0：从当前位置删除到行首
	dd：删除当前行
	ndd：从当前行向下删除n行
	dG：删除至文档尾部
	dnG：删除至文档的n行
	dw：删除光标所在单词

复制粘贴
	yy：复制当前行
	y$或Y：复制当前光标到行尾
	nyy：复制n行
	p：将剪贴板的内容粘贴到光标后一行
	P：将剪贴板的内容粘贴到光标前一行
	用光标复制：按v进入可视模式，按方向键，选中你想复制的内容，按y复制，最后移动到想要粘贴的地方按p粘贴	删除同理

撤销
	u：撤销

替换
	r：替换光标所在字符
	:s/vivian/sky/ 替换当前行第一个vivian为sky
	:s/vivian/sky/g 替换当前行所有vivian为sky
	:n,$s/vivian/sky/替换第n行开始到最后一行中每一行的第一个vivian为sky
	:n,$s/vivian/sky/g替换第n行开始到最后一行中每一行所有vivian为sky
	:%s/vivian/sky/ 替换每一行的第一个vivian为sky  等同于:g/vivian/s//sky/
	:%s/vivian/sky/g 替换每一行中所有vivian为sky   等同于:g/vivian/s//sky/g

先进入命令模式(Esc) 
:/str 查找str，按n下一处，按N上一处 
:1 回首行 
:$ 到尾行

4、export命令
用于设置环境变量，可新增、修改或删除环境变量。export的效力仅及于该次登录操作，注销或重开一个窗口，export设置的环境变量都不存在了。
export [-fnp] [变量名称=值]
-f：代表变量名称为函数名称
-n：删除指定的变量。
-p：列出所有的shell赋予程序的环境变量
eg：
	设置环境变量  export TEST_PATH=$HOME/app/logs

5、&符号
当在前台运行某个作业时，终端被该作业占据，可以在命令后加上“&”实现后台运行。例如 sh test.sh &

6、>、<、>>的区别
> ：直接把内容生成到指定文件，会覆盖原文件中的内容；还有一种用途是直接生成一个空白文件，相当于touch命令 ---- ‘> aaa.txt’
>>：尾部追加，不会覆盖原文件

7、nohup命令
使用&命令后，作业被提交到后台运行，当前控制台没有被占用，但是一旦把当前控制台关掉（退出账户），作业就会停止运行。nohup命令可以在你退出账户之后继续运行相应的进程。nohup就是不挂起的意思，no hang up。例如 nohup command &。使用nohup命令提交作业，默认情况下该作业的所有输出都被重定向到一个名为nohup.out的文件中，除非另外指定了输出文件：“nohup command > myout.file 2>&1 &”
2>&1解析：
	command > out.file是将command的输出重定向到out.flie文件，即输出内容不显示到屏幕上。
	2>&1 是将标准出错重定向到标准输出，这里的标准输出已经重定向到了out.file文件中，即将标准出错也输入到out.file文件中
	0,1,2分别代表stdin标准输入，stdout标准输出，stderr标准错误，2与>结合代表错误重定向
command > /dev/null等价于command 1 > /dev/null，执行command产生的标准输出重定向到/dev/null中
command 1>a 2>&1与command 1>a 2>a的区别
	区别在于前者只打开一次文件a，后者会打开文件a两次，并导致stdout标准输出被stderr标准错误覆盖
	&1的含义可以理解为用标准输出的引用，引用的就是重定向标准输出产生打开的a，从效率上来讲，&的效率要更高
	举个例子，有一个test.sh
	  #!/bin/sh
	  t
	  date
	执行./test.sh >res.log ，发现错误输出打印到了屏幕上，而date的结果打印到了res.log文件中，这是由于我们没有指定错误输出
	执行./test.sh >res.log 2>&1，发现错误和标准输出都打印到了res.log中

8、
