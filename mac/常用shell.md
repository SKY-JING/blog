<div style="text-align: center; font-weight: 700; font-size: 2em;">常用shell命令</div>

# 一、基础指令
## 1.1 scp
上传文件到服务器或从服务器下载文件
```
scp [-r/上传整个文件夹] 本地文件 用户名@服务器ip:服务器地址
scp [-r] 用户名@服务器ip:服务器地址 本地路径
```

## 1.2 mkdir
创建文件夹,可以使用-p选项来一起创建路径中不存在的文件夹
```
mkdir 文件夹名
```

## 1.3 rm
删除文件夹，如果想安全删除文件请使用srm
```
rm -rf 文件夹名(r向下递归，不过多少目录，一并删除。f强行删除不做任何提示)
```

## 1.4 cp
拷贝文件到另一个目录
```
cp 目录一 目录次二
-r 拷贝整个文件夹
-f 不保留移动日志
```

## 1.5 man
帮助文档：上下箭头查看，q退出，空格翻页,输入/和关键字进行搜索
```
man ls
```

## 1.6 pwd
显示当前目录的绝对路径
```
pwd
```

## 1.7 cd
切换路径（路径种有空格或者特殊字符需要用反斜杠‘\’转义，也可直接将要切换到的文件直接拖入终端，还可使用tab自动补充）
```
cd
```

## 1.8 ls
列表形式显示文件夹下文件：-a参数显示隐藏文件,-l控制输出格式，-A显示.(当前文件夹),..(父文件夹)
```
ls
```

## 1.9 cat
顺序读取文件输出到终端窗口，语法为cat后接需要查看的文件路径，cat也可以使用>>来增加文本文件的内容，如cat ../textOne.txt >> textTwo.txt会把 textOne.txt 的内容添加到 textTwo.txt 的结尾
```
cat
```

## 1.10 less
功能类似cat的查看文本，使用同man，多了一个按v键使用vi文本编辑器
```
less
```

## 1.11 which
查看某个命令的文件路径（如which man ls pwd cd）
```
which
```
## 1.12 file
根据文件内容输出文件类型
```
file
```

## 1.13 find
根据搜索关键词定位文件路径,搜索根目录时使用-x避免搜索／Volumes文件夹，如果想使用Soptlight搜索服务，使用mdfind命令接搜索关键字（如find Desktop -name “*.png”）
```
find
```
## 1.14 通配符
```
\*：表示任意长度的任何字符，如\*.tiff代表所有格式为tiff的文件
?：表示任意单个字符，如b?ok匹配book但是不匹配brook
[]：定义一定范围的字符，如[Dd]ocument匹配Document和document,doc[1-9]匹配doc1-doc9
```

## 1.15 mv
移动文件，参数同cp
```
mv
```

## 1.16 vi
打开终端的文本编辑器,Mac os x中nano也可打开文本编辑器，和less命令类似，vi命令会占用整个 Terminal 空间来显示文件内容。打开后，在“command模式”，vi 会等你输入一些预定义字符来告诉 vi 你想做什么。你也可以使用键盘上的箭头键单纯地浏览文件。你想编辑时，按A开始（会进入编辑模式）。文字会插入到光标处。如果你想保存，需要先退出编辑模式进入 command 模式。方法是按下esc键。回到 command 模式后，按住shift同时按两次Z来保存并退出。如果你不想保存，在 command 模式输入:quit!并按enter return直接退出
```
vi
```

## 1.17 su
切换用户，使用who -m命令验证当前登陆身份
```
su
```

## 1.18 sudo
获取系统root权限，sudo -s临时切换整个shell来获取root级别权限
```
sudo
```

## 1.19 history
查看历史记录
```
history
```

## 1.20 open
用finder打开当前的位置
```
open .
```

