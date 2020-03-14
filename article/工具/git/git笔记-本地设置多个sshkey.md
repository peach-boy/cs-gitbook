### 1.生成两个sshkey
```
$ ssh-keygen -t rsa -C "1email@company.com” -f ~/.ssh/id_rsa

$ ssh-keygen -t rsa -C "2email@github.com” -f ~/.ssh/github_rsa
```
此时在本地.ssh目录里会出现以下文件，id_rsa和id_rsa.pub，github_rsa和github_rsa.pub，即生成了秘钥对

### 2.在gitlab和github中添加对应公钥（将.pub文件中的文本复制到对应的平台）

### 3.新建config文件（映射文件）

```
#gitlab
Host gitlab.wuxingdev.cn
    Hostname gitlab.wuxingdev.cn
    User git_username 
    PreferredAuthentications publickey
    IdentityFile C:\Users\wuxiantao\.ssh\\id_rsa
#github
Host github.com
    Hostname github.com
    User github_username  没有实际作用
    PreferredAuthentications publickey
    IdentityFile C:\Users\wuxiantao\.ssh\\github_rsa
```

### 注意，如果使用了 git config --global user.name/email设置了全局用户名和邮箱，每次提交，都会使用此设置。也可以为每个git项目单独设置name和email


