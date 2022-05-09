# 找回root密码

### 系统：CentOS 7+

1、重启系统，在开机过程中，快速按下键盘上的方向键上和下，目的是告知引导程序，我们需要在引导页面选择不同的操作，以便让引导程序暂停。

2、按键盘 e 键，进入编辑模式，找到 linux16 的那一行。将光标一直移动到 LANG=en_US.UTF-8 后面，空格，再追加 init=/bin/sh。

3、按下Ctrl+X 进行引导启动(单用户模式启动)，成功后进入该界面。然后输入以下命令

```bash
#挂载根目录
mount -o remount,rw /
#选择要修改密码的用户名，这里选择root用户进行修改，可以更换为你要修改的用户
passwd root
#输入2次一样的新密码
#新密码
#重复输入新密码
```

4、更新系统信息

```bash
touch /.authorelabel
```

5、最后输入以下命令重启系统即可

```bash
exec /sbin/init
#或
exec /sbin/reboot
```