# 二、功能指令
## 2.1 获取ip地址
```
ifconfig | grep "inet " | grep -v 127.0.0.1
```
## 2.2 重启
```
sudo shutdown -r now
```
## 2.3 电源管理/省电
```
// 获取当前电源管理设置的信息
pmset -g

// 计算机在无活动30分钟后休眠
sudo pmset sleep 30

// 设置显示器五活动15分钟后关闭
sudo pmset displaysleep 15
```
## 2.4 os x 外观
```
// 禁用仪表盘
defaults write com.apple.dashboard mcx-disabled -boolean YES killall Dock

// 启用仪表盘
defaults write com.apple.dashboard mcx-disabled -boolean NO killall Dock

// 强制Finder程序显示隐藏文件
defaults write com.apple.finder AppleShowAllFiles TRUE

// 强制Finder程序不显示隐藏文件
defaults write com.apple.finder AppleShowAllFiles FALSE
```
## 2.5 网络
```
// ping 某个主机
ping -o oschina.net

// 使用traceroute 诊断到某个主机的陆游节点
traceroute oschina.net

// 检查某个主机是否运行HTTP服务，或者检查某网站是否可用
curl -I www.oschina.net | head -n 1

// 管理windows网络(相当于Windows下的NET命令)，该命令有很多选项，运行下面命令来查看这些选项
man net

// 使用dig来诊断域名信息
dig www.oschina.net A
dig www.oschina.net MX

// 查看谁正在登录到你的Mac机器
w

// 显示系统路由表
netstat -r

// 显示活动网络连接
netstat -an

// 显示网络统计
netstat -s
```
## 2.6 故障诊断
```
// 列表所有打开的文件
lsof

// 启动Bonjour -- 当网络中没有Mac时很有用
sudo launchctl unload /System/Library/LaunchDaemons/com.apple.mDNSResponder.plist
sudo launchctl load /System/Library/LaunchDaemons/com.apple.mDNSResponder.plist

// 弹出CD（注意不一定是disk1）
diskutil eject disk1
```
## 2.7 文本操作命令
```
// 统计剪贴板中文本的行数
pbpaste | wc -l

// 统计剪贴板中文本的单词数
pbpaste | wc -w

// 对剪贴板中的文本行进行排序后重新写回剪贴版
pbpaste | sort | pbcopy

// 对剪贴版中的文本行进行倒序后放回剪贴板
pbpaste | rev | pbcopy

// 移除剪贴版中重复的文本行，然后写回剪贴板
pbpaste | sort | uniq | pbcopy

// 找出剪贴板中文本中存在的重复行，并复制后写回剪贴板(包含重复行的一行)
pbpaste | sort | uniq -d | pbcopy

// 找出剪贴板中文本中存在的重复行，并复制后写回剪贴板(不包含重复行)
pbpaste | sort | uniq -u | pbcopy

// 对剪贴板中的HTMl文本进行清理后写回剪贴板
pbpaste | tidy | pbcopy

// 显示剪贴板中文本的前五行
pbpaste | head -n 5

// 显示剪贴板中文本的最后五行
pbpaste | tail -n 5

// 将剪贴板中文本里存在的Tab跳格符号转成空格
pbpaste | expand | pbcopy
```
## 2.8 iTunes相关
```
// 更改iTunes链接行为为本机iTunes库，而不是iTunes store
defaults write com.apple.iTunes invertStoreLinks -bool YES

// 更改iTunes链接行为为iTunes store，而不是本机iTunes库
defaults write com.apple.iTunes invertStoreLinks -bool NO
```
## 2.9 其他命令
```
// A:
htpasswd -nb username password

// B:
AuthType Basic
AuthName "restricted area"
AuthUserFile /path/to/you/site/.htppasswd
require valid-user

// 显示终端窗口中之前输入的命令
history

// 将文件转成HTML，支持格式包括Text,.RTF,.DOC
textutil -convert html file.extension

// Nano是一个很简单易用的文本编辑器，可用于快速更改文本文件,比vim功能弱很多，但是很方便
nano [file_to_esit]

// 在nona编辑器中，可食用ctrl+0来保持，ctrl+x来退出

// 清理终端显示的内容
clear
```
## 2.10 其他Mac OS X 终端资源
```
// 前面的很多例子我们用了 pbpaste 来从剪贴板中获取数据，也可使用 cat 来从文件中获取数据
cat [/path/to/filename]
```