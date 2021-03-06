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

18. 查看pid: select pg_backend_pid()

19. regexp_split_to_table 切割字符查询

select *from tbl_device_info where sn in (select regexp_split_to_table($1, ','))  $1= [devsnList.join()]

20. 时间转化成秒
select extract(epoch FROM (begin_time - end_time ))



数据类型说明：

character varying(n), varchar(n):变长，最大为1G.存储空间为：4+n，如果使用charcter varying 不带长度，那么该类型可以接受任何长度的字符串。

character(n), char(n):定长，不足补空白，最大1G.存储空间为：4+n

text: 变长，无长度限制。

显示时间格式： show datestyle ; set datestyle = 'YMD'

访问复合类型时： select (persion_info).name from author ; 要加()

insert into author values(('张三', 29, TRUE), '自传')；

update author set persion_info = ('王二', 49, TRUE) where id = 1; or set persion_info = ROW('王二', 39, TRUE) where uid = 1;

update author set persion_info.name = '王二二' where id = 2; or set persion_info.age = (persion_info).age + 1 where id = 2;

查看索引大小：select pg_indexes_size('');

create index tb_key on tb_name using gin (collon); or using gist (collon);

创建模式：

create schema addfd ;
alter chema addfd rename to newname;
alter chema addfd owner to newowner;


创建复合主键,unique：

create table test02(id int , id2 int, id3 int, not varchar(20), constraint pk_test02 primary key(id, id2), constraint uk_test02 unique(id3));

按照其他表创建新表：

create table baby (like test02); 没有复制相关的约束条件

create table baby(like test02 including all);完全复制

唯一约束：

create table books {
  book_no integer UNIQUE, //字段约束
  name text,
  price numeric
};         
or create table books {
   book_no integer,
   name text,
   price numeric,
   UNIQUE(book_no) //表约束
}；

create unique index index_unique_class_no on table_name(collon);

外键约束：

create table class {
   class_no int primary key,
   class_name varchar(40)
};

create tale student {
  student_no int primary key,
  student_name varchar(40),
  age int, 
  class_no int REFERENCES class(class_no)
};

删除普通字段：

alter table drop column class_teacher;

删除外键字段：

alter table class drop column class_no cascade;

增加约束：

alter table student add CHECK(age <16);
alter table class add constraint unique_class_teacher unique(class_teacher);

增加非空约束：

alter table student alter column student_name set not null;

删除约束：

alter table student drop constraint student_age_check;

删除非空约束:

alter table student alter column student_name drop not null;

修改默认值：

alter table student alter column age set default 15;

删除默认值：

alter table student alter column age drop default;

修改字段数据类型：

alter table student alter column student_name type text;

只有在字段里现有的每个项都可以隐式转换成新类型时，才能成功。比如把varchar(n) 转成integer，会因为字符串无法隐式转换成数字类型而失败。

重命名字段：

alter table books rename column book_no to book_id; 

表继承：

create table persions {
  name text,
  sex boolean,
  age int 
};

create table students {
  class_no int 
}INHERITS (persions);


如： insert into students values('张三', 15, true, 1);
    insert into students values('李四', 14, false, 2);
    
 select *from persions;
 
当查询父表时，会把这个父表中的子表的数据也查询出来，反之不行。往父表中更新数据，子表不能看到这条数据的。
如果只想把父表本身数据查询出来，需要在查询的表名前加'only'。select *from only persions; 
所有父表的检查约束和非空约束都会自动被所有子表继承，不过其他类型的约束（唯一，主键，外键）则不会被继承。

在已有数据的表上添加索引，需要加concurrently进行并发创建索引，避免阻塞。 如：create index concurrently idx_test_note on table_name(note);

如果想重建原来的索引，可以先另外建个新的，然后删除原来的，如：create index concurrently idx_test_note2 on table_name(note); drop index idx_test_note;

并发创建索引时，如果创建过程被取消，可能会留下一个无效的索引，导致更新变慢。如何创建的事唯一索引，这个无效索引会导致插入重复值失败。此时，手动删除即可。


查看锁的信息，select *from pg_locks;

查看数据库启动时间：
select pg_postmaster_start_time();

查看数据库版本： 

select version();

查看配置加载时间：
select pg_conf_load_time();

查看数据库是否备份：

select pg_is_in_backup(), pg_backup_start_time();

查看数据库大小：

 select pg_database_size('rs01'), pg_size_pretty(pg_database_size('rs01'));

查看表大小：

select pg_size_pretty(pg_relation_size('a')); //只计算表大小，不包括索引大小

select pg_size_pretty(pg_total_relation_size('a'));//包括索引

查看索引大小：

select pg_size_pretty(pg_indexes_size('表名'));

通过pg_stat_user_indexes.idx_scan可检查利用索引进行扫描的次数；这样可以确认那些索引可以清理
select idx_scan from pg_stat_user_indexes;

查看表空间大小：

select pg_size_pretty(pg_tablespace_size('pg_global'));

查看表对应的数据文件：

select pg_relation_filepath('tbl_device_info');


修改配置后，生效方式(需要重启的此方法行不通)：

1.pg_ctl reload   2. select pg_reload_conf()

查询长时间运行的SQL：

select *from pg_stat_activity;

取消一个正在长时间执行的SQL：
pg_cancel_backend(pid); pg_terminate_backend(pid);

explain 命令解释：

 explain select *from tbl_device_info limit 1;
                                             QUERY PLAN                                             
----------------------------------------------------------------------------------------------------
 Limit  (cost=0.00..0.03 rows=1 width=97)
   ->  Remote Subquery Scan on all (db_1,db_2,db_3,db_4)  (cost=0.00..9095.36 rows=342636 width=97)
         ->  Limit  (cost=0.00..9095.36 rows=342636 width=97)
               ->  Seq Scan on tbl_device_info  (cost=0.00..9095.36 rows=342636 width=97)
(4 rows)

Seq Scan “全表扫描”， 
cost=0.00..0.03 rows=1 width=97： ‘0.00’:表示启动成本,'0.03':返回所有数据成本, 'rows':表示会返回多少行, 'width':每行平均宽度多少字节

顺序扫描一个数据块：cost = 1; 随机扫描一个数据块，cost = 4; 处理一个数据行的CPU, cost= 0.01; 处理一个索引行的CPU， cost= 0.005;每个操作符的CPU代价为0.0025 

explain (format json/xml/YAML) select *from tbl_device_info limit 1;
加上 analyze 后 可获得更精确的执行计划。

explain (analyze true, buffers true) select *from  tbl_device_info limit 1;

规则：

create RULE rule_mytab_insert as on insert to mytab do also insert into mytab_log(oprtype, oprtime, id, note) values();

create RULE rule_mytab_update as on update to mytab do also (insert into mytab_log() values(); insert into mytab_log()values());

create RULErule_mytab_delete as on delete to mytab do also insert into mytab_log() values();

归还视图权限给某个用户：

grant select on myview to user01;

自定义函数：
CREATE OR REPLACE FUNCTION ruishi.fn_user_add_dev_2(i_user_id bigint, i_dev_id bigint)

 RETURNS record
 LANGUAGE plpgsql
 
AS $function$

DECLARE
    v_dev_sn VARCHAR;
    v_alias VARCHAR;
    v_devsn_num_tail INTEGER := 0;
    v_is_default BOOLEAN := false;
    v_is_device_admin BOOLEAN := false;
    v_retcode INTEGER := 0;
    v_retmsg VARCHAR := 'Success';
    v_mfrs_id INTEGER := 0;
    v_modetype INTEGER := 0;
    v_softversion VARCHAR;
    v_ctrlversion VARCHAR;
    v_alias_prefix VARCHAR;
    
BEGIN
    SELECT sn, softversion, ctrlversion, modetype INTO v_dev_sn, v_softversion, v_ctrlversion, v_modetype FROM tbl_device_info WHERE devid=i_dev_id and mfrs_id = (select mf
rs_id from tbl_user_login where uid=i_user_id);
    IF FOUND THEN
        IF EXISTS (SELECT 1 FROM tbl_user_devices WHERE uid=i_user_id AND devid=i_dev_id) THEN
            SELECT alias, is_default, is_device_admin INTO v_alias, v_is_default, v_is_device_admin FROM tbl_user_devices WHERE uid=i_user_id AND devid=i_dev_id;
        ELSE
            IF NOT EXISTS (SELECT 1 FROM tbl_user_devices WHERE uid=i_user_id AND is_default=true) THEN
                v_is_default := true;  -- if user has no default device, then set it for current
            END IF;

            IF NOT EXISTS (SELECT 1 FROM tbl_user_devices WHERE devid=i_dev_id AND is_device_admin=true) THEN
                v_is_device_admin := true;  -- if device has no admin, then set it for current
            END IF;

            SELECT mfrs_id, modetype INTO v_mfrs_id, v_modetype FROM tbl_device_info WHERE devid=i_dev_id;
            SELECT alias_prefix, devsn_num_tail INTO v_alias_prefix, v_devsn_num_tail FROM tbl_device_alias_rule WHERE mfrs_id=v_mfrs_id AND device_type=v_modetype;
            IF NOT FOUND THEN
                v_alias_prefix = 'Unknown';
            END IF;
            IF v_devsn_num_tail > 0 THEN
                v_alias := v_alias_prefix || '-' || upper(right(v_dev_sn, v_devsn_num_tail));
            ELSE
                v_alias = v_alias_prefix;
            END IF;
            INSERT INTO tbl_user_devices (uid, devid, alias, is_default, is_device_admin, is_shared) VALUES (i_user_id, i_dev_id, v_alias, v_is_default, v_is_device_admin, 
true);
        END IF;
    ELSE
        v_retcode := 12003;
        v_retmsg := FORMAT('Device (devid: %s) not exists', i_dev_id);
    END IF;

    RETURN (i_dev_id, v_alias, v_is_default, v_is_device_admin, v_softversion, v_ctrlversion, v_modetype, v_retcode, v_retmsg);
