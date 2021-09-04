xxnet
=============
### 安装

1. set systemc proxy use automatic pac :http://127.0.0.1:8086/proxy.pac
2. 打开xx-net http://127.0.0.1:8085
    configuration中的 gaeappid中输入：irunshow2012|icesxrungoagent2|icesxrun-xx-net|icesxrun-xx-net2
3. 登录google，让chorme同步插件后，配置switchyomega
4. 关闭系统的automatic pac
5. 开启ipv6
```sh
sudo apt-get install miredo
sudo service miredo restart
#修改路由器上的dns，否则miredo有时无法建立，经测试114.114.114.114暂时可以
```

### 修改端口和ip

1. web ip 端口

   ```sh
   vi XX-Net/data/launcher/config.yaml
   ```

   代理端口

   ```sh
   web_control.py:            "proxy_listen": str(config.listen_ip) + ":" + str(config.listen_port),
   ```

   

   ```sh
   icesx@ztf:~/software/XX-Net/data/gae_proxy$ cat config.json 
   {
     "GAE_APPIDS": [
       "irunshow2012", 
       "icesxrungoagent2", 
       "icesxrun-xx-net", 
       "icesxrun-xx-net2"
     ],
    "listen_ip":"0.0.0.0",
    "listen_port":21003 
   }
   ```

   ## 远程代理

   1. 远程服务器上安装xxnet

   2. 修改 8085 和ip为必要的端口与ip

      ```
      vi XX-Net/data/launcher/config.yaml
      ```

   3. 修改8087 和ip为必要的端口与ip以及appid
   
      ```sh
      vi /XX-Net/data/gae_proxy/config.json 
      {
        "GAE_APPIDS": [
          "irunshow2012", 
          "icesxrungoagent2", 
          "icesxrun-xx-net", 
          "icesxrun-xx-net2"
        ],
       "listen_ip":"0.0.0.0",
       "listen_port":21003 
      }
      ```
   
   4. 增加CA
   
   5. deploy
   
      ```sh
      vi ./code/default/gae_proxy/local/download_gae_lib.py
      
          client = simple_http_client.Client(proxy={
              "type": "http",
              "host": "0.0.0.0",
              "port": 21001,
              "user": None,
              "pass": None
          }, timeout=timeout, cert=cert)
      ```
   

### 修改login server



```
 cat /TOOLS/software/XX-Net/data/x_tunnel/client.json 
{
  "login_account": "icesxrun@gmail.com",
  "login_password": "",
  "socks_host": "0.0.0.0",
  "server_host": "miami01.xx-net.org"

```

  "server_host": "miami01.xx-net.org"

"server_host":"v13.xx-net.org"
