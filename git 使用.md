### git 使用
git init  : 在当前目录创建git仓库		

git status : 查看当前目录文件状态			

git add . : 添加当前目录(工作空间)所有文件到暂存
	
git commit -m '注释'  : 提交暂存区文件到本地仓库	
git remote : 查看当前目录的git仓库关联的远程仓库地址
	
git remote --help : 远程关联帮助	

git remote add origin 远程仓库地址 :  关联远程仓库

git push origin master : 上传本地仓库到远程

git pull 

git tag : 查看标签	

git tag -a '标签' -m '注释'  ：打标签 eg: '0.1.0'

git tag -d '标签名称' : 删除本地标签eg: git tag -d '0.0.1'
	
git push --tags : 上传本地所有标签到远程

git push origin :标签 : 删除远程的标签eg: git push origin :0.0.1

git push origin 0.0.1 ： 提交0.0.1标签对应的版本到origin (区别一个冒号)

git log : 查看日志or

git 的 release 版本就是对应的标签，打一个标签就有一个对应的release版本, 本地打标签然后推送到远程, 




