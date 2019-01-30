Git

 (1) SSH问题

```
$ git push -u origin master
ERROR: Permission to KenetGit/start_your_ssm.git denied to fangjm56.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

> 出现这个问题是因为计算机本地的SSH配置文件(RSA加密)是属于fangjm56的，而不是KenetGit，所以需要先清除本地的fangjm56的SSH文件，所在目录：/c/Users/jiamoufang/.ssh/

1. git 下输入，默认完成操作即可。

```
 ssh-keygen -t rsa -C "fangjiamou@gmail.com"
```

2. Github账户设置settings中设置SSH即可。

(2) 本地分支与远程未建立连接

```
fatal: 'origin' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

1. 查看分支

> $ git branch
> -- master

2. 查看与上游的联系(为空的话说明联系已经断开，fetch不了也push不了)

> $ git remote -v

3. 添加联系

> $ git remote add origin ssh:git@github.com:KenetGit/gitNotes.git
>
> （后面如果还是提交失败的话）git remote set-url origin git@github.com:KenetGit/gitNotes.git

4. 
   