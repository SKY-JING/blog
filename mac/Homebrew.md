<div style="text-align: center; font-weight: 700; font-size: 2em;">Homebrew安装及使用</div>

# 一、安装
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

# 二、移除
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
```
# 三、基本使用

```shell
// 搜索包
brew search telnet

// 安装包
brew install telnet

// 查看包信息，比如目前的版本，依赖，安装后注意事项等
brew info telnet

// 卸载包
brew uninstall telnet

// 显示已安装的包
brew list

// 查看brew的帮助
brew –help

// 更新， 这会更新 Homebrew 自己
brew update

// 检查过时（是否有新版本），这会列出所有安装的包里，哪些可以升级
brew outdated
brew outdated telnet

// 升级所有可以升级的软件们
brew upgrade
brew upgrade telnet

// 清理不需要的版本极其安装包缓存
brew cleanup
brew cleanup telnet
```