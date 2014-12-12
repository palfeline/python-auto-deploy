# Python自动化打包业务和认证平台 V2.1-Release #

## 1.文档摘要 ##

> Python 自动化打包业务和认证平台，本机只需执行脚本，远程即可自动部署。脚本采用Python编写，远程调用使用Fabric实现。

## 2.更新日志 ##

2014-11-28
> 文档版本为「1.0」，文档名为「Python自动化打包业务和认证平台 V1.0」，备注为「文档正式版，已测试通过」，By Robin。

2014-12-04
> 文档版本为「2.0」，文档名为「Python自动化打包业务和认证平台 V2.0-Release」，备注为「文档正式版第二版，修复若干Bug」，By Robin。

2014-12-12
> 文档版本为「2.1」，文档名为「Python自动化打包业务和认证平台 V2.1-Release」，备注为「添加部署Tomcat脚本」，By Robin。

## 3.版本信息 ##

本机 XXX：

> 系统版本：<br/>
> 主机名：XXX <br/>
> IP：xxx.xxx.xxx.xxx <br/>
> Python：2.6.6

远程服务器 XXX：

> 系统版本：XXX <br/>
> 主机名：XXX <br/>
> IP：xxx.xxx.xxx.xxx <br/>
> Python：2.7.3 <br/>
> JDK：1.8.25 <br/>
> Maven：3.2.3 <br/>
> SVN：1.6.17

Tomcat服务器 XXX：

> 系统版本：XXX <br/>
> 主机名：XXX <br/>
> IP：xxx.xxx.xxx.xxx

## 4.先决条件 ##

**本机安装软件：**

> Python 2.7.5

安装包如下：

> apt-get: python python-pip python-dev subversion subversion-tools <br/>
> yum: python python-pip python-devel subversion <br/>
> pip: fabric

**远程服务器安装软件：**

> JDK：JDK 1.6.45 1.7.71 1.8.25 方便不同版本切换<br/>
> Maven：3.2.3 <br/>
> SVN：1.6.17

安装包如下：

> dos2unix subversion subversion-tools

**Tomcat服务器安装软件：**

> JDK：JDK 1.6.45 1.7.71 1.8.25 方便不同版本切换<br/>
> Tomcat：6.0.45 <br/>
> SVN：1.6.17 <br/>
> ant：1.9.4 用于打包 <br/>
> Jmeter：2.12 用于后期做自动化测试扩展

## 5.软件综述 ##

本软件包括三个目录，其中auto_deploy_app_v2为Linux版本。auto_deploy_app_windows为Windows版本。Linux版本包括两个Python脚本以及一个配置文件。Windows版本包括两个Python脚本以及两个配置文件。auto_deploy_app_to_tomcat为部署到Tomcat脚本，其中包括两个Python脚本、一个Python脚本配置文件、一个ant build.xml、一个crontab配置文件、五个Shell脚本。

## 6.脚本详解之Linux版本 ##

### 6.1 软件概要 ###

Linux版目录结构如下：

	tree auto_deploy_app_v2

