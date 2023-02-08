---
title: Fiddler Https抓包及安装Fiddler证书
url: https://www.yuque.com/endday/blog/mcdgxm
---

0 ) 删除以下东西
a.清除C:\Users\Administrator\AppData\Roaming\Microsoft\Crypto\RSA 目录下所有文件（首次安装fiddler请忽略）
b.清除电脑上的根证书，WIN+R快捷键，输入：certmgr.msc， 然后回车，查找所有fiddler证书，然后删除。（首次安装fiddler请忽略）
![](..\assets\mcdgxm\1631783602487-19e6cc27-7458-48b0-acb3-69c77975f894.jpeg)
c.清除浏览器上的证书文件，此处需要仔细检查查找带有FiddlerRoot的字样，并删除，以谷歌浏览器为例说明，在浏览器上输入chrome://settings/&#x20;
![](..\assets\mcdgxm\1631783603000-c552a016-6afe-4a78-b44e-e46ca1894ec8.jpeg)
d.打开fiddler，点击工具栏中的Tools—>Options，点击Actions,选择最后一项，Reset All certificates,然后关闭
![](..\assets\mcdgxm\1631783603477-42f21a27-d44b-4ed3-9036-28322ae7066f.png)
e.卸载老版本，安装新版本fiddler v5.0
1 ) 使用AutoResponder代替willow，配置如下：
&#x20;  ![](..\assets\mcdgxm\1631783603987-1f7fcc33-be1d-411c-85a2-446cf1adc821.png)
2）打开Fiddler，依次选择Tools->Fiddler Options->HTTPS，勾选“Capture HTTPS CONNECTs”和“Decrypt HTTPS traffic”。 点击“OK”以后Fiddler会弹出一个对话框问你是否要让Windows信任Fiddler生成的自签证书，选择“Yes”以后，还会弹出一些对话框，直接“Yes”或“Ok”即可。
此时就可以满足PC的抓包了
![](..\assets\mcdgxm\1631783604297-55dc9abb-caeb-4395-8d81-0ec7ae35b9ac.png)

3）( delete 手机抓包不需要此步骤了)在PC上安装一个Fiddler插件CertMaker for iOS and Android，下载：\[http://www.telerik.com/fiddler/add-ons]\(http://www.telerik.com/fiddler/add-ons" \t "http://km.oa.com/group/19674/articles/show/\_blank)
![](..\assets\mcdgxm\1631783604556-f4ec3a76-9c3e-4fa5-b493-f92650d96776.png)
4）终端设备连上PC的代理，用浏览器访问 <http://ipv4.fiddler:8888/> ，在最下方点击FiddlerRoot certificate 安装新的证书
![](..\assets\mcdgxm\1631783605033-3ad05ac8-8310-46ea-94eb-fa7d8085c6a9.png)
安装完后看下我们的新证书：
![](..\assets\mcdgxm\1631783605820-84e83623-5b2a-4659-9e55-4d2af6887e2e.png)
