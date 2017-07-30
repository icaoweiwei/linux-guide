# 帐号管理

## 新增与删除用户

### useradd
```
[root@www ~]# useradd [-u UID] [-g 初始群组] [-G 次要群组] [-mM] [-c 说明栏] [-d 家目录绝对路径] [-s shell] 使用者账号名
选项与参数：
-u  ：后面接的是 UID ，是一组数字。直接指定一个特定的 UID 给这个账号；
-g  ：后面接的那个组名就是我们上面提到的 initial group 啦～该群组的 GID 会被放置到 /etc/passwd 的第四个字段内。
-G  ：后面接的组名则是这个账号还可以加入的群组。这个选项与参数会修改 /etc/group 内的相关数据喔！
-M  ：强制！不要创建用户家目录！(系统账号默认值)
-m  ：强制！要创建用户家目录！(一般账号默认值)
-c  ：这个就是 /etc/passwd 的第五栏的说明内容啦～可以随便我们配置的啦～
-d  ：指定某个目录成为家目录，而不要使用默认值。务必使用绝对路径！
-r  ：创建一个系统的账号，这个账号的 UID 会有限制 (参考 /etc/login.defs)
-s  ：后面接一个 shell ，若没有指定则默认是 /bin/bash 的啦～
-e  ：后面接一个日期，格式为 YYYY-MM-DD 此项目可写入 shadow 第八字段，亦即账号失效日的配置项目啰；
-f  ：后面接 shadow 的第七字段项目，指定口令是否会失效。0为立刻失效，-1 为永远不失效(口令只会过期而强制于登陆时重新配置而已。)

范例一：完全参考默认值创建一个用户，名称为 vbird1
[root@www ~]# useradd vbird1
[root@www ~]# ll -d /home/vbird1
drwx------ 4 vbird1 vbird1 4096 Feb 25 09:38 /home/vbird1
# 默认会创建用户家目录，且权限为 700 ！这是重点！

[root@www ~]# grep vbird1 /etc/passwd /etc/shadow /etc/group
/etc/passwd:vbird1:x:504:505::/home/vbird1:/bin/bash
/etc/shadow:vbird1:!!:14300:0:99999:7:::
/etc/group:vbird1:x:505:    <==默认会创建一个与账号一模一样的群组名
```

#### useradd 默认值
```
[root@www ~]# useradd -D
GROUP=100		<==默认的群组
HOME=/home		<==默认的家目录所在目录
INACTIVE=-1		<==口令失效日，在 shadow 内的第 7 栏
EXPIRE=			<==账号失效日，在 shadow 内的第 8 栏
SHELL=/bin/bash		<==默认的 shell
SKEL=/etc/skel		<==用户家目录的内容数据参考目录
CREATE_MAIL_SPOOL=yes   <==是否主动帮使用者创建邮件信箱(mailbox)
```
这个数据其实是保存在 /etc/default/useradd 文件中的！

## passwd
```
[root@www ~]# passwd [--stdin]  <==所有人均可使用来改自己的口令
[root@www ~]# passwd [-l] [-u] [--stdin] [-S] [-n 日数] [-x 日数] [-w 日数] [-i 日期] 账号 <==root 功能
选项与参数：
--stdin ：可以透过来自前一个管道的数据，作为口令输入，对 shell script 有帮助！
-l  ：是 Lock 的意思，会将 /etc/shadow 第二栏最前面加上 ! 使口令失效。
-u  ：与 -l 相对，是 Unlock 的意思！
-S  ：列出口令相关参数，亦即 shadow 文件内的大部分信息。
-n  ：后面接天数，shadow 的第 4 字段，多久不可修改口令天数。
-x  ：后面接天数，shadow 的第 5 字段，多久内必须要更动口令。
-w  ：后面接天数，shadow 的第 6 字段，口令过期前的警告天数。
-i  ：后面接“日期”，shadow 的第 7 字段，口令失效日期。

范例一：请 root 给予 vbird2 口令
[root@www ~]# passwd vbird2
Changing password for user vbird2.
New UNIX password: <==这里直接输入新的口令，屏幕不会有任何反应
BAD PASSWORD: it is WAY too short <==口令太简单或过短的错误！
Retype new UNIX password:  <==再输入一次同样的口令
passwd: all authentication tokens updated successfully.  <==竟然还是成功修改了！

范例二：使用 standard input 创建用户的口令
[root@www ~]# echo "abc543CC" | passwd --stdin vbird2
Changing password for user vbird2.
passwd: all authentication tokens updated successfully.
```

