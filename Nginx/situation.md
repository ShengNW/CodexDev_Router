【20260201】

本机公网与 Nginx 反代调研结果：

1) 公网 IP
- IPv4: 51.15.210.14
- IPv6: 2001:bc8:711:132a:dc00:ff:feda:571b
- 默认路由: 62.210.0.1 via ens2

2) Nginx 现有站点
- /etc/nginx/conf.d/llm.conf
  - server_name: llm.yeying.pub / www.llm.yeying.pub
  - 80 -> 301 https://llm.yeying.pub
  - 443 -> 反代 http://127.0.0.1:3011
  - 证书: /etc/letsencrypt/live/llm.yeying.pub/
- /etc/nginx/sites-available/router.conf (已在 sites-enabled)
  - server_name: shengnw.win
  - 80 -> 反代 http://127.0.0.1:3000

3) 监听端口 (ss -lntp)
- 80/443: nginx
- 3011: router
- 22: ssh

4) DNS 解析验证 (dig)
- llm.yeying.pub A -> 51.15.210.14 (指向本机)
- router.yeying.pub A -> 119.8.189.233 (不指向本机)
- router.yeying.pub AAAA -> 无

5) 结论
- 本机 Nginx 配置中未出现 router.yeying.pub
- 证书目录仅有 llm.yeying.pub
- 当前机器对公网暴露 80/443/3011 (UFW inactive)
