# 简述
  为了便于argo 社区用户更好的体验, 简化安装安装步骤. 但是切记按照不是要求准备服务器!
## 安装前服务器检测
1. 使用 root 或者具有 sudo 权限的用户操作
1. 下载构建包 `$ wget http://arkinstall.analysys.cn/upgrade/go.tar.gz`
1. 解压构建包 (解压到当前目录且不要删除 go.tar.gz ) `$ tar zxf go.tar.gz` 
1. 拷贝配置文件模板到当前目录  `$ cp go/files/scatter/sys.conf /tmp/` 
1. 修改配置参数 (IP:内网IP | PORT:ssh 端口 | USER:ROOT | PASSWD:root 用户的密码) `$ vim sys.conf`
1. 进如工作目录 `$ cd go`
1. 测试 `$ bin/python3 tools/pre.py -t -c /tmp/sys.conf`  结果为 pass 方可继续操作
1. 修改主机名和hosts `$ bin/python3 tools/pre.py -s -c /tmp/sys.conf` 主机名会被改成ark1 
1. 构建 argo 用户 `$ bin/python3 tools/pre.py -u argo -c /tmp/sys.conf`   
1. 切换 `$argo su - argo`
1. 检测 `$python3 go/tools/pre.py -x -c /tmp/sys.conf` 结果为 pass 方可继续操作
     
## 开始安装
1. 下载安装包`$ http://ark_install.analysys.cn/argo.4.6.tar.gz`  
1. 创建安装目录`$ sudo mkdir /opt/soft` 
1. 解压`$ sudo tar zxf -C /opt/soft` 
1. 进入安装目录 `$ cd /opt/soft`
1. 安装`$ sudo sh standalone_offline_installer.sh` 
    
## 开始使用
   打开web界面会提示输入license
## 后续升级
    参见 upgrede
     
