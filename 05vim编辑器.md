# vim编辑器

简介：vim是由vi发展而来的一个文本编辑器，有不同的模式对应不同的功能

```bash
yum -y install vim  # 安装vim
```

命令格式：`vim 文件名`

功能：

- 若文件存在，则打开并编辑文件
- 若文件不存在，则创建并编辑文件

## 1.vim编辑流程

1. `vim 文件名`，默认进入命令模式
2. 按`a/i`切换输入模式
3. 按`Esc`切换命令模式
4. `:`切换底线命令模式
5. `wq`保存并退出，`q!`强制退出

## 2.vim模式的参数

### 2.1命令模式

### 2.2底线命令模式

## 3.修改系统文件

### 3.1修改网卡IP地址

网卡配置文件位于：`/etc/sysconfig/network-scripts/网卡名`

启动网卡命令：`ifup 网卡名`

禁用网卡命令：`ifdown 网卡名`

```bash
yum install net-tools.x86_64  # 安装net-tools包，其中包括ifconfig命令
ifconfig  # 用于显示和设置网卡的参数

# 方法一
vim /etc/sysconfig/network-scripts/ifcfg-ens32
# 修改IPADDR
ifconfig  # 重启网络后才生效
systemctl restart network  # 重启网络
```



```bash
# 方法二
yum -y install bash-completion  # 安装超级补齐工具
# nmcli connection modify --help
# nmcli con mod 网卡名 ipv4.method manual ipv4.addr "192.168.1.2/24, 10.10.1.5/8" connection.autoconnect yes

nmcli connection modify ens32 ipv4.method manual ipv4.address "192.168.0.60/24" connection.autoconnect yes
# nmcli connection modify 修改
# 网卡名 ipv4.method 配置ipv4地址方法
# manual 手动配置
# ipv4.address ipv4地址
# IP地址/掩码 connection.autoconnect yes 开机自动连接
numcli connection up ens32  # 重新激活网卡
# 激活网卡 numcli connection up 网卡名
# 关闭网卡 numcli connection down 网卡名
# 重启网卡 numcli connection reload 网卡名
```



由于IP地址变化，需要重新远程连接

```bash
ifconfig
```



命令格式：`host 域名`

功能：将域名解析为IP地址

```bash
host www.baidu.com
# 若提示-bash: host: command not found，则执行yum -y install bind-utils
```

命令格式：`nslookup 域名`

功能：用于查询域名解析是否正常

```bash
nslookup www.baidu.com
```

```bash
# 返回信息如下
Server:		223.5.5.5
Address:	223.5.5.5#53

Non-authoritative answer:
www.baidu.com	canonical name = www.a.shifen.com.
Name:	www.a.shifen.com
Address: 110.242.68.3
Name:	www.a.shifen.com
Address: 110.242.68.4

```

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens32
# 修改DNS为223.4.5.5，修改后，重启生效
systemctl restart network

nslookup www.baidu.com
# 报错信息如下
# ;; connection timed out; no servers could be reached
```



命令格式：`alias 别名='命令'`

功能：用于设置命令别名，与C语言中的`typedef`有异曲同工之妙

取消临时别名：`unalias 别名`

不同用户定义别名的关系：谁定义谁使用

临时别名

```bash
alias  # 查看当前系统可用别名

# 观察以下两个命令输出的异同
ls /  # 输出有高亮
\ls /  # 输出无高亮，\可以使别名功能暂时失效

cat -n /root/.bashrc  # 命令别名存放于.bashrc文件中

alias lh='ls -lah --color=auto'  # =两边不能有空格，命令中有空格时要用''包裹，此为临时别名
lh /

unalias lh  # 取消别名
lh /
# 报错：-bash: lh: command not found

alias ls=reboot  # 定义别名不要与系统命令冲突
ls  # 执行关机命令，别名优先级高于系统命令
```

永久别名

```bash
vim /root/.bashrc  # 修改配置文件，可实现永久别名
# alias lh='ls -lah --color=auto'

source .bashrc  # 重新读取.bashrc文件，使配置文件生效
```

命令格式：`history`

功能：查看历史命令

常用选项：

- `-a`，将当前缓存中的命令历史写入`.bash_history`文件中
- `-d`，指定删除某条历史命令
- ` -c`清空历史命令列表
- `!number`
- `!命令`
- `!!`重复执行上一条命令

默认记录`1000`条，先将命令记录于系统缓存，用户`exit`后记录入`.bash-history`文件

修改历史记录条数

```bash
vim /etc/profile  # 编辑history配置文件
```

在命令模式下输入 `:46`

将`HISTSIZE=1000`改为`HISTSIZE=100`

```bash
source /etc/profile
exit  # 退出重新登录
cat -n /root/.bash_history  # 查看历史命令，核对条数
```



命令格式：`date`

功能：查看历史命令

NTP标准时间服务器

```bash
date
# CST表示亚洲上海时区
date +%Y  # 只显示年份
date +%F  # 显示年-月-日
# 2023-07-01

```



命令格式：`wc -[选项] 文件名`

功能：统计文件的字节数、行数，并输出

一个汉字在`gbk`编码下占`2`字节，在`utf-8`编码下占`3`字节

常用选项：

- ` -c `统计字节数
- ` -l`统计行数

```bash
wc /etc/passwd
#   23   49 1141 /etc/passwd
#   行数  单词数 字节数  文件名
wc -c /etc/passwd
```



### |管道符

管道符`|`将命令的输出结果交给另一条命令作为参数继续处理

管道没有上限

```bash
cat -n /etc/passwd | head | tail -5  # 查看passwd文件的第6-10行
ifconfig ens32 | head -2  # 查看网卡信息的前两行
```

### grep文件内容过滤

命令格式：`grep [-选项] "查找内容" 目标文件`

功能：`grep`用于查找文件中符合条件的字符串

常用选项：

- `-n`显示行号
- `-i`忽略大小写
- `-v`排除匹配的内容
- ` ^`匹配开头关键字
- `$`匹配结尾关键字
- `^$`匹配空行

```bash
grep -n "root" /etc/passwd | wc -l  # 统计包含root字符串的行数
grep -n "^root" /etc/passwd  # 输出以root开头的行
grep -n "bash$" /etc/passwd  # 输出以bash结尾的行

egrep -v "#|^$" /etc/login.defs | wc -l  # 统计有效行，即排除空行、#开头的注释行

```



shell四剑客

- `grep`文件内容过滤
- `find`查找文件或目录的存放路径
- `which`查找程序文件
- `sed`非交互式文本编辑器
- `awk`文件内容过滤（支持列过滤）



`free -h`查看当前内存信息



### 重定向

功能：将前面命令的输出结果，写入到其它的文本文件

重定向符号：

- `>`覆盖重定向，只收集正确输出内容
- `>>`追加重定向，只收集正确输出内容
- `cat > 文件名<< EOF`输入覆盖重定向
- `cat >> 文件名 << EOF`输入追加重定向
- `2>`覆盖重定向，只收集错误输出内容
- `2>>`追加重定向，只收集错误输出内容
- `&>`覆盖输出内容全收集
- `&>>`追加输出内容全收集

```bash
egrep -nv "#|^$" /etc/login.defs > test.txt  # 将命令的输出重定向(写入)至test.txt文件中
free -h >> test.txt  # 追加重定向(追加写入)
cat > test.c << EOF  # 将命令行的输入，存储到test.c文件中
lscat /etc/passwd 2> text_error.txt
```



命令格式：`echo [-选项] [参数]`

功能：用于输出指定的字符串和变量

```bash
echo $PATH

echo HelloWorld

```



