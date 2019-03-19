### 《Linux系统与网络管理》实验二：From GUI to CLI
#### 软件环境
+ ubuntu-18.04 server 64bit
+ 在asciinema注册一个账号，并在本地安装配置好asciinema
#### 实验内容
+ 确保本地已经完成asciinema auth，并在asciinema成功关联了本地账号和在线账号
+ 上传本人亲自动手完成的vimtutor操作全程录像
+ 在自己的github仓库上新建markdown格式纯文本文件附上asciinema的分享URL
+ 完成vimtutor自查清单
#### 实验步骤
1. 安装asciinema（根据asciinema官网资料）
+ 在Debian上安装语句为```sudo apt-get install 在asciinema```
+ 在Ubuntu上安装语句为
```
sudo apt-add-repository ppa:zanchey/asciinema
sudo apt-get update
sudo apt-get install asciinema
```
根据自己尝试，仅```sudo apt-get install 在asciinema```即可
2. 关联账号
```asciinema auth```
3. 录像操作
```
#开始录制
asciinema rec
#结束录制
<ctrl-d> or type "exit"

##### vimtutor操作录像

##### asciinema的分享URL

##### vimtutor完成后的自查清单
1. vim的几种工作模式
+ Normal模式
+ Insert模式
+ Visual模式
2. Normal模式下
```
#从当前行开始，一次向下移动光标10行的操作方法
10j
#快速移动到文件开始行
gg
#快速移动到文件结束行
G
#快速跳转到文件中的第N行
Ngg
```
3. Normal模式下
```
#删除单个字符
x
#删除单个单词
dw
#从当前光标位置一直删除到行尾
d$
#从当前光标位置一直删除到单行
dd
#从当前光标位置一直删除到当前行开始向下数N行
dNd
```
4. 如何在vim中快速插入N个空行？如何在vim中快速输入80个-？
```
#快速插入N个空行
No <ESC>
#快速插入80个-
插入模式下CTRAL+0 80i- <ESC>
```
5. 如何撤销最近一次编辑操作？如何重做最近一次被撤销的操作？
```
#撤销最近一次编辑操作
u
#重做最近一次被撤销的操作
ctrl+R
```
7. vim中如何实现剪切粘贴单个字符？单个单词？单行？如何实现相似的复制粘贴操作呢？
```
#剪切粘贴单个字符
x
#剪切单个单词
dw
#剪切单行
d$
#粘贴
p
#相似的复制粘贴操作：光标停留在需要复制的起始字符上，v进入Visual模式，选择需要复制的文字(高亮），选好后y复制，光标移动到需要粘贴的地方，p粘贴。
```
8. 为了编辑一段文本你能想到哪几种操作方式（按键序列）？
9. 查看当前正在编辑的文件名的方法？查看当前光标所在行的行号的方法？
```
#查看当前正在编辑的文件名
Ctrl+G
#查看当前光标所在行的行号的方法
Ctrl+G
```
10. 在文件中进行关键词搜索你会哪些方法？如何设置忽略大小写的情况下进行匹配搜索？如何将匹配的搜索结果进行高亮显示？如何对匹配到的关键词进行批量替换？
```
#在文件中进行关键词搜索
/+关键字
#设置忽略大小写的情况下进行匹配搜索
:set ignorecase
#设置高亮查找
:set hlsearch
#将当前文件中全替换a为b::%s/a/b/g
```
11. 在文件中最近编辑过的位置来回快速跳转的方法？
``` `` ```
12. 如何把光标定位到各种括号的匹配项？例如：找到(, [, or {对应匹配的),], or }
``` % ```
13. 在不退出vim的情况下执行一个外部程序的方法？
``` :!ls ```
14. 如何使用vim的内置帮助系统来查询一个内置默认快捷键的使用方法？如何在两个不同的分屏窗口中移动光标？
```
#使用vim的内置帮助系统来查询一个内置默认快捷键的使用方法
:help w
#在两个不同的分屏窗口中移动光标
Ctrl+W w
```
##### 参考资料
