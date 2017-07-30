# 数据流重导向

## 标准输出与标准错误输出

- 标准输出指的是命令运行所回传的正确的信息。
- 标准错误输出可理解为命令运行失败后，所回传的错误信息。

数据流重导向可以将 standard output (简称 stdout) 与 standard error output (简称 stderr) 分别传送到其他的文件或装置去，而分别传送所用的特殊字符则如下所示：

- 标准输入(stdin)：代码为 `0` ，使用 `<` 或 `<<`
- 标准输出(stdout)：代码为 `1` ，使用 `>` 或 `>>`
- 标准错误输出(stderr)：代码为 `2` ，使用 `2>` 或 `2>>`

### `>`

若以 `>` 输出到一个不存在的文件，系统会自动创建该文件；若以 `>` 输出到一个已存在的文件中，系统就会先将这个文件内容 **清空** ，然后再将数据写入！

### `>>`

若以 `>>` 输出到一个不存在的文件，系统会自动创建该文件；若以 `>>` 输出到一个已存在的文件中，数据会在该文件的最下方 **累加** 进去！

```bash
范例一：利用一般身份账号搜寻 /home 底下是否有名为 .bashrc 的文件存在
$ find /home -name .bashrc
find: /home/lost+found: Permission denied  <== Standard error
find: /home/alex: Permission denied        <== Standard error
find: /home/arod: Permission denied        <== Standard error
/home/unixuser/.bashrc                     <== Standard output
```

```bash
范例二：承范例一，将 stdout 与 stderr 分存到不同的文件去
$ find /home -name .bashrc > list_right 2> list_error
```

## `/dev/null` 垃圾桶黑洞装置与特殊写法

```bash
范例三：承范例二，将错误的数据丢弃，屏幕上显示正确的数据
$ find /home -name .bashrc 2> /dev/null
/home/unixuser/.bashrc  <==只有 stdout 会显示到屏幕上， stderr 被丢弃了
```

```bash
范例四：将命令的数据全部写入名为 list 的文件中
$ find /home -name .bashrc > list 2> list  <==错误
$ find /home -name .bashrc > list 2>&1     <==正确
$ find /home -name .bashrc &> list         <==正确
```

## 标准输入：`<` 与 `<<`

- `<`：将原本需要由键盘输入的数据，改由文件内容来取代。
- `<<`：代表的是 结束的输入字符。

```bash
范例：用 cat 直接将输入的信息输出到 catfile 中， 且当由键盘输入 eof 时，该次输入就结束
$ cat > catfile << "eof"
> This is a test.
> OK now stop
> eof  <==输入这关键词，立刻就结束而不需要输入 [ctrl]+d

$ cat catfile
This is a test.
OK now stop     <==只有这两行，不会存在关键词那一行！
```

利用 `<<` 右侧的控制字符，我们可以终止一次输入， 而不必输入 <kbd>Ctrl + d</kbd> 来结束。

## 使用命令输出重导向的场景

- 屏幕输出的信息很重要，而且我们需要将他存下来的时候；
- 背景运行中的程序，不希望他干扰屏幕正常的输出结果时；
- 一些系统的例行命令 (例如写在 /etc/crontab 中的文件) 的运行结果，希望他可以存下来时；
- 一些运行命令的可能已知错误信息时，想以 `2> /dev/null` 将他丢掉时；
- 错误信息与正确信息需要分别输出时。