> auto_deploy_app_v2 <br/>
> |-- auto_deploy_app_remote.py <br/>
> |-- auto_deploy_app_v_final.py <br/>
> `-- config.conf <br/>
> <br/>
> 0 directories, 3 files

其中，「auto_deploy_app_remote.py」是主执行脚本，用于显示帮助以及调用相应函数。「auto_deploy_app_v_final.py」是核心执行脚本，实现所有的相关功能。「config.conf」是脚本的配置文件。

该脚本实现的功能如下：

* 打印帮助
* 部署准备
* 检出项目
* 更新项目
* 部署业务平台
* 部署认证平台
* 启动、关闭、重启业务平台
* 启动、关闭、重启认证平台
* 修改数据库配置

### 6.2 脚本帮助 ###

我们通过如下命令可以获得该脚本的帮助。

	./auto_deploy_app_remote.py -h

```bash
Auto deploy application to the remote web server. Write in Python.
 Version 1.0. By Robin Wen. Email:dbarobinwen@gmail.com
 
 Usage auto_deploy_app.py [-hcustrakgdwp]
   [-h | --help] Prints this help and usage message
   [-p | --deploy-prepare] Deploy prepared. Run as root
   [-c | --svn-co] Checkout the newarkstg repo via svn
   [-u | --svn-update] Update the newarkstg repo via svn
   [-s | --shutdown-core] Shutdown the core platform via the stop.sh scripts
   [-t | --startup-core] Startup the core platform via the startup.sh scripts
   [-r | --restart-core] Restart the core platform via the restart.sh scripts
   [-a | --shutdown-auth] Shutdown the auth platform via the stop.sh scripts
   [-k | --startup-auth] Startup the auth platform via the startup.sh scripts
   [-g | --restart-auth] Restart the auth platform via the restart.sh scripts
   [-d | --deploy-core-platform] Deploy core platform via mvn
   [-w | --deploy-auth-platform] Deploy auth platform via mvn
   [-x | --update-database-setting] Update the database setting
```

在脚本名后加上「-h 或者 --help」表示打印帮助。
同理，加上「-p | --deploy-prepare」表示部署准备，加上「-c | --svn-co」表示检出项目，加上「-u | --svn-update」表示更新项目，加上「-s | --shutdown-core」表示关闭业务平台，加上「-t | --startup-core」表示启动业务平台，加上「-r | --restart-core」表示重启业务平台，加上「-a | --shutdown-auth」表示关闭认证平台，加上「--startup-auth」表示启动认证平台，加上「-g | --restart-auth」表示重启认证平台，加上「-d | --deploy-core-platform」表示部署业务平台，加上「-w | --deploy-auth-platform」表示部署认证平台，加上[-x | --update-database-setting]表示修改数据库配置。

### 6.3 脚本概述 ###

如前所述，「auto_deploy_app_remote.py」是主执行脚本，用于显示帮助以及调用相应函数。「auto_deploy_app_v_final.py」是核心执行脚本，实现所有的相关功能。核心执行脚本采用Fabric实现远程执行命令，主执行脚本再通过**fab -f 脚本名 任务名**调用相应方法。

主执行脚本和核心执行脚本的方法名基本一致，主执行脚本包括如下方法：main(argv)、usage()、svn_co()、svn_update()、shutdown_core()、startup_core()、restart_core()、shutdown_auth()、startup_auth()、restart_auth()、deploy_core_platform()、deploy_auth_plaform()、deploy_prepare()和updata_database_setting()。

核心执行脚本包括如下方法：main(argv)、usage()、svn_co()、svn_update()、shutdown_core()、startup_core()、restart_core()、shutdown_auth()、startup_auth()、restart_auth()、deploy_core_platform()、deploy_auth_platform()、deploy_prepare()、updata_database_setting()和getConfig()。

**主执行脚本：**

* main(argv) 主函数
* usage() 使用说明函数
* svn_co() 检出项目函数
* svn_update() 更新项目函数
* shutdown_core() 关闭业务平台方法
* startup_core() 启动业务平台方法
* restart_core() 重启业务平台方法
* shutdown_auth() 关闭认证平台方法
* startup_auth() 启动认证平台方法
* restart_auth() 重启认证平台方法
* deploy_core_platform() 部署业务平台方法
* deploy_auth_platform() 部署认证平台方法
* deploy_prepare() 部署准备方法
* updata_database_setting() 修改数据库配置方法。

**主执行脚本**

主执行脚本内容如下：
参考脚本auto_deploy_app_remote.py。

**核心执行脚本**

方法和主执行脚本基本一致，相同的不赘述。核心执行脚本还提供getConfig()方法，用于读取配置文件。

核心执行脚本内容如下：
参考脚本auto_deploy_app_v_final.py。

### 6.4 配置文件概述 ###

完整配置文件内容如下：

```bash
# Database config section.
[database]
# Database address.
db_addr=
# Database port.
db_port=
# Database username.
db_usr=
# Datbase password.
db_pwd=

