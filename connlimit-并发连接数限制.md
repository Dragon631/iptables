# 并发连接数

> -m | --match connlimit
>
> - 限制每个IP地址或地址块与服务器的并发连接数
>
> - 语法
>
>   - --connlimit-above n
>     - 如果现有连接数高于n，则匹配
>   - --connlimit-upto n
>     - 如果现有连接低于或等于n，则匹配
>   - --connlimit-mask prefix_length
>     - 使用网络前缀长度对主机进行分组
>   - --connlimit-saddr --connlimit-daddr
>     - 将限制应用于源地址或目的地址，默认为源地址
>
> - 示例：
>
>   ```SH
>   $ iptables -I INPUT -p tcp --dport 22 -m connlimit \
>   --connlimit-above 2 --connlimit-mask 24 -j REJECT
>   ```
>
>   

