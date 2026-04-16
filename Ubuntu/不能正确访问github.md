问题：
curl:Failed to connect to raw.githubusercontent.com port 443 after 0ms:拒绝连接

解决办法：
1、打开网址https://www.ipaddress.com
2、搜索拒绝连接的网址（raw.githubusercontent.com）
3、找到ipv4地址
4、修改/etc/hosts文件
sudo vim /etc/hosts
5、添加
185.199.108.133  raw.githubusercontent.com