# Remote server section.
[remote]
# Remote server ip.
remote_ip=
# Remote server port.
remote_port=
# Remote server username.
remote_usr=
# Remote server password.
remote_pwd=

# SVN path section.
[svn_path]
# Svn main directory of newarkstg repo.
svn_ns_dir=
# Svn core platform path.
svn_core_platform_path=
# Svn core platform path config path
svn_core_platform_config_path=
# Svn core platform path config auth path
svn_core_platform_config_auth_path=
# Svn core platform path config api path
svn_core_platform_config_api_path=
# Svn core platform dao path
svn_core_platform_dao_path=
# Svn core platform target path.
svn_core_platform_target_path=

# SVN configuration section. 
[svn]
svn_username=
svn_password=
svn_url=

# Core platform path config section.
[core_path]
# Core platform path.
core_platform_path=
# Core platform config path.
core_platform_config_path=
# Core platform config api path
core_platform_config_api_path=
# Core platform config auth path
core_platform_config_auth_path=
# Core platform bundles path
core_platform_bundles_path=

# Auth platform path config section.
[auth_path]
# Auth platform path.
auth_path=
# Auth platform configuration path.
auth_platform_config_path=
# Auth platform configuration api path.
auth_platform_config_api_path=
# Auth platform configuration auth path.
auth_platform_config_auth_path=
# Authplatform bundles path
auth_platform_bundles_path=

# Memcached configuration section.
[memcached]
# Memcached server ip.
memcached_ip=
# Memcached server port.
memcached_port=

