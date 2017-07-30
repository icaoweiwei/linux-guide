# Bash Shell 的操作环境

## 命令运行的顺序
1. 以相对/绝对路径运行命令，例如 `/bin/ls` 或 `./ls`；
1. 由 alias 找到该命令来运行；
1. 由 bash 内建的(builtin) 命令来运行；
1. 透过 `$PATH` 这个变量的顺序搜寻到的第一个命令来运行。

想要了解命令搜寻的顺序，其实透过 `type -a ls` 也可以查询的到啦！

## bash 的进站与欢迎信息：`/etc/issue`, `/etc/motd`
### /etc/issue 内的各代码意义
```
\d 本地端时间的日期；
\l 显示第几个终端机接口；
\m 显示硬件的等级 (i386/i486/i586/i686...)；
\n 显示主机的网络名称；
\o 显示 domain name；
\r 操作系统的版本 (相当于 uname -r)
\t 显示本地端时间的时间；
\s 操作系统的名称；
\v 操作系统的版本。
```
当我们使用 `telnet` 连接到主机时，主机的登陆画面就会显示 `/etc/issue.net` 而不是 `/etc/issue` 。

如果您想要让使用者登陆后取得一些信息，例如您想要让大家都知道的信息， 那么可以将信息加入 `/etc/motd` 里面去！

## bash 的环境配置文件

### login shell 与 non-login shell
#### login shell
取得 bash 时需要完整的登陆流程的，就称为 login shell。举例来说，你要由 tty1 ~ tty6 登陆，需要输入用户的账号与密码，此时取得的 bash 就称为 login shell 。
#### non-login shell
取得 bash 接口的方法不需要重复登陆的举动，举例来说：
1. 你以 X window 登陆 Linux 后， 再以 X 的图形化接口启动终端机，此时那个终端接口并没有需要再次的输入账号与密码，那个 bash 的环境就称为 non-login shell了。
2. 你在原本的 bash 环境下再次下达 bash 这个命令，同样的也没有输入账号密码， 那第二个 bash (子程序) 也是 non-login shell 。

这两个取得 bash 的情况中，读取的配置文件数据并不一样。

### login shell 只会读取这两个配置文件
1. /etc/profile：这是系统整体的配置，你最好不要修改这个文件；
1. ~/.bash_profile 或 ~/.bash_login 或 ~/.profile：属于使用者个人配置，你要改自己的数据，就写入这里！

#### /etc/profile
这个文件配置的变量主要有：
- PATH：会依据 UID 决定 PATH 变量要不要含有 sbin 的系统命令目录；
- MAIL：依据账号配置好使用者的 mailbox 到 /var/spool/mail/账号名；
- USER：根据用户的账号配置此变量内容；
- HOSTNAME：依据主机的 hostname 命令决定此变量内容；
- HISTSIZE：历史命令记录笔数。CentOS 5.x 配置为 1000 ；

加载外部的配置数据：
- /etc/inputrc：其实这个文件并没有被运行啦！/etc/profile 会主动的判断使用者有没有自定义输入的按键功能，如果没有的话， /etc/profile 就会决定配置 `INPUTRC=/etc/inputrc` 这个变量！此文件内容为 bash 的热键啦、[tab]要不要有声音啦等等的数据！
- /etc/profile.d/\*.sh：其实这是个目录内的众多文件！只要在 /etc/profile.d/ 这个目录内且扩展名为 .sh ，另外，使用者能够具有 r 的权限， 那么该文件就会被 /etc/profile 加载进来。在 CentOS 5.x 中，这个目录底下的文件规范了 bash 操作接口的颜色、 语系、ll 与 ls 命令的命令别名、vi 的命令别名、which 的命令别名等等。如果你需要帮所有使用者配置一些共享的命令别名时， 可以在这个目录底下自行创建扩展名为 .sh 的文件，并将所需要的数据写入即可喔！
- /etc/sysconfig/i18n：这个文件是由 /etc/profile.d/lang.sh 加载进来的！这也是我们决定 bash 默认使用何种语系的重要配置文件！ 文件里最重要的就是 LANG 这个变量的配置啦！