#### PAM 模块
新的 distributions 是使用较严格的 PAM 模块来管理口令，这个管理的机制写在 /etc/pam.d/passwd 当中。而该文件与口令有关的测试模块就是使用：pam_cracklib.so，这个模块会检验口令相关的信息， 并且取代 /etc/login.defs 内的 PASS_MIN_LEN 的配置啦！

#### 口令要求
- 口令不能与账号相同；
- 口令尽量不要选用字典里面会出现的字符串；
- 口令需要超过 8 个字符；
- 口令不要使用个人信息，如身份证、手机号码、其他电话号码等；
- 口令不要使用简单的关系式，如 1+1=2， Iamvbird 等；
- 口令尽量使用大小写字符、数字、特殊字符($,\_,-等)的组合。

### chage
```
[root@www ~]# chage [-ldEImMW] 账号名
选项与参数：
-l ：列出该账号的详细口令参数；
-d ：后面接日期，修改 shadow 第三字段(最近一次更改口令的日期)，格式 YYYY-MM-DD
-E ：后面接日期，修改 shadow 第八字段(账号失效日)，格式 YYYY-MM-DD
-I ：后面接天数，修改 shadow 第七字段(口令失效日期)
-m ：后面接天数，修改 shadow 第四字段(口令最短保留天数)
-M ：后面接天数，修改 shadow 第五字段(口令多久需要进行变更)
-W ：后面接天数，修改 shadow 第六字段(口令过期前警告日期)

范例一：列出 vbird2 的详细口令参数
[root@www ~]# chage -l vbird2
Last password change                               : Feb 26, 2009
Password expires                                   : Apr 27, 2009
Password inactive                                  : May 07, 2009
Account expires                                    : never
Minimum number of days between password change     : 0
Maximum number of days between password change     : 60
Number of days of warning before password expires  : 7

范例二：创建一个名为 agetest 的账号，该账号第一次登陆后使用默认口令，
        但必须要更改过口令后，使用新口令才能够登陆系统使用 bash 环境
[root@www ~]# useradd agetest
[root@www ~]# echo "agetest" | passwd --stdin agetest
[root@www ~]# chage -d 0 agetest
# 此时此账号的口令创建时间会被改为 1970/1/1 ，所以会有问题！

范例三：尝试以 agetest 登陆的情况
You are required to change your password immediately (root enforced)
WARNING: Your password has expired.
You must change your password now and login again!
Changing password for user agetest.
Changing password for agetest
(current) UNIX password:  <==这个账号被强制要求必须要改口令！
```

### usermod
```
[root@www ~]# usermod [-cdegGlsuLU] username
选项与参数：
-c  ：后面接账号的说明，即 /etc/passwd 第五栏的说明栏，可以加入一些账号的说明。
-d  ：后面接账号的家目录，即修改 /etc/passwd 的第六栏；
-e  ：后面接日期，格式是 YYYY-MM-DD 也就是在 /etc/shadow 内的第八个字段数据啦！
-f  ：后面接天数，为 shadow 的第七字段。
-g  ：后面接初始群组，修改 /etc/passwd 的第四个字段，亦即是 GID 的字段！
-G  ：后面接次要群组，修改这个使用者能够支持的群组，修改的是 /etc/group 啰～
-a  ：与 -G 合用，可“添加次要群组的支持”而非“配置”喔！
-l  ：后面接账号名称。亦即是修改账号名称， /etc/passwd 的第一栏！
-s  ：后面接 Shell 的实际文件，例如 /bin/bash 或 /bin/csh 等等。
-u  ：后面接 UID 数字啦！即 /etc/passwd 第三栏的数据；
-L  ：暂时将用户的口令冻结，让他无法登陆。其实仅改 /etc/shadow 的口令栏。
-U  ：将 /etc/shadow 口令栏的 ! 拿掉，解冻啦！
```

