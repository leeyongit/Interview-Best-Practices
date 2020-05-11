# php-fpm的运行原理
php-fpm是一种master（主）/worker（子）多进程架构。master进程主要负责CGI及PHP环境初始化、事件监听、子进程状态等等，worker进程负责处理php请求。
从FPM接收到请求，到处理完毕，其具体的流程如下：

1. FPM的master进程接收到请求。
2. master进程根据配置指派特定的worker进程进行请求处理，如果没有可用进程，返回错误，这也是我们配合Nginx遇到502错误比较多的原因。
3. worker进程处理请求，如果超时，返回504错误。
4. 请求处理结束，返回结果。


