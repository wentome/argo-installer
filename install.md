# 简述
&ensp;&ensp;&ensp;&ensp;**为了便于Argo社区用户更好的体验, 本版简化了安装步骤，但切记要按照要求准备配置环境，否则由配置引发的问题我们不予支持哦!**

## 安装前服务器检测和配置
1. 服务器最低配置要求. 系统：centos7.4|6|7 , CPU:4核8线, 支持 avx 指令集, 内存:16g, 系统盘大于200G, 数据盘 data1大于500G ,且不要使用磁盘分区
1. 使用 root 用户操作
1. 下载构建包 `$ wget http://arkinstall.analysys.cn/upgrade/go.tar.gz`
1. 解压构建包 (解压到当前目录且不要删除 go.tar.gz ) `$ tar zxf go.tar.gz` 
1. 拷贝配置文件模板到默认目录  `$ cp go/files/scatter/sys.conf /tmp` (如果使用其他路径在后续的命令中用-c /path/sys.conf 参数指定)
1. 修改配置参数 (IP:内网IP | PORT:ssh 端口 | USER:root | PASSWD:root 用户的密码) `$ vim /tmp/sys.conf`
1. 进入工作目录 `$ cd go`
1. 测试ssh连接 `$ bin/python3 tools/pre.py -t`  结果为 pass 方可继续操作
1. 修改主机名和hosts `$ bin/python3 tools/pre.py -s` 主机名会被改成ark1 
1. 构建 argo 用户 `$ bin/python3 tools/pre.py -u argo`
1. 备份日志 `$ mv /tmp/pre.log /tmp/pre.log.install`
1. 切换到 argo 用户 `$ su - argo`
1. 挂载数据盘 /dev/sdb -> /data1 (默认配置如有特殊可修改 /tmp/sys.conf) `$ python3 go/tools/pre.py -md`
1. 检测 `$ python3 go/tools/pre.py -x` 结果均为 pass 方可继续操作 如有疑问, 截图检测结果到社区群，咨询技术小伙伴
1. 环境检测均通过后初始化环境`$ python3 go/tools/pre.py -init`
     
## 开始安装
1. 切换到 argo 用户 `$ su - argo`
1. 下载安装包`$ wget http://arkinstall.analysys.cn/argo.4.6.tar.gz`  
1. 解压`$ tar zxf argo.4.6.tar.gz`  
1. 复制安装文件 `$ sudo cp argo.4.6/* /opt/soft`
1. 进入安装目录 `$ cd /opt/soft`
1. 安装`$ sudo sh standalone_offline_installer.sh`  为避免安装过程断网导致安装失败, 建议使用 screen 命令
    
## 开始使用
1. 在这里获取license：https://ark.analysys.cn/license.html
1. 部署完成后，登录平台管理员帐号（admin 111111），根据页面引导，输入 License 激活系统，进入项目管理页面，即可开始创建项目。
1. 在集成SDK/可视化埋点后可开始利用方舟产品能力创建分析模型，开启分析之旅。

在安装使用过程中如有问题，请优先从已有文档中查找答案：
1. 官方产品文档：https://docs.analysys.cn/ark/ 
2. 论坛：https://ark.analysys.cn/forum/

当自研无法解决问题，可以通过我们的顾问进入Argo微信群提问，提问时请遵守社区规范，群内小伙伴都是自发回答。

## 后续升级
参见首页 upgrade 文档
     
