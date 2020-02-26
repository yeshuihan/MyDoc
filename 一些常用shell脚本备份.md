```shell
## 查找字符存，排除out目录
function findstr(){
    find . ! -path "./out/*" -type f|xargs grep -ri --color=auto $1
}
## 在java文件中，查找字符存，排除out目录
function findjavastr(){
    find . ! -path "./out/*" -name "*.java" -type f|xargs grep -ri --color=auto $1
}
## 在mk文件中，查找字符存，排除out目录
function findmkstr(){
    find . ! -path "./out/*" -name "*.mk" -type f|xargs grep -ri --color=auto $1
}
## 在xml文件中，查找字符存，排除out目录
function findxmlstr(){
    find . ! -path "./out/*" -name "*.xml" -type f|xargs grep -ri --color=auto $1
}
## 对于repo status命令的简单过滤，去除当中没有变化的部分
function repostatus(){
str=$(repo status $1)
 
OLD_IFS="$IFS"
IFS=$'\n'
arr=("$str")
 
last=""
for line in ${arr[@]}
do
if [[ "$line" != "project"* ]]
then
if [[ "$last" = "project"* ]]
then
echo $last
fi
echo $line
fi
last=$line
done
 
IFS="$OLD_IFS"
}
## 对于repo forall -pc "git pull"命令的简单过滤，去除当中没有变化更新的部分
function repoforallgitpull(){
str=$(repo forall -pc "git pull")
 
OLD_IFS="$IFS"
IFS=$'\n'
arr=("$str")
 
procject=""
needshow="true"
for line in ${arr[@]}
do
    if [[ "$line" = "project"* ]]
    then
        procject=$line
        needshow="true"
    fi
    if [[ "$line" != "Already up-to-date."* && "$line" != "project"* ]]
    then
        if [[ "$needshow" = "true" ]]
        then
            echo $procject
            needshow="false"
        fi
        echo $line
    fi
done
 
IFS="$OLD_IFS"
}
# 检查分支的追踪分支是否正确，可能会受分支的提交内容影响
### $1 要检查的分支名
### $2 受检查分支对应的追踪分支
function checktrack(){
str=$(repo forall -pc "git branch -vv")
 
one=$1
two=$2
 
OLD_IFS="$IFS"
IFS=$'\n'
arr=("$str")
 
last=""
for line in ${arr[@]}
do
    if [[ "$line" = "project"* ]]
    then
        last=$line
    fi
     
    if [[ "$line" = *" $one "* ]]
    then
        if [[ "$line" != *"$two"* ]]
        then
            echo $last
            echo $line
        fi
    fi
done
 
IFS="$OLD_IFS"
}
```

