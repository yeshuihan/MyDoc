增加lg 别名

```shell
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

显示文件名中文

```shell
git config --global core.quotepath false
```



重置密码：

```shell
git config --system --unset credential.helper
```



设置追踪远程分支：
`切换到test`

```shell
git checkout test修改追踪仓库（一定要先切换）
git branch --set-upstream-to origin/master
```