#### ~/.bash_profile
在 login shell 的 bash 环境中，所读取的个人偏好配置文件其实主要有三个，依序分别是：
1. ~/.bash_profile
1. ~/.bash_login
1. ~/.profile

其实 bash 的 login shell 配置只会读取上面三个文件的其中一个，而读取的顺序则是依照上面的顺序。也就是说，如果 ~/.bash_profile 存在，那么其他两个文件不论有无存在，都不会被读取。 如果 ~/.bash_profile 不存在才会去读取 ~/.bash_login，而前两者都不存在才会读取 ~/.profile 。 
```

[root@www ~]# cat ~/.bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then   <==底下这三行在判断并读取 ~/.bashrc
        . ~/.bashrc
fi

# User specific environment and startup programs
PATH=$PATH:$HOME/bin        <==底下这几行在处理个人配置
export PATH
unset USERNAME
```

### non-login shell 仅会读取 ~/.bashrc
#### ~/.bashrc
```
[root@www ~]# cat ~/.bashrc
# .bashrc

# User specific aliases and functions
alias rm='rm -i'             <==使用者的个人配置
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then  <==整体的环境配置
        . /etc/bashrc
fi
```
`/etc/bashrc` 帮我们的 bash 定义出底下的数据：
- 依据不同的 UID 规范出 umask 的值；
- 依据不同的 UID 规范出提示字符 (就是 PS1 变量)；
- 呼叫 /etc/profile.d/\*.sh 的配置

这个 /etc/bashrc 是 CentOS 特有的 (其实是 Red Hat 系统特有的)，其他不同的 distributions 可能会放置在不同的档名就是了。由于这个 ~/.bashrc 会呼叫 /etc/bashrc 及 /etc/profile.d/\*.sh ， 所以，万一你没有 ~/.bashrc (可能自己不小心将他删除了)，那么你会发现你的 bash 提示字符可能会变成这个样子：
```
-bash-3.2$
```
这是正常的，因为你并没有呼叫 /etc/bashrc 来规范 PS1 变量！而且这样的情况也不会影响你的 bash 使用。 如果你想要将命令提示字符捉回来，那么可以复制 /etc/skel/.bashrc 到你的家目录，再修订一下你所想要的内容，并使用 source 去调用 ~/.bashrc ，那你的命令提示字符就会回来啦！

### `source` ：读入环境配置文件的命令
利用 `source` 或小数点 (`.`) 都可以将配置文件的内容读进来目前的 shell 环境中！

### 其他相关配置文件
#### /etc/man.config
这的文件的内容 规范了使用 man 的时候， man page 的路径到哪里去寻找！

如果你是以 tarball 的方式来安装你的数据，那么你的 man page 可能会放置在 /usr/local/softpackage/man 里头，那个 softpackage 是你的套件名称， 这个时候你就得以手动的方式将该路径加到 /etc/man.config 里头，否则使用 man 的时候就会找不到相关的说明档。

这个文件内最重要的其实是 `MANPATH` 这个变量配置！ 我们搜寻 man page 时，会依据 `MANPATH` 的路径去分别搜寻啊！

这个文件在各大不同版本 Linux distributions 中，文件名都不太相同，例如 CentOS 用的是 `/etc/man.config` ，而 SuSE 用的则是 `/etc/manpath.config` ， 可以利用 [tab] 按键来进行文件名的补齐！

#### ~/.bash_history
默认的情况下， 我们的历史命令就记录在这里。这个文件能够记录几笔数据，则与 `HISTFILESIZE` 这个变量有关。每次登陆 bash 后，bash 会先读取这个文件，将所有的历史命令读入内存， 因此，当我们登陆 bash 后就可以查知上次使用过哪些命令。

#### ~/.bash_logout
这个文件则记录了 当我注销 bash 后，系统再帮我做完什么动作后才离开 的意思。默认的情况下，注销时， bash 只是帮我们清掉屏幕的信息而已。

