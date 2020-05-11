## nginx和php-fpm通信的两种方式 unix socket和TCP

nginx和fastcgi的通信方式有两种，一种是TCP 一种是unix socket

- TCP使用的是 127.0.0.1:9000端口，将fastcgi_pass参数修改为127.0.0.1:9000
- unix socket 使用套接字 /dev/shm/php-cgi.sock，两个进程引用同一个socket描述符文件就可以建立通道进行通信了，fastcgi_pass unix:/dev/shm/fpm-cgi.sock;

### 创建sock文件
```sh
sudo touch /dev/shm/fpm-cgi.sock
sudo chown www-data:www-data /dev/shm/fpm-cgi.sock
sudo chmod 666 /dev/shm/fpm-cgi.sock
```
    原理上来说，unix socket方式肯定要比tcp的方式快而且消耗资源少，因为socket之间在nginx和php-fpm的进程之间通信，而tcp需要经过本地回环驱动，还要申请临时端口和tcp相关资源，unix socket会显得不是那么稳定，当并发连接数爆发时，会产生大量的长时缓存，在没有面向连接协议支撑的情况下，大数据包很有可能就直接出错并不会返回异常。而TCP这样的面向连接的协议，多少可以保证通信的正确性和完整性。