xxnet
=============
1. set systemc proxy use automatic pac :http://127.0.0.1:8086/proxy.pac
2. 打开xx-net http://127.0.0.1:8085
	configuration中的 gaeappid中输入：irunshow2012|icesxrungoagent2|icesxrun-xx-net|icesxrun-xx-net2
3. 登录google，让chorme同步插件后，配置switchyomega
4. 关闭系统的automatic pac
5. 开启ipv6
	$sudo apt-get install miredo
	$sudo service miredo restart
	$修改路由器上的dns，否则miredo有时无法建立，经测试114.114.114.114暂时可以
	