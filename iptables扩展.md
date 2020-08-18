> 扩展：`ls /usr/lib64/xtables`
>
> 帮助：`man iptables-extensions`

![image-20200818210424893](image-20200818210424893.png)

![image-20200818211248825](image-20200818211248825.png)

![image-20200818211201242](image-20200818211201242.png)

> 匹配扩展

- -m | --match 扩展名称

- 可同时指定多个匹配扩展模块

- 使用 -h | --help 来获得该扩展的帮助信息

- 如果通过 -p 或 --protocol 指定协议（tcp、udp、icmp），当遇到有未知选项时，会自动加载与协议同名的匹配模块

- 常用 匹配扩展：

  ```sh
  multiport
  mac
  iprange
  state
  string
  connlimit
  time
  limit
  GeoIP # 自定义加载
  ```

- 多端口： `-m | --match multiport`

  > 必须紧跟在协议后面，最多可指定15个端口或范围
  >
  > 只能与以下协议之一使用：tcp、udp、udplite、dccp和sctp
  >
  > 注意：不要与以下通用匹配混淆
  >
  > ```sh
  > [!] --sport | --source-port port[:port]
  > [!] --dport | --destination-port port[:port]
  > ```
  >
  > 语法：
  >
  > ```sh
  > [!] --source-ports | --sports port[,port|,port:port] ...
  > # 如果源端口是给定端口之一，则匹配
  > 
  > [!] --destination-ports | --dports port[,port|,port:port] ...
  > # 如果目标端口是给定端口之一，则匹配
  > 
  > [!] --ports port[,port|,port:port] ...
  > 
  > # 示例
  > iptables -A INPUT -p tcp -m multiport --portS 22,80,1024:65535 -j ACCEPT
  > ```

- IP范围：`-m | --match iprange`

  > 匹配给定的任意IP地址范围
  >
  > 注意：不要与以下通用匹配混淆
  >
  > ```sh
  > [!] -s | --source address[/mask][,...]
  > [!] -d | --destination address[/mask][,...]
  > ```
  >
  > 语法：
  >
  > ```sh
  > [!] --src-range from[-to]
  > # 匹配指定范围内的源IP
  > 
  > [!] --dst-range from[-to]
  > # 匹配指定范围内的目标IP
  > 
  > # 示例：
  > iptables -I INPUT -m iprange --src-range 192.168.1.10-192.168.1.20 -j DROP
  > ```

- 字符串：`-m | --match string`

  > 匹配指定的字符串，要求Linux内核 >= 2.6.14
  >
  > 语法：
  >
  > ```sh
  > --algo {bm|kmp}
  > # 指定模式匹配策略(bm = Boyer-Moore, kmp = Knuth-Pratt-Morris)
  > # 示例：
  > iptables -A INPUT -p tcp --dport 80 \
  > -m string --algo bm --string 'GET /index.html' -j LOG
  > 
  > --from offset
  > # 设置开始偏移量，默认为0
  > --to offset
  > # 设置应扫描的偏移量，默认为数据包大小
  > # 字节偏移-1（从0开始计数）是最后扫描的字节
  > [!] --string pattern
  > # 匹配给定的模式
  > [!] --hex-string pattern
  > # 以十六进制表示法匹配给定的模式
  > ```

- 时间限制：`-m | --match time`

  > 匹配数据包到达时间/日期在是否在给定范围内
  >
  > 默认情况下，所有时间都被解释为UTC
  >
  > 语法：
  >
  > ```sh
  > --datestart yyyy[-MM[-DD[Thh[:mm[:ss]]]]]
  > --datestop yyyy[-MM[-DD[Thh[:mm[:ss]]]]]
  > --timestart hh:mm[:ss]
  > --timestop hh:mm[:ss]
  > [!] --monthdays day[,day...] # 可以使用数字1-7或Mon,Tue,Wed,Thu,Fri,Sat,Sun
  > [!] --weekdays day[,day...]  # 可以使用数字1-7或Mon,Tue,Wed,Thu,Fri,Sat,Sun
  > --kerneltz    # 使用内核时区而不是UTC  
  > --contiguous  # 指定跨日期时连续的时间，例如：星期一23:00到01:00
  > ```
  >
  > 
  >
  > 示例：
  >
  > ```sh
  > -m time --weekdays sa,su
  > # 匹配周末，或者写成--weekdays 6,7
  > -m time --datestart 2020-10-01 --datestop 2020-10-07
  > # 匹配国庆7天
  > -m time --timestart 12:00 --timestop 14:00 --kernelts
  > # 中午时间
  > -m time --weekdays Fr --monthdays 22,23,24,25,26,27,28
  > # 本月的第四个星期五
  > -m time --daystart 2008-01-01T17:00 --datestop 2008-01-01T23:59:59
  > # 包含开头与结束时间在内。注意，停止时间与新一天的第一秒是不匹配
  > -m time --weekdays Mo --timestart 23:00 --timestop 01:00
  > # 星期一从凌晨0点到凌晨1点一小时，然后从晚上23:00在开始再一小时，这是不连续的两个小时
  > -m time --weekdays Mo --timestart 23:00 --timestop 01:00 --contiguous
  > # 从星期一晚上23:00 开始的连个小时，这个是连续的两个小时，会跨到星期二
  > ```
  >
  > ```sh
  > # 从早上8:00点到晚上18:00禁止上网冲浪
  > iptables -A OUTPUT -p tcp --dport 80 \
  > -m --timestart 8:00 --timestop 18:00 -j REJECT
  > ```
  >
  > 

- mac地址：`-m | --match mac`

  > 匹配二层的以太网地址源地址
  >
  > 地址格式：xx:xx:xx:xx:xx:xx
  >
  > 语法：
  >
  > ```sh
  > [!] --mac-source address
  > 
  > # 示例：
  > iptables -A INPUT -i <local interface> -p tcp \
  > -m mac --mac-source xx:xx:xx:xx:xx:xx -j DROP
  > ```

目标扩展