```
范例一：修改使用者 vbird2 的说明栏，加上『VBird's test』的说明。
[root@www ~]# usermod -c "VBird's test" vbird2
[root@www ~]# grep vbird2 /etc/passwd
vbird2:x:700:100:VBird's test:/home/vbird2:/bin/bash

范例二：用户 vbird2 口令在 2009/12/31 失效。
[root@www ~]# usermod -e "2009-12-31" vbird2
[root@www ~]# grep vbird2 /etc/shadow
vbird2:$1$50MnwNFq$oChX.0TPanCq7ecE4HYEi.:14301:0:60:7:10:14609:

范例三：我们创建 vbird3 这个系统账号时并没有给予家目录，请创建他的家目录
[root@www ~]# ll -d ~vbird3
ls: /home/vbird3: No such file or directory  <==确认一下，确实没有家目录的存在！
[root@www ~]# cp -a /etc/skel /home/vbird3
[root@www ~]# chown -R vbird3:vbird3 /home/vbird3
[root@www ~]# chmod 700 /home/vbird3
[root@www ~]# ll -a ~vbird3
drwx------  4 vbird3 vbird3 4096 Sep  4 18:15 .  <==用户家目录权限
drwxr-xr-x 11 root   root   4096 Feb 26 11:45 ..
-rw-r--r--  1 vbird3 vbird3   33 May 25  2008 .bash_logout
-rw-r--r--  1 vbird3 vbird3  176 May 25  2008 .bash_profile
-rw-r--r--  1 vbird3 vbird3  124 May 25  2008 .bashrc
drwxr-xr-x  3 vbird3 vbird3 4096 Sep  4 18:11 .kde
drwxr-xr-x  4 vbird3 vbird3 4096 Sep  4 18:15 .mozilla
# 使用 chown -R 是为了连同家目录底下的用户/群组属性都一起变更的意思；
# 使用 chmod 没有 -R ，是因为我们仅要修改目录的权限而非内部文件的权限！
```

### userdel
删除用户的相关数据，而用户的数据有：
- 用户账号/口令相关参数：/etc/passwd, /etc/shadow
- 使用者群组相关参数：/etc/group, /etc/gshadow
- 用户个人文件数据： /home/username, /var/spool/mail/username..
```
[root@www ~]# userdel [-r] username
选项与参数：
-r  ：连同用户的家目录也一起删除

范例一：删除 vbird2 ，连同家目录一起删除
[root@www ~]# userdel -r vbird2
```
一般而言，如果该账号只是“暂时不激活”的话，那么将 /etc/shadow 里头账号失效日期 (第八字段) 配置为 0 就可以让该账号无法使用，但是所有跟该账号相关的数据都会留下来！ 使用 userdel 的时机通常是“你真的确定不要让该用户在主机上面使用任何数据了！”

如果想要完整的将某个账号完整的移除，最好可以在下达 `userdel -r username` 之前， 先以 `find / -user username` 查出整个系统内属于 username 的文件，然后再加以删除吧！

## 用户功能

### finger
finger 可以查阅很多用户相关的信息，大部分都是在 /etc/passwd 这个文件里面的信息。
```
[root@www ~]# finger [-s] username
选项与参数：
-s  ：仅列出用户的账号、全名、终端机代号与登陆时间等等；
-m  ：列出与后面接的账号相同者，而不是利用部分比对 (包括全名部分)

范例一：观察 vbird1 的用户相关账号属性
[root@www ~]# finger vbird1
Login: vbird1                           Name: (null)
Directory: /home/vbird1                 Shell: /bin/bash
Never logged in.
No mail.
No Plan.
```
- Login：为使用者账号，亦即 /etc/passwd 内的第一字段；
- Name：为全名，亦即 /etc/passwd 内的第五字段(或称为批注)；
- Directory：就是家目录了；
- Shell：就是使用的 Shell 文件所在；
- Never logged in.：figner 还会调查用户登陆主机的情况喔！
- No mail.：调查 /var/spool/mail 当中的信箱数据；
- No Plan.：调查 ~vbird1/.plan 文件，并将该文件取出来说明！
