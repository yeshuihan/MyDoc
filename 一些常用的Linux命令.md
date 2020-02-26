git lg | grep -ri -C1 “logo”   //在git log中查找相关的内容，显示前后一行
查找目录下的所有文件中是否含有某个字符串

find .|xargs grep -ri –color=auto “IBM”

查找目录下的所有文件中是否含有某个字符串,并且只打印出文件名

find .|xargs grep -ri –color=auto “IBM” -l

 

find ./ ! -path “./out/*” -type f|xargs grep -ri –color=auto “IBM” ? //在文件中查找字符串,忽略out/目录

repo forall [project] -c “git log” 查看么某个project的loggit log -name –stat 查看log，以及修改的文件

 