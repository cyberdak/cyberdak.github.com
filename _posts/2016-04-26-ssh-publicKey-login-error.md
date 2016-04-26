---
layout : post
title : SELinux 引起的ssh公钥认证失败问题
category : linux
---


最近打算强化linux的安全配置。
具体就是

+ 更换ssh登陆端口
+ 禁用root账号远程登陆
+ 安装fail2ban来防止暴力破解
+ 使用公钥登陆，禁用密码登陆

首先说一下，更换端口部分很简单，把sshd_config 的Port 换成一个五位数端口就行了。
之前在论坛上交流，大家基本都是改到五位数以上。本来以为可以杜绝爆破了，但是后来有发现一个帖子讨论的时候，有人依然对着五位数的端口爆破了两个月。
所以萌生了进一步加强安全设置的想法。

初步想法是使用fail2ban或者公钥来认证。

想了下，迟早是要用公钥的，直接一步到位好了。因为用了公钥的话，可以完全禁止密码登陆，自然也就不存在fail2ban的使用场景了（这里只针对ssh的爆破，设想的方案是登陆失败3次就使用iptables来禁用ip，但是黑客大牛手里肯定一堆代理。）。

先在虚拟机中用CentOS6.7 测试一下

建立~/.ssh目录,并赋予700权限

```
mkdir .ssh
chmod 700 .ssh
```
进入.ssh目录，生成密钥。

`
ssh-keygen
`

一路回车就会生成 `id_rsa` 和 `id_rsa.pub`
将`id_rsa.pub`文件改名为`authorized_keys`，并修改权限为600,`authorized_keys`是sshd_config 配置中的默认文件名

```
mv id_rsa.pub authorized_keys
chmod 600 authorized_keys
```

重启sshd

```
service sshd restart
```

但是登陆一下结果是这样的：

`Public-key authentication with the server for user sw failed. Please verify username and public/private key pair.`

各种办法都试了一圈，还是找不到。

之后使用另外一台服务器的`ssh -vvv` 来连接，为了获取足够多的信息

信息大概是这样子的

```
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug2: key: /home/sw/.ssh/identity (0xXXXXXXXXX)
debug2: key: /home/sw/.ssh/id_rsa ((nil))
debug2: key: /home/sw/.ssh/id_dsa ((nil))
debug3: Wrote 64 bytes for a total of 1109
debug1: Authentications that can continue: publickey,password
debug3: start over, passed a different list publickey,password
debug3: preferred publickey,keyboard-interactive,password
debug3: authmethod_lookup publickey
debug3: remaining preferred: keyboard-interactive,password
debug3: authmethod_is_enabled publickey
debug1: Next authentication method: publickey
debug1: Offering public key: /home/sw/.ssh/identity
debug3: send_pubkey_test
debug2: we sent a publickey packet, wait for reply
debug3: Wrote 512 bytes for a total of 1621
debug1: Authentications that can continue: publickey,password
debug1: Trying private key: /home/sw/.ssh/id_rsa
debug3: no such identity: /home/sw/.ssh/id_rsa
debug1: Trying private key: /home/sw/.ssh/id_dsa
debug3: no such identity: /home/sw/.ssh/id_dsa
debug2: we did not send a packet, disable method
debug3: authmethod_lookup password
debug3: remaining preferred: ,password
debug3: authmethod_is_enabled password
debug1: Next authentication method: password
```

还是没有有用信息

这时怀疑到SELinux上面，比较这个东西完全不讲道理。各种看起来合理的操作都会被SELinux拦截一番（这里开始理解为什么阿里云的centos镜像默认关闭了SELinux）。
通过google搜索了一下，果然是这个SELinux的锅。

`cat /var/log/audit/audit.log | grep denied`

通过本条命令查看失败日志

```
type=AVC msg=audit(1461618507.140:776): avc:  denied  { open } for  pid=3065 comm="sshd" name="authorized_keys" dev=sda1 ino=1056263 scontext=unconfined_u:system_r:sshd_t:s0-s0:c0.c1023 tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file
```

缺少`open`权限

在SELinux模式下，.ssh 需要为`.ssh_home_t` 不然就没有相应的权限。

这里用`ls -alZ` 查看一下目录。得到结果如下.

```
drwx------. root root unconfined_u:object_r:admin_home_t:s0 .ssh
```

果然啊

在另外一个服务器上，就是后来用来测试登陆测试一下，那台服务器的.ssh目录是由sshgen程序直接建立的，而这里的是由手工`mkdir`建立的，就导致了权限错误问题。

然后就是修复目录权限了

```
restorecon -r -vv /root/.ssh
```

之后就可以成功登陆了.