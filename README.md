# postgresql
命令参数:

1.只想显示匹配的表:\dt

2.只匹配索引：\di

3.只显示序列：\ds

4.只显示视图：\dv

5.只显示函数：\df

6.显示执行时间：\timing

7.更全的显示：\d+

8.显示表空间：\db

9.显示所有用户：\du or \dg

10.显示表的权限分配：\dp or \z

11.设置编码格式：\encoding uft8;

12.设置输出格式：\pset border 0: 表示输出内容无边框  \pset border 1: 边框只在内部 \pset border 2: 内外都有边框

13.列输出: \x

14.输出一行信息： \echo

15. 事务自动提交关闭：\set AUTOCOMMIT off

16. 输入部分命令，两次tab键可以自动补全

17.显示某个命令的实际执行过程： \set ECHO_HIDDEN on|off

数据类型说明：

character varying(n), varchar(n):变长，最大为1G.存储空间为：4+n，如果使用charcter varying 不带长度，那么该类型可以接受任何长度的字符串。

character(n), char(n):定长，不足补空白，最大1G.存储空间为：4+n

text: 变长，无长度限制。

显示时间格式： show datestyle ;

