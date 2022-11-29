# cherry-pick
举例:当前项目存在分支master,develop两大分支 
master存在三次提交,将第一次提交与第三次提交合并至develop分支
![](../images/git%26cherry-pick/1669709197118-b4ff6bc9-99eb-428b-b3fa-d2c4e6807563.png)
## IDEA操作
### 将当前分支切换至develop
![](../images/git%26cherry-pick/1669709197218-fc497300-3fbc-4c7c-8a16-ca8d5fdc3d0f.png)
### git日志中选中需要合并的commit 进行cherry-pick
![](../images/git%26cherry-pick/1669709197106-c8e9248c-c069-48f3-a8ed-3c8c097676e7.png)
### 合并之后本地分支效果
![](../images/git%26cherry-pick/1669709197326-685d8a15-28f9-4517-b05c-c87d107c0018.png)
### 将本地提交推送至远程仓库
![](../images/git%26cherry-pick/1669709197184-27397d63-4e09-417b-8af6-6005157a96e2.png)
# 命令操作
### git log 查询master分支需要合并的commit ID
![](../images/git%26cherry-pick/1669709197736-3826f8c3-490d-43bf-9e85-23df6c82261d.png)

### git checkout <branch> 切换当前分支至develop
![](../images/git%26cherry-pick/1669709198390-3421d1b6-7eca-47ca-b2d4-5ae232729ad9.png)


### git cherry-pick <commit ID> 合并需要的提交
### git log 查看当前合并结果是否正确
![](../images/git%26cherry-pick/1669709198858-38dd6f65-b138-4828-b0fe-5fd9d2c428fb.png)

### git push 推送至远程


![](../images/git%26cherry-pick/1669709198811-c5be7822-68ab-4b6c-85d6-c8912fb8d77e.png)


**注意:若需要cherry-pick时出现冲突,需要先解决冲突,确认文件内容无误后再推送提交**



