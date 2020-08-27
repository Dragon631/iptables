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

- TCP 头部匹配选项

  > [!] --tcp-flags mask comp
  >
  > mask: 应该检查的标志，SYN ACK FIN RST URG PSH ALL NONE（逗号分隔）
  >
  > Comp: 必须设置的以逗号分隔的标志列表

  > [!] --syn
  >
  > 仅匹配设置了SYN位的TCP数据包

  > 示例：
  >
  > ```sh
  > iptables -I INPUT -p tcp -m tcp --dport 22 \
  > --tcp-flags SYN,ACK,FIN,RST,URG,PSH SYN -j ACCEPT
  > iptables -I OUTPUT -p tcp -m tcp --sport 22 \
  > --tcp-flags SYN,ACK,FIN,RST,URG,PSH SYN,ACK -j ACCEPT
  > iptables -I INPUT -p tcp -m tcp --dport 22 \
  > --sync -j ACCEPT
  > ```

- TCP/IP包

  ![image-20200827202908694](E:\_添加技能包\iptables\image-20200827202908694.png)

> **URG：**紧急指针标志
>
> - 1：紧急指针有效
> - 0：忽略紧急指针
>
> **ACK：**确认序号标志
>
> - 1：确认号有效
> - 0：忽略确认号段
>
> **PSH：**push标志
>
> - 1：带有push标志的数据，表示接收方在接收到该报文后应尽快将这个报文段交给应用程序，而不是缓冲区排队
>
> **RST：**重置连接标志
>
> - 用于重置由于主机崩溃或其他原因而出现错误的连接
> - 或者用于拒绝非法的报文段和拒绝连接请求
>
> **SYN：**同步序号
>
> - 用于建立连接过程，在连接请求中，SYN=1和ACK=0表示该数据段没有使用捎带的确认域
> - 而连接应答捎带一个确认，即SYN=1和ACK=1。
>
> **FIN：**结束标志
>
> - 用于释放连接，为1时表示发送方已经没有数据发送了，即关闭本方数据流

- TCP三次握手四次挥手

  ![https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=2590032753,2466318043&fm=173&app=49&f=JPEG?w=640&h=716&s=E7F239D247AFCCEA106594580300D072](https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=2590032753,2466318043&fm=173&app=49&f=JPEG?w=640&h=716&s=E7F239D247AFCCEA106594580300D072)

> 三次握手
>
> ![image-20200827201913216](E:\_添加技能包\iptables\image-20200827201913216.png)

