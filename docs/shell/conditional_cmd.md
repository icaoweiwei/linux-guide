# 有条件的命令

## 不考虑命令相关性的连续命令下达 `cmd;cmd`
在命令与命令中间利用分号 (`;`) 来隔开，这样一来，分号前的命令运行完后就会立刻接着运行后面的命令了。

## `$?`(命令回传值) 与 `&&` 或 `||`
若前一个命令运行的结果为正确，在 Linux 中会回传一个 $? = 0 的值。

- `cmd1 && cmd2`
1. 若 cmd1 运行完毕且正确运行($?=0)，则开始运行 cmd2。
2. 若 cmd1 运行完毕且为错误 ($?≠0)，则 cmd2 不运行。
- `cmd1 || cmd2`
1. 若 cmd1 运行完毕且正确运行($?=0)，则 cmd2 不运行。
2. 若 cmd1 运行完毕且为错误 ($?≠0)，则开始运行 cmd2。

```
范例：以 ls 测试 /tmp/somefile 是否存在，若存在则显示 "exist" ，若不存在，则显示 "not exist"！
$ ls /tmp/somefile || echo "not exist" && echo "exist"
ls: 无法访问/tmp/somefile: 没有那个文件或目录
not exist
exist

$ ls /tmp/somefile && echo "exist" || echo "not exist"
ls: 无法访问/tmp/somefile: 没有那个文件或目录
not exist
```
解析：
1. 若 `ls /tmp/somefile` 不存在，因此回传一个非 0 的数值；
1. 接下来经过 `||` 的判断，发现前一个命令回传非 0 的数值，因此，程序开始运行 `echo "not exist"` ，而 `echo "not exist"` 程序肯定可以运行成功，因此会回传一个 0 值给后面的命令；
1. 经过 `&&` 的判断，咦！是 0 啊！所以就开始运行 `echo "exist"` 。最终同时出现 not exist 与 exist 。

由于命令是一个接着一个去运行的，因此，如果真要使用判断， 那么这个 `&&` 与 `||` 的顺序就不能搞错。一般来说，假设判断式有三个，也就是：
```
command1 && command2 || command3
```
