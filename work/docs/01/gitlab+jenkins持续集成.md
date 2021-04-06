## gitlab安装
### 安装依赖软件
```bash
## 相关依赖
yum install -y policycoreutils openssh-server openssh-clients postfix

## 设置postfix开机启动，postfix支持gitlab发信功能
systemctl enable postfix && systemctl start postfix
```
### 下载gitlab包并安装
```bash
## 下载安装包
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-12.10.10-ce.0.el7.x86_64.rpm

## 安装
rpm -ivh gitlab-ce-12.10.10-ce.0.el7.x86_64.rpm
```
### 修改配置文件并启动
```bash
## 修改配置文件，主要是修改端口，默认为80
vim /etc/gitlab/gitlab.rb
## 修改为如下：
external_url 'http://192.168.0.198:8081'
nginx['listen_port'] = 8081

## 重新加载配置文件&&启动
gitlab-ctl reconfigure
gitlab-ctl restart
```
### 设置默认密码
- 等待启动完成后访问`http://192.168.0.198:8081`,首次访问需要按提示设置`root`密码

### 502错误
- 一开始端口改成`8080` 发现一直502,原来是服务自带tomcat服务，设置成`8080`一只占用了，所以服务起不来

## gitlab管理
