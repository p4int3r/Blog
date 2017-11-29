# w3af工具
---

* Web Application Attack and Audit Framework（Web应用程序攻击和审计框架）

* 安装：
  第一步：更新软件安装包，`apt-get update`；
  第二步：安装并更新pip，`apt-get install -y python-pip`，`pip install --upgrade pip`；
  （可选）：运行pip可能出现SSL_ST_INIT错误，重新卸载安装pip，`python -m easy_install -U pip`；
  第三步：从GitHub下载w3af，`git clone https://github.com/andresriancho/w3af.git`；
  第四步：安装pybloomfiltermmap，`apt-get install -y pybloomfiltermmap`；
  第五步：cd到w3af目录，运行`./w3af_gui`，显示在/tmp目录下生成安装依赖的脚本；
  第六步：cd到/tmp目录，执行`./w3af_dependency_install.sh`；
  （可选）：如果执行完第六步还报错，运行命令`apt-get build-dep python-lxml`；
  第七步：修改文件w3af/w3af/core/controllers/dependency_check/requirements.py；
      PIPDependency(‘pybloomfilter’, ‘pybloomfiltermmap’, ‘0.3.15’),
      PIPDependency(‘OpenSSL’, ‘pyOpenSSL’, ‘16.2.0’),
      PIPDependency(‘lxml’, ‘lxml’, ‘3.7.1’),
  第八步：修改配置文件w3af/w3af/core/controllers/dependency_check/platforms/mac.py；
      MAC_CORE_PIP_PACKAGES.remove(PIPDependency(‘pybloomfilter’, ‘pybloomfiltermmap’, ‘0.3.15’)
  第九步：安装graphviz，`apt-get install graphviz`；
  第十步：下载webkit依赖并安装；
      wget http://ftp.br.debian.org/debian/pool/main/p/pywebkitgtk/python-webkit_1.1.8-3_amd64.deb
      wget http://ftp.br.debian.org/debian/pool/main/w/webkitgtk/libjavascriptcoregtk-1.0-0_2.4.11-3_amd64.deb
      wget http://ftp.br.debian.org/debian/pool/main/p/python-support/python-support_1.0.15_all.deb
      wget http://ftp.br.debian.org/debian/pool/main/w/webkitgtk/libwebkitgtk-1.0-0_2.4.11-3_amd64.deb

      dpkg -i libjavascriptcoregtk-1.0-0_2.4.11-3_amd64.deb
      dpkg -i python-support_1.0.15_all.deb
      dpkg -i libwebkitgtk-1.0-0_2.4.11-3_amd64.deb
      dpkg -i python-webkit_1.1.8-3_amd64.deb
  （可选）：在安装python-webkit_1.1.8-3_amd64.deb时会提示出错，运行`apt --fix-broken install`，再重新安装；
  第十一步：安装gtksourceview2，运行`apt-get install gtksourceview2`；
  第十二步：运行`./w3af_gui`或者`./w3af_console`，成功启动w3af。

* 创建快捷方式，在`/usr/share/applications`目录下，创建kali-w3af.desktop文件：
      [Desktop Entry]
      Name=w3af
      Encoding=UTF-8
      Exec=sh -c "/root/Tools/w3af/w3af_gui"
      Icon=/root/Tools/w3af/w3af.png
      StartupNotify=false
      Terminal=false
      Type=Application
      Categories=03-webapp-analysis;
      X-Kali-Package=w3af
* w3af_console的使用：
  执行w3af_console进入命令行模式；
  （1）显示当前命令提示符下的帮助信息；
      help
  （2）进入plugins模块；
      w3af>>> plugins
  （3）查看插件；
      w3af/plugins>>> list [插件类型]
  （4）选择小插件；
      w3af/plugins>>> [插件类型] [插件名称]
  （5）选择所有插件；
      w3af/plugins>>> [插件类型] all
  （6）回到上一级；
      back
  （7）进入配置文件；
      w3af>>> profiles
  （8）在选择好插件之后，可以保存自定义的profile；
      w3af/profiles>>> save_as [自定义profile名称]
  （9）使用某个profile；
      w3af/profiles>>> use [profile名]
  （10）全局配置http-settings；
      w3af>>> http-settings
  （11）查看http-settings配置选项；
      w3af/config:http-settings>>> view
  （12）设置http-settings的配置选项；
      w3af/config:http-settings>>> set [选项名] [选项值]
  （13）保存http-settings的配置选项；
      w3af/config:http-settings>>> save
  （14）全局配置misc-settings与http-settings同理；
  （15）设置扫描目标；
      w3af>>> target
      w3af/config:target>>> set target [url]
  （16）开始扫描；
      w3af>>> start 

* 脚本文件目录：w3af/scripts，可以使用在w3af_console命令行下的命令集结成相似的脚本文件；
      w3af_console -s [脚本文件]    

* w3af_console示例：
  （1）查看插件；
      w3af/plugins>>> list audit
  （2）选择小插件，执行命令后，相应插件的status列变为Enable；
      w3af/plugins>>> audit xss sqli
  （3）使用profile；
      w3af/profiles>>> use fast_scan
  （4）设置http-settings的配置选项；
      w3af/config:http-settings>>> set rand_user_agent true
  

* w3af_gui的使用：
  （1）对应于w3af_console的http-settings；
      【Configuration】->【HTTP Config】
  （2）编码解码工具；
      【Tools】->【Encode/Decode】
  （3）扫描时配置身份验证信息；
      【Configuration】->【HTTPConfig】->【Basic HTTP Authentication】/【NTML Authentication】
  （4）基于表单的身份验证；
      【Active Plugin】->【auth】
  （5）基于Cookie的身份验证；
      第一步：先将cookie信息保存成文件cookie.txt（可以使用Firebug导出，字段使用tab键分隔）；
      # Netscape HTTP Cookie File
      .domain.com    TRUE   /       FALSE   1731510001      user    admin
      第二步：【Configuration】->【HTTP Config】->【Cookies】； 
      第三步：导入cookie文件进行保存。
  （6）基于HTTP头的身份验证；
      第一步：复制HTTP请求头中相对固定的字段（含有cookie）保存成文本文件；
      第二步：在【Configuration】->【HTTP Config】->【General】中配置步骤一中的文本文件；
      第三步：进行扫描。
  （7）截断代理功能；
      进行代理配置：【Tools】->【Proxy】->【Options】
      或者
      在【Active Plugin】->【crawl】->【spider_man】
  （8）可以进行手工修改重放、模糊测试和响应比较；
      【Tools】->【Proxy】->【History】
  （9）exploit功能；
      【exploit】