> **第一次握手**
>
> 客户端发送一个TCP的SYN标志位置1的包指明客户打算连接的服务器的端口
>
> 初始序号X,保存在包头的序列号(Sequence Number)字段里
>
> ![https://www.centos.bz/wp-content/uploads/2012/08/100327002911.png](https://www.centos.bz/wp-content/uploads/2012/08/100327002911.png)
>
> ```sh
> # 放行第一次握手，即设置SYN位，并放行
> iptables -I INPUT -p tcp -m tcp --dport 22 \
> --tcp-flags SYN,ACK,FIN,RST,URG,PSH SYN -j ACCEPT
> 
> # 简写
> iptables -I INPUT -p tcp -m tcp --dport 22 --sync -j ACCEPT # 表示只有SYN位设值
> ```

> **第二次握手**
>
> 服务器发回确认包(ACK)应答
>
> 即SYN标志位和ACK标志位均为1
>
> 同时，将确认序号(Acknowledgement Number)设置为客户的I S N加1以，即X+1
>
> ![100327003054](http://www.centos.bz/wp-content/uploads/2012/08/100327003054.png)
>
> ```sh
> # 放行第一次握手，即设置SYN,ACK位，并放行
> iptables -I OUTPUT -p tcp -m tcp --sport 22 \
> --tcp-flags SYN,ACK,FIN,RST,URG,PSH SYN,ACK -j ACCEPT
> ```

- 状态`-m | --match state`

  > state 扩展可以访问数据包的连接跟踪状态
  >
  > 是conntrack 模块的子集
  >
  > 语法：
  >
  > ```yaml
  > [! ] --state state
  > state: 
  > NEW: 新创建连接的数据包
  > ESTABLISHED: 与在两个方向上看到分组的连接相关联的数据包
  > RELATED: 新连接的数据包，而且与现有连接相关联。如：FTP数据传输或ICMP错误消息
  > INVALID: 没有与任何已知连接相关联的数据包，建议DROP
  > UNTRACKED: 未跟踪的数据包
  > ```
  >
  > iptables的state machine
  >
  > - state machine （状态机）是iptables中的一个特殊部分
  > - 内核中connection tracking machine框架来实现对连接的跟踪
  > - 通过TCP，UDP或ICMP协议等唯一性信息来跟踪数据流
  >   - NEW、ESTABLISHED、RELATED、INVALID、UNTRACKED
  > - 跟踪文件：cat /proc/net/nf_conntrack
  > - 加载的模块：lsmod | grep track
  >
  > ```sh
  > # 实验：
  > iptables -L
  > ll /proc/net/nf_conntrack
  > iptables -I OUTPUT -m state --state NEW -j ACCEPT  # NEW匹配第一个包
  > ll /proc/net/nf_conntrack
  > cat /proc/net/nf_conntrack
  > lsmod |grep track
  > ```
  >
  > 状态防火墙的优点：
  >
  > - 比飞状态防火墙更安全
  > - 允许编写更加严谨的规则策略，能
  >
  > 状态跟踪机制的缺点：
  >
  > - 需要维护一个状态表，需要比静态防火墙更多的内存空间
  > - 软件实现对状态文件内容的创建、查询、删除需要花费额外的时间，对CPU产生一定的压力
  >
  > ![image-20200827215049252](E:\_添加技能包\iptables\image-20200827215049252.png)
  >
  > ![image-20200827215024613](E:\_添加技能包\iptables\image-20200827215024613.png)
  >
  > ![image-20200827215210449](E:\_添加技能包\iptables\image-20200827215210449.png)
  >
  > ![image-20200827215224821](E:\_添加技能包\iptables\image-20200827215224821.png)
  >
  > ![image-20200827215308462](E:\_添加技能包\iptables\image-20200827215308462.png)
  >
  > ![image-20200827215334822](E:\_添加技能包\iptables\image-20200827215334822.png)

  > 单服务器防护
  >
  > ```sh
  > # 此配置不能访问外部资源，如DNS解析、ping外部主机等
  > iptables -A INPUT -p tcp -m multiport --dports 22,80 -j ACCEPT
  > iptables -A OUTPUT -p tcp -m multiport --sports 22,80 -j ACCEPT
  > 
  > iptables -A INPUT -p tcp --dport 3306 -m iprange --src-range \
  > 192.168.1.201-192.168.1.209 -j ACCEPT
  > iptables -A OUTPUT -p tcp --sport 3306 -m iprange --dst-range \
  > 192.168.1.201-192.168.1.209 -j ACCEPT
  > 
  > iptables -A INPUT -i lo -j ACCEPT
  > iptables -A OUTPUT -o lo -j ACCEPT
  > 
  > iptables -P INPUT DROP
  > iptables -P OUTPUT DROP
  > ```
  >
  > 单服务器防护（优化１）
  >
  > ```sh
  > #　此配置可以主动访问任意外部资源，但防止外部资源非相关包的入站
  > iptables -A INPUT -p tcp -m multiport --dports 22,80 -j ACCEPT
  > iptables -A OUTPUT -p tcp -m multiport --sports 22,80 -j ACCEPT
  > iptables -A INPUT -p tcp --dport 3306 -m iprange --src-range \
  > 192.168.1.201-192.168.1.209 -j ACCEPT
  > iptables -A OUTPUT -p tcp --sport 3306 -m iprange --dst-range \
  > 192.168.1.201-192.168.1.209 -j ACCEPT
  > iptables -A INPUT -i lo -j ACCEPT
  > iptables -A OUTPUT -o lo -j ACCEPT
  > iptables -P INPUT DROP
  > iptables -P OUTPUT DROP
  > 
  > iptables -A OUTPUT -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
  > iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
  > # 外部发起的连接NEW和INVALID将被默认规则DROP掉
  > ```
  >
  > 单服务器防护（优化２）
  >
  > ```sh
  > # 此配置可以主动访问任意外部资源
  > # 并限定源端口范围1024以上非特权端口
  > # 允许ping外部主机，并防止外部资源非相关包的入站
  > iptables -A INPUT -p tcp -m multiport --dports 22,80 -j ACCEPT
  > iptables -A OUTPUT -p tcp -m multiport --sports 22,80 -j ACCEPT
  > iptables -A INPUT -p tcp --dport 3306 -m iprange --src-range \
  > 192.168.1.201-192.168.1.209 -j ACCEPT
  > iptables -A OUTPUT -p tcp --sport 3306 -m iprange --dst-range \
  > 192.168.1.201-192.168.1.209 -j ACCEPT
  > iptables -A INPUT -i lo -j ACCEPT
  > iptables -A OUTPUT -o lo -j ACCEPT
  > iptables -P INPUT DROP
  > iptables -P OUTPUT DROP
  > 
  > iptables -A OUTPUT -p tcp -m multiport --sports 1024:65535 \
  > -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
  > iptables -A OUTPUT -p udp -m multiport --sports 1024:65535 \
  > -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
  > iptables -A OUTPUT -p icmp -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
  > iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
  > ```
  >
  > 

目标扩展









