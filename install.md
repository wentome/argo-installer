# 简述
  为了便于argo 社区用户更好的体验 ，简化安装步骤。
## 部署前环境配置检测
     下载工具包 wget http://ark_install.analysys.cn/upgrade/go.tar.gz
     解压 tar zxf go.tar.gz 
     修改配置文件 cp go/files/sctxx/sys.conf /tmp  编辑配置文件  vim /tmp/sys.conf
     检测网络连接bin/python3 tools/pre.py -t -c /tmp/sys.conf
     修改主机名 bin/python3 tools/pre.py -s -c /tmp/sys.conf
     创建 argo 用户 bin/python3 tools/pre.py -u argo -c /tmp/sys.conf
     切换 argo su - argo
     python3 go/tools/pre.py -x -c /tmp/sys.conf
     
## 开始安装
    下载安装包
    解压
    安装
    
## 开始使用
   license
     
