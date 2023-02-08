---
title: 配置多个ssh私钥
url: https://www.yuque.com/endday/blog/mhxx9u
---

在.ssh文件夹中手动创建config文件或者输入命令touch config生成，并按下面的模板填写，该文件用于配置私钥对应的服务器。

```yaml
# gitlab
Host gitlab.ylwnl.com  　
HostName gitlab.xxx.com 　　
PreferredAuthentications publickey  
IdentityFile ~/.ssh/gitlab_id_rsa 

# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa

# gitee
Host gitee.com
HostName gitee.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitee_id_rsa
```

配置说明：
Host：自定义别名，会影响git相关命令
HostName：真实的服务器地址（域名）
User：之前配置的用户名可以省略（xxx@xxx.com）
PreferredAuthentications：权限认证（publickey,password publickey,keyboard-interactive）一般直接设为publickey
IdentityFile：rsa文件地址
