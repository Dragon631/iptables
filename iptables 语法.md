**iptables 基本语法**

- man帮助

  ```sh
  iptables [-t table] {-A|-C|-D} chain rule-specification
  ip6tables [-t table] {-A|-C|-D} chain rule-specification
  iptables [-t table] -I chain [rulenum] rule-specification
  iptables [-t table] -R chain rulenum rule-specification
  iptables [-t table] -D chain rulenum
  iptables [-t table] -S [chain [rulenum]]
  iptables [-t table] {-F|-L|-Z} [chain [rulenum]] [options...]
  iptables [-t table] -N chain
  iptables [-t table] -X [chain]
  iptables [-t table] -P chain target
  iptables [-t table] -E old-chain-name new-chain-name
  rule-specification = [matches...] [target]
  match = -m matchname [per-match-options]
  target = -j targetname [per-target-options]
  ```

- 简化

  ```sh
  iptables [-t table] <option> <chain> <matching criteria> <target>
               表        选项      链         匹配条件         目标
  ```

- 链管理

  ```sh
  -N | --new-chain chain
  # 创建一个新的自定义规则链
  -F | --flush [chain]
  # 清空所有内置规则链，或清空指定的内置规则链
  -X | --delete-chain [chain]
  # 删除所有自定义规则链，或删除指定的自定义规则链
  -P | --policy chain target
  # 为内置规则链设置默认策略（目标），可以是ACCEPT或DROP
  -L | --list [chain]
  # 列出所有规则链的所有规则，或列出指定规则链的规则
  -Z | --zero [chain [rulenum]]
  # 重置所有规则链包和字节的计算器，或重置指定规则链的包和字节的计数器
  -E | --rename-chain old-chain new-chain
  # 重命名自定义的规则链名称
  ```

  - 列表命令（-L | --list [chain]）

    ```sh
    -n | --numeric
    # 以数值而不是名字的形式列出IP地址和端口号
    -v | --verbose
    # 列出每条规则的信息，例如字节和包计数器，规则选项和响应的网络接口
    -x | --exact
    # 列出计数器的精确值，而不是估值
    --line-numbers
    # 列出规则的在规则链中的行号
    ```

  - 规则管理

    ```sh
    -A | --append chain rule-specification
    # 添加一条规则到链的末尾
    iptables -t filter -A INPUT -j DROP
    # 在filter表的INPUT链里追加一条规则（作为最后一条规则）
    # 匹配所有访问本机IP的数据包，匹配到的丢弃
    # filter为默认表，可省略
    
    -I | --insert chain [rulenum] rule-specification
    # 插入一条规则到链的头部，或指定行的前面
    iptables -I INPUT -j DROP
    # 在filter表的IPUT链里插入一条规则（插入成第1条）
    iptables -I INPUT 3 -j DROP
    # 在filter表的INPUT链里插入一条规则（插入成第3条）
    # -I 链名 [规则号码]，如果不写规则号码，则默认是1
    # 确保规则号码 <= (已有规则数 + 1)， 否则报错
    
    -R | --replalce chain rulenum rule-specification
    # 替换指定等号的规则
    iptables -R INPUT 3 -J ACCEPT
    # 将原来编号为 3 的规则内容替换为 "-j ACCEPT"
    # 注意： 确保规则号码 <= 已有规则数，否则报错
    
    -D | --delete chain rule-specification
    # 删除指定的规则
    -D | --delete chain rulenum
    # 删除指定行号的规则
    iptables -D INPUT -s 192.168.0.1 -j DROP
    # 按内容匹配，不管其位置在哪里
    iptables -D INPUT 3 （按号码匹配）
    # 删除filter 表 INPUT链中的第三条规则（不管它的内容是什么）
    # 若规则列表中有多条相同的规则时，按内容匹配只删除序号最小的一条
    # 按号码匹配删除时，确保规则号码 <= 已有规则数，否则报错
    # 按内容匹配删除时，确保规则存在，否则报错
    ```

  - 指定匹配条件

    - 通用匹配

      - 可直接使用，不依赖于其他条件或扩展
      - 包括网络协议、IP地址、网络接口等条件

      ```sh
      [!] -i | --in-interface name
      -i eth0    # 从网络接口 eth0 进来
      -i ppp0    # 从网络接口 ppp0 进来
      ! -i tun0  # 非网络接口 tun0 进来
      -i eth+    # 从网络接口 eth0、eth1、eth2...进来
      
      [!] -o | --out-interface name
      -o ens192  # 从网络接口 ens192 出站
      
      [!] -p | --protocal protocol
      -p tcp
      -p udp
      -p icmp --icmp-type 类型 # ping: type 8; pong: type 0
      
      [!] -s | --source address [/mask] [,...]
      -s 192.168.0.1
      -s 192.168.1.0/24
      
      [!] -d | --destination address [/mask] [,...]
      -d 202.102.0.20
      -d 202.102.0.0/16
      -d www.baidu.com  # 匹配去往域名 www.baidu.com 的数据包
      
      [!] -f | --fragment name
      # 分段，很少使用
      ```

      ```sh
      [!] -p | --protocal protocol tcp # TCP头部匹配选项
      [!] --sport | --source-port port[:port]
      [!] --dport | --destination-port port[:port]
      [!] --tcp-flags mask comp
      [!] --syn
      [!] --tcp-option number
      # 示例：可以是个别端口，可以是端口范围
      --sport 1000       # 匹配源端口是 1000 的数据包
      --sport 1000:3000  # 匹配源端口是 1000-3000 的数据包（含1000、3000）
      --sport :3000      # 匹配源端口是 3000 以下的数据包（含3000）
      --sport 1000:      # 匹配源端口是 1000 以上的数据包（含1000）
      
      --dport 80
      --dport 6000:8000
      --dport :3000
      --dport 1000:
      # 注意：--sport 和 --dport 必须匹配 -p 参数使用
      
      [!] -p | --protocal protocol udp # UDP头部匹配选项
      [!] --sport | --source-port por[:port]
      [!] --dport | --destination-port port[:port]
      
      [!] -p | --protocal protocol icmp # ICMP类型匹配选项
      [!] --icmp-type {type [/code]|typename}
      # 可能通过iptables -p icmp -h 来查看类型
      any
      echo-reply (pong)
      destination-unreachable
         network-unreachable
         host-unreachable
         protocol-unreachable
         port-unreachable
         fragmentation-needed
         source-route-failed
         network-unknown
         host-unknown
         network-prohibited
         host-prohibited
         TOS-network-unreachable
         TOS-host-unreachable
         communication-prohibited
         host-precedence-violation
         precedence-cutoff
      source-quench
      redirect
         network-redirect
         host-redirect
         TOS-network-redirect
         TOS-host-redirect
      echo-request (ping)
      router-advertisement
      router-solicitation
      time-exceeded (ttl-exceeded)
         ttl-zero-during-transit
         ttl-zero-during-reassembly
      parameter-problem
         ip-header-bad
         required-option-missing
      timestamp-request
      timestamp-reply
      address-mask-request
      address-mask-reply
      ```

      - 匹配应用示例

      ```sh
      # 端口匹配
      -p udp --dport 53
      # 匹配网络中目的地址是 53 的UDP 协议数据包
      
      # 地址匹配
      -s 10.1.0.0/24 -d 172.71.0.0/16
      # 匹配来自10.1.0.0/24	去往172.17.0.0/16 的所有数据包
      
      # 端口和地址联合匹配
      -s 192.168.0.1 -d www.baidu.com -p tcp --dport 80
      # 配置来自192.168.0.1，去往www.baidu.com 的80 端口的TCP 协议数据包
      ```

      

    - 
      隐含匹配

      - 要求以特定的协议匹配作为前提
      - 包括端口、TCP标记、ICMP类型等条件

    - 显示匹配/扩展匹配

      - 要求以 '-m 扩展模块' 的形式明确指出类型
      - 包括多端口、MAC地址、IP范围、数据包状态等条件

  - 指定目标：target 

    > 命令：
    >
    > ```sh
    > -j | --jump target
    > ```
    >
    > filter 表中的target可以是：
    >
    > - 内置的目标：
    >   - ACCEPT: 允许通过
    >   - DROP: 丢弃数据包
    >   - RETURN: 停止遍历此链，继续前一个调用链中的下一个规则
    > - 用户自定义链
    > - 扩展目标
    >   - 参见 man iptables-extensions

  