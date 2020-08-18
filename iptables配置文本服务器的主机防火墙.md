> 实验环境：LAMP

```
CentOS、Apache HTTP、MySQL、PHP
```

> 配置防火墙策略

- 使用nmap扫描未配置防火墙的Web服务器

  ```
  nmap -sS -O 192.168.100.123
  ```

- 防火墙策略

  ```sh
  # 允许ssh连接
  iptables -A INPUT -p tcp --dport 22 -j ACCEPT
  iptabels -A OUTPUT - p tcp --sprot 22 -j ACCEPT
  
  # 允许web客户端的连接
  iptables -A INPUT -p tcp --dport 80 -j ACCEPT
  iptabels -A OUTPUT - p tcp --sprot 80 -j ACCEPT
  
  # 修改默认的防火墙策略，设置INPUT、OUTPUT链的规则
  # -P, --policy chain target
  iptables -P INPUT DROP
  iptables -P OUTPUT DROP
  
  # lo环回接口流量也是通过iptables来管理
  # 允许本地连接127.0.0.1
  iptables -A INPUT -i lo -j ACCEPT
  iptables -A OUTPUT -o lo -j ACCEPT
  
  # 查看结果
  iptables -L -n -v
  
  # 使用nmap扫描安全加固后的web服务器
  nmap -sS -O 192.168.100.123
  ```

  ```sh
  # 使用iptables扩展:v2
  iptables -A INPUT -p tcp -m multiport --dports 22,80 -j ACCEPT
  iptables -A OUTPUT -p tcp -m multiport --sports 22,80 -j ACCEPT
  
  iptables -A INPUT -p tcp --dport 3306 -m iprange --src-range \
  192.168.1.1-192.168.1.3 -j ACCEPT
  iptables -A OUTPUT -p tcp --sport 3306 -m iprange --dst-range \
  192.168.1.1-192.168.1.3 -j ACCEPT
  
  iptables -A INPUT -i lo -j ACCEPT
  iptables -A OUTPUT -o lo -j ACCEPT
  
  iptables -P INPUT DROP
  iptables -P OUTPUT DROP
  ```

  