# Other configuration section
[other]
# Core platform version.
core_version=
# Remote log path
remote_log_path=
# Api port
api_port=
# Core platform jar name
core_platform_jar=
# Auth platform jar name
auth_platform_jar=
# Core jar
core_jar=
# Auth jar
auth_jar=
```

接下来，我逐一进行讲解。

配置文件包括以下段：database、remote、svn_path、svn、core_path、auth_path、memcached和other。

每个段的说明如下：

* database 该段定义数据库配置。
	* db_addr MySQL数据库地址。
	* db_usr MySQL数据库用户名。
	* db_pwd MySQL数据库密码。
	* db_port MySQL数据库端口，默认为3306。
* remote 该段定义远程服务器登录信息。
	* remote_ip 部署远程服务器IP。
	* remote_port 部署远程服务器端口。
	* remote_usr 部署远程服务器用户名。
	* remote_pwd 部署远程服务器密码。
* svn_path 该段定义远程服务器SVN目录。
	* svn_ns_dir 项目主SVN目录。
	* svn_core_platform_path 业务平台SVN目录。
	* svn_core_platform_config_path 业务平台主配置文件目录。
	* svn_core_platform_config_auth_path 业务平台AUTH配置文件目录。
	* svn_core_platform_config_api_path 业务平台API配置文件目录。
	* svn_core_platform_dao_path 业务平台DAO目录。
	* svn_core_platform_target_path 业务平台Target目录，用于存放打包后的文件。
* svn 该段定义SVN的账户信息。
	* svn_username SVN用户名。
	* svn_password SVN密码。
	* svn_url SVN地址。
* core_path 该段定义部署后的业务平台目录。
	* core_platform_path 业务平台主目录。
	* core_platform_config_path 业务平台配置文件目录。
	* core_platform_config_api_path 业务平台API配置文件目录。
	* core_platform_config_auth_path 业务平台AUTH配置文件目录。
	* core_platform_bundles_path 业务平台Bundles目录。
* auth_path 该段定义部署后的认证平台目录。
	* auth_path 认证平台主目录。
	* auth_platform_config_path 认证平台配置文件目录。
	* auth_platform_config_api_path 认证平台API配置文件目录。
	* auth_platform_config_auth_path 认证平台AUTH配置文件目录。
	* auth_platform_bundles_path 认证平台Bundles目录。
* memcached 该段定义Memcached相关信息。
	* memcached_ip Memcached服务器IP。
	* memcached_port Memcached服务器端口。
* other 该段定义其他配置信息。
	* core_version 业务平台版本号。
	* remote_log_path 远程服务器日志文件目录，用于存放部署业务平台产生的日志。
	* api_port 业务平台的API端口。
	* core_platform_jar 打包生成的业务平台jar包，完整文件名。
	* auth_platform_jar 认证平台jar包，完整文件名。
	* core_jar 业务平台jar包，不带后缀。
	* auth_jar 认证平台jar包，不带后缀。

**注：以上是所有的配置项，请酌情修改。**

## 6.5 脚本使用 ##

如果您是第一次使用该脚本打包，请依次执行如下命令：

	# 第一步，编辑配置文件；
	vim config.conf
	
	# 第二步，显示帮助；
	./auto_deploy_app_remote.py -h
	
	# 第三步，准备部署（此步可以略过，因为环境已经搭建好）；
	./auto_deploy_app_remote.py -p
	
	# 第四步，检出项目；
	./auto_deploy_app_remote.py -c
	
	# 第五步，部署业务平台；
	./auto_deploy_app_remote.py -d
	
	# 第六步，部署认证平台；
	./auto_deploy_app_remote.py -w
	
	# 第七步，修改数据库配置；
	./auto_deploy_app_remote.py -x
	
	# 第八步，启动认证平台
	./auto_deploy_app_remote.py -k
	
	# 第九布，启动业务平台
	./auto_deploy_app_remote.py -t

**注：第八步可以使用「./auto_deploy_app_remote.py -g」代替，第九步可以使用「./auto_deploy_app_remote.py -r」代替。**

如果您是使用该脚本更新项目，请依次执行如下命令：

	# 第一步，如有需要，编辑配置文件；
	vim config.conf
	
	# 第二步，显示帮助；
	./auto_deploy_app_remote.py -h
	
	# 第三步，更新项目；
	./auto_deploy_app_remote.py -u

	# 第四步，关闭认证平台；
	./auto_deploy_app_remote.py -a
	
	# 第五步，关闭业务平台
	./auto_deploy_app_remote.py -s
	
	# 第六步，部署业务平台；
	./auto_deploy_app_remote.py -d
	
	# 第七步，部署认证平台；
	./auto_deploy_app_remote.py -w
	
	# 第八步，修改数据库配置；
	./auto_deploy_app_remote.py -x
	
	# 第九步，启动认证平台
	./auto_deploy_app_remote.py -k
	
	# 第十布，启动业务平台
	./auto_deploy_app_remote.py -t

**注：第九步可以使用「./auto_deploy_app_remote.py -g」代替，第十步可以使用「./auto_deploy_app_remote.py -r」代替。**

## 7.脚本详解之Windows版本 ##

### 7.1 软件概要 ###

Windows版本目录结构如下：

	tree auto_deploy_app_windows

> auto_deploy_app_windows <br/>
> |-- auto_deploy_app_remote.py <br/>
> |-- auto_deploy_app_v_final.py <br/>
> `-- config.conf <br/>
> `-- logging.conf <br/>
> <br/>
> 0 directories, 4 files

其中，「auto_deploy_app_remote.py」是主执行脚本，用于显示帮助以及调用相应函数。「auto_deploy_app_v_final.py」是核心执行脚本，实现所有的相关功能。「config.conf」是脚本的配置文件。「logging.conf」是日志配置文件。

该脚本实现的功能如下：

* 打印帮助
* 部署准备
* 检出项目
* 更新项目
* 部署业务平台
* 部署认证平台
* 启动、关闭、重启业务平台
* 启动、关闭、重启认证平台
* 修改数据库配置

### 7.2 脚本帮助 ###

参考6.2 节。

### 7.3 脚本概述 ###

参考6.3 节。

### 7.4 配置文件概述 ###

参考6.4 节。

## 7.5 脚本使用 ##

参考6.5 节。

## 8.脚本详解之Tomcat版本 ##

### 8.1 软件概要 ###

Tomcat版本目录结构如下：

	tree auto_deploy_app_to_tomcat/

> auto_deploy_app_to_tomcat/ <br/>
> ├── auto_deploy_app_remote.py <br/>
> ├── auto_deploy_app_v_final.py <br/>
> ├── auto_execute_mall.sh <br/>
> ├── auto_scp_tomcat_log.sh <br/>
> ├── build.xml <br/>
> ├── config.conf <br/>
> ├── crontab <br/>
> ├── remote_restart.sh <br/>
> ├── remote_shutdown.sh <br/>
> └── remote_startup.sh <br/>
> 
> 0 directories, 10 files

该脚本实现的功能如下：

* 打印帮助
* 检出Mall Admin项目
* 检出Mall API项目
* 更新Mall Admin项目
* 更新Mall API项目
* 部署Mall Admin项目
* 部署Mall API项目
* 启动、关闭、重启Mall Admin项目
* 启动、关闭、重启Mall API项目

### 8.2 脚本帮助 ###

	./auto_deploy_app_remote.py -h
 
 ``` bash
 Auto deploy application to the remote web server. Write in Python.
 Version 1.0. By Robin Wen. Email:dbarobinwen@gmail.com
 
 Usage auto_deploy_app.py [-hciumakgstrwd]
   [-h | --help] Prints this help and usage message
   [-c | --svn-co-admin] Checkout the mall admin repo via svn
   [-i | --svn-co-api] Checkout the mall api repo via svn
   [-u | --svn-update-admin] Update the mall admin repo via svn
   [-m | --svn-update-api] Update the mall api repo via svn
   [-a | --shutdown-admin] Shutdown the mall admin via the remote_shutdown.sh scripts
   [-k | --startup-admin] Startup the mall admin  via the remote_startup.shscripts
   [-g | --restart-admin] Restart the mall admin via the remote_restart.sh scripts
   [-s | --shutdown-api] Shutdown the mall api via the remote_shutdown.sh scripts
   [-t | --startup-api] Startup the mall api via the remote_startup.sh scripts
   [-r | --restart-api] Restart the mall api via the remote_restart.sh scripts
   [-w | --deploy-admin] Deploy mall admin via ant
   [-d | --deploy-api] Deploy mall api via ant
 ``` 

 在脚本名后加上「-h 或者 --help」表示打印帮助。
同理，加上「-c | -c | --svn-co-admin」表示检出Mall Admin项目，加上「-i | --svn-co-api」表示检出Mall API项目，加上「-u | --svn-update-admin」表示更新Mall Admin项目，加上「-m | --svn-update-api」表示更新Mall API项目，加上「-a | --shutdown-admin」表示关闭Mall Admin项目，加上「-k | --startup-admin」表示启动Mall Admin项目，加上「-g | --restart-admin」表示重启Mall Admin项目，加上「-s | --shutdown-api」表示关闭Mall API项目，加上「-t | --startup-api」表示启动Mall API项目，加上「-r | --restart-api」表示重启Mall API项目，加上「-w | --deploy-admin」表示部署Mall Admin项目，加上「-d | --deploy-api]」表示部署Mall API项目。

### 8.3 脚本概述 ###

如前所述，「auto_deploy_app_remote.py」是主执行脚本，用于显示帮助以及调用相应函数。「auto_deploy_app_v_final.py」是核心执行脚本，实现所有的相关功能。核心执行脚本采用Fabric实现远程执行命令，主执行脚本再通过**fab -f 脚本名 任务名**调用相应方法。

主执行脚本和核心执行脚本的方法名基本一致，主执行脚本包括如下方法：main(argv)、usage()、svn_co_admin()、svn_co_api()、svn_update_admin()、svn_update_api()、shutdown_admin()、startup_admin()、restart_admin()、shutdown_api()、startup_api()、restart_api()、deploy_admin()和deploy_api()。

核心执行脚本包括如下方法：main(argv)、usage()、svn_co_admin()、svn_co_api()、svn_update_admin()、svn_update_api()、shutdown_admin()、startup_admin()、restart_admin()、shutdown_api()、startup_api()、restart_api()、deploy_admin()、deploy_api()和getConfig()。

**主执行脚本：**

* main(argv) 主函数
* usage() 使用说明函数
* svn_co_admin 检出Mall Admin项目函数
* svn_co_api 检出Mall API项目函数
* svn_update_admin 更新Mall Admin项目函数
* svn_update_api() 更新Mall API项目函数
* shutdown_admin  关闭Mall Admin项目函数
* startup_admin 启动Mall Admin项目函数
* restart_admin 重启Mall Admin项目函数
* shutdown_api 关闭Mall API项目函数
* startup_api 启动Mall API 项目函数
* restart_api 重启Mall API 项目函数
* deploy_admin 部署Mall Admin项目函数
* deploy_api 部署Mall API 项目函数

**主执行脚本**

主执行脚本内容如下：
参考脚本auto_deploy_app_remote.py。

**核心执行脚本**

方法和主执行脚本基本一致，相同的不赘述。核心执行脚本还提供getConfig()方法，用于读取配置文件。

核心执行脚本内容如下：
参考脚本auto_deploy_app_v_final.py。

`auto_execute_mall.sh`脚本实现了自动从SVN检出项目，自动部署到Tomcat。

参考auto_execute_mall.sh脚本。

`auto_scp_tomcat_log.sh`脚本实现了从Tomcat服务器自动拉取日志。为了更好的查看日志，拉取了近三天（昨天、今天和明天）的日志。

参考auto_scp_tomcat_log.sh脚本。

### 8.4 配置文件概述 ###

完整配置文件内容如下：

```bash
# Remote server section.
[remote]
# Remote server ip.
remote_ip=
# Remote server port.
remote_port=
# Remote server username.
remote_usr=
# Remote server password.
remote_pwd=

