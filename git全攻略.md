## git全攻略

```
# 配置全局git用户
	git config --global user.name “用户名”
	git config --global user.email “邮箱”
# 生成ssh公钥
	ssh-keygen -t rsa -C “邮箱” 一路回车
	复制.ssh/id_rsa.pub内容到git用户的setting中
------------至此，git可联通----------------------------
# 初始化本地仓库
	cd 进入本地工程目录 
	git init
# 添加项目内容至本地仓库
	git add .
# 添加内容进本地stage
	git commit -a -m "comment内容"
# 设置该仓库远端地址
	git remote add origin "远端地址"
# push本次提交（初次）
	git push -u origin master
---------至此，第一次提交完成---------------------------
# 显示所有分支（远端和本地）
	git branch -a
# 新建并切换分支
	git checkout -b "分支名"
# 提交到对应分支
	git push origin "分支名"
#设置默认对应远端
	git branch --set-upstream-to=origin/"分支名"
------------至此，可以工作在新分支上-------------------
# 查看提交记录
	git log
# 依次回退提交记录
	依次 git reset "提交的hash码"
----------回档提交记录方法----------------------------
```

------

## 