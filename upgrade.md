## 系统升级
  * 系统升级包含大版本、小版本两种升级
  * 大版本升级主要做架构调整，新功能发布
  * 小版本升级主要做 bug 修复，系统参数调整
## 升级前准备
1. 低于4.3.5 版本需要创建argo用户  
2. 升级操作需要切换到 argo 用户操作 (`$ su - argo`)
## 大版本升级
#### 1. 在线升级    
  `$ upgrader -ma`    自动升级到最新版本 n.n.x000
#### 2. 离线升级  
  `$ upgrader -ma -l argoma.x.x.x.tar.gz`
## 小版本升级
#### 1. 在线升级  
  `$ upgrader -mi`     自动升级到最新版本 n.n.nxxx
#### 2. 离线升级
   `$ upgrader -mi -l argomi.x.x.x.tar.gz`