# SVN path section.
[svn_path]
# Svn main directory of repo.
svn_sw_dir=
# Svn mall admin path.
svn_admin_path=
# Svn mall api path.
svn_api_path=
# Mall admin path.
admin_path=
# Mall api path.
api_path=

# Mall admin svn configuration section. 
[svn_admin]
# Mall admin svn url.
svn_admin_url=
# Mall admin svn username.
svn_admin_username=
# Mall admin svn password.
svn_admin_password=

# Mall api svn configuration section. 
[svn_api]
# Mall api svn url.
svn_api_url=
# Mall api svn username.
svn_api_username=
# Mall api svn password.
svn_api_password=

# Tomcat section
[tomcat]
# Tomcat webapps path.
tomcat_path=
# Tomcat bin path.
tomcat_bin_path=

# Other configuration section
[other]
# Remote log path
remote_log_path=
```

接下来，我逐一进行讲解。

配置文件包括以下段：remote、svn_path、svn_admin、svn_api、tomcat和other。

每个段的说明如下：

* remote 该段定义远程服务器登录信息。
	* remote_ip 部署远程服务器IP。
	* remote_port 部署远程服务器端口。
	* remote_usr 部署远程服务器用户名。
	* remote_pwd 部署远程服务器密码。
*  svn_path 该段定义SVN的相关路径。
	* svn_sw_dir 该段定义SVN的主目录。
	* svn_admin_path 该段定义Mall Admin SVN的主路径。
	* svn_api_path 该段定义Mall API SVN的主路径。
	* admin_path 该段定义Mall Admin SVN的具体路径。
	* api_path  该段定义Mall API SVN的具体路径。
*  svn_admin 该段定义Mall Admin的SVN相关信息。
	*  svn_admin_url Mall Admin SVN的URL。
	*  svn_admin_username Mall Admin SVN的URL。
	*  svn_admin_password Mall Admin SVN的密码。
*  svn_api 该段定义Mall API的SVN相关信息。
	*  svn_api_url Mall API SVN的URL。
	*  svn_api_username Mall API SVN的URL。
	*  svn_api_password Mall API SVN的密码。
* tomcat 该段定义Tomcat相关信息。
	* tomcat_path Tomcat的webapps路径。
	* tomcat_bin_path Tomcat的启动脚本路径。
* other 该段定义其他配置信息。
	* remote_log_path 远程服务器Log路径。

如有需要，请酌情修改。

## 8.5 脚本使用 ##

**Step 1：** 运行脚本之前，需要把ant使用的build.xml分别放到~/build-xml/admin和~/build-xml/api中，方便程序读取。

**注意：需要把YOUR_PROJECT改成您的项目名。**

**Step 2：** 把Repo中以`remote_`开头的脚本放到Tomcat的bin目录。

**Step 3：** 把auto_deploy_app_v_final.py中的YOUR_PROJCET替换成您的项目名。命令如下：

	sed -i 's/YOUR_PROJECT/XXX/g' auto_deploy_app_v_final.py

**Step 4：** 把以auto_开头的四个脚本以及config.conf配置文件放到远程服务器，脚本中的路径（YOUR_PATH）、Tomcat目录（YOUR_TOMCAT_HOME）、用户名（YOUR_NAME）、密码（YOUR_IP）请酌情修改，然后使用crontab中的任务自动执行。

	crontab -e

crontab 任务如下。

	crontab -l

> 00 00 * * * bash YOUR_PATH/auto_execute_mall.sh <br/>
> 0 01 * * * bash YOUR_PATH/auto_scp_tomcat_log.sh <br/>
> */1 * * * * ntpdate pool.ntp.org

该任务定义了凌晨0点部署项目，以及凌晨1点拷贝日志。

**Step 5：** 早晨上班就可以看到昨晚的部署日志了，如果有问题，把日志给开发人员，再做调试。So easy, 妈妈再也不用担心我加班了！:)

Enjoy！

## 9.GitHub地址 ##

python-auto-deploy：https://github.com/dbarobin/python-auto-deploy

## 10.项目说明 ##

auto_deploy_app_v2: 适用于Linux。<br/>
auto_deploy_app_windows: 适用于Windows。<br/>
auto_deploy_app_to_tomcat: 适用于Linux下部署Tomcat。

## 11.作者信息 ##

温国兵

* Robin Wen
* <a href="mailto:dbarobinwen@gmail.com"><img src="http://i.imgur.com/7yOaC7C.png" title="Robin's Gmail" border="0" height="16px" width="16px" alt="Robin's Gmail" /></a>
* <a href="https://github.com/dbarobin" target="_blank"><img src="https://farm9.staticflickr.com/8653/15815300288_69bb6bb56d_o_d.png" title="Github" border="0" alt="Github" height="16px" width="16px" /></a>
* <a href="https://dbarobin.github.io/" target="_blank"><img src="http://i.imgur.com/dEfMkyt.jpg" title="Robin's Blog" border="0" alt="Robin's Blog" height="16px" width="16px" /></a>
* <a href="http://blog.csdn.net/justdb" target="_blank"><img src="http://i.imgur.com/BROigUO.jpg" title="DBA@Robin's CSDN" height="16px" width="16px" border="0" alt="DBA@Robin's CSDN" /></a>