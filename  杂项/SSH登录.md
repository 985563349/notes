### SSH登录

操作系统：Ubuntu Server 20.04 LTS 64bit

#### 建立连接
1. 使用ubuntu帐户登录服务器。

2. 执行以下命令，设置root帐户密码。

```shell
sudo passwd root
```

3. 输入root帐户密码，按**Enter**，重复输入root帐户密码，按**Enter**。返回如下信息，即密码设置成功。

```shell
passwd: password updated successfully
```

4. 执行以下命令，打开`ssh_config`配置文件。

```shell
sudo vi /etc/ssh/sshd_config
```

5. 按 `i` 进入编辑模式，找到`#Authentication`，将`PermitRootLogin`参数修改为`yes`(允许root帐户登录)。若参数被注释，则去掉首行的注释符号（`#`）。

```shell
# Authentication:
#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

6.  找到`#Authentication`，将`PasswordAuthentication`参数修改为`yes`（允许远程密码登录）。若配置文件中无此配置项，则添加`PasswordAuthentication yes`项即可。

```shell
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
PasswordAuthentication yes
```

7. 按`ESC`，输入`:wq`，保存文件并返回。

8. 执行以下命令，重启ssh服务。

```shell
sudo service ssh restart
```

9. 打开Mac终端，执行以下命令新建ssh远程连接。

```shell
# ssh <username>@<IP address or domain name>。

ssh root@101.43.4.238
```

10. 输入已获取的密码，按**Enter**，即可完成远程连接。

------

#### FAQ

SSH登录服务器`REMOTE HOST IDENTIFICATION HAS CHANGED`错误解决办法：

输入以下命令更新本地ssh：

```shell
ssh-keygen -R <IP address>
```