END;
$function$

使用方法：
select devid, alias, is_default, is_device_admin, softversion, ctrlversion, modetype, retcode, retmsg from fn_user_add_dev_2($1, $2) as (devid bigint, alias varchar, is_default bool, is_device_admin bool, softversion varchar, ctrlversion varchar, modetype integer, retcode integer, retmsg varchar)


2.delete all
CREATE OR REPLACE FUNCTION ruishi.fn_delete_device_all(devsn_list_str character varying, delete_register_record boolean DEFAULT false)
 RETURNS SETOF device_sn_id
 LANGUAGE plpgsql
AS $function$
DECLARE
    devid_rows RECORD;
BEGIN
    FOR devid_rows IN SELECT devid, sn FROM tbl_device_info WHERE sn IN (select regexp_split_to_table(devsn_list_str, ',')) LOOP
        PERFORM fn_device_deleting(devid_rows.devid, delete_register_record);
        RETURN NEXT devid_rows;
    END LOOP;
END;
$function$


3.delete

CREATE OR REPLACE FUNCTION ruishi.fn_device_deleting(dev_id bigint, delete_register_record boolean DEFAULT false)
 RETURNS bigint
 LANGUAGE plpgsql
AS $function$
DECLARE
    uid_rows RECORD;
BEGIN
    insert into tbl_admin_reset_device_log (op_result, delete_register_record) values (dev_id, delete_register_record);
    --DELETE FROM tbl_user_devices WHERE devid=dev_id;
    FOR uid_rows IN SELECT uid FROM tbl_user_devices WHERE devid=dev_id LOOP
        PERFORM fn_user_del_dev(uid_rows.uid, dev_id);
    END LOOP;
    DELETE FROM tbl_device_faultevent WHERE devid=dev_id;
    DELETE FROM tbl_device_keyevent WHERE devid=dev_id;
    --DELETE FROM tbl_device_cleanmap WHERE devid=dev_id;
    --DELETE FROM tbl_device_cleaninfo WHERE devid=dev_id;
    UPDATE tbl_device_cleaninfo SET deleted=true WHERE devid=dev_id;
    DELETE FROM tbl_device_cycle_status_log WHERE devid=dev_id;

    IF delete_register_record THEN
        DELETE FROM tbl_device_info WHERE devid=dev_id;
    ELSE
        UPDATE tbl_device_info SET cycle_status=1, cycle_utime=ctime WHERE devid=dev_id;
    END IF;

    RETURN dev_id;
END;
$function$

4.

CREATE OR REPLACE FUNCTION ruishi.fn_user_del_dev(i_user_id bigint, i_dev_id bigint)
 RETURNS record
 LANGUAGE plpgsql
AS $function$
DECLARE
    v_user_dev_cnt INTEGER := 0;
    v_is_device_admin BOOLEAN := false;
    v_uid_list_str VARCHAR;
    v_retcode INTEGER := 0;
    v_retmsg VARCHAR := 'Success';
BEGIN
    SELECT is_device_admin INTO v_is_device_admin FROM tbl_user_devices WHERE uid=i_user_id AND devid=i_dev_id;
    IF FOUND THEN
        IF v_is_device_admin = true THEN
            SELECT array_to_string(ARRAY(SELECT uid FROM tbl_user_devices WHERE devid=i_dev_id), ',') INTO v_uid_list_str;
            DELETE FROM tbl_user_devices WHERE devid=i_dev_id;
        ELSE
            DELETE FROM tbl_user_devices WHERE uid=i_user_id AND devid=i_dev_id;

            SELECT count(*) INTO v_user_dev_cnt FROM tbl_user_devices WHERE uid=i_user_id;
            IF v_user_dev_cnt = 1 THEN
                UPDATE tbl_user_devices SET is_default=true WHERE uid=i_user_id AND is_default=false;
            END IF;
        END IF;
    ELSE
        v_retcode := 10014;
        v_retmsg := 'User device ' || i_dev_id || ' not exists';
    END IF;

    RETURN (i_dev_id, v_uid_list_str, v_retcode, v_retmsg);
END;
$function$

设置一次显示完全

unset EDITOR
unset PAGER
