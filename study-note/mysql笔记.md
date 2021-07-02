[TOC]


 ## 数据库备份恢复

~~~shell
    查看版本号：select version();
    备份数据库（数据+结构）：[root@zhao531-1 ~]# mysqldump -u root -p <dbname> > myyangbenku.sql
    操作远程：mysqldump -h 10.xx.xx.xx -u root -p <dbname> > myyangbenku.sql
    参数：--max_allowed_packet=512M 报错got packet bigger than 'max_allowed_packet' bytes 时加此参数。
    数据恢复：mysql -uroot -p<password> <dbname> < /mytest_bak.sql   #库必须保留，空库也可
~~~
## 触发器

~~~sql
CREATE TRIGGER trigger_name trigger_time trigger_event ON tb_name FOR EACH ROW trigger_stmt
  trigger_name：触发器的名称
  tirgger_time：触发时机，为BEFORE或者AFTER
  trigger_event：触发事件，为INSERT、DELETE或者UPDATE
  tb_name：表示建立触发器的表明，就是在哪张表上建立触发器
  trigger_stmt：触发器的程序体，可以是一条SQL语句或者是用BEGIN和END包含的多条语句
  所以可以说MySQL创建以下六种触发器：
  BEFORE INSERT,BEFORE DELETE,BEFORE UPDATE
  AFTER INSERT,AFTER DELETE,AFTER UPDATE
  
  -- 创建触发器
  DELIMITER $
  CREATE TRIGGER user_log AFTER INSERT ON users FOR EACH ROW
  BEGIN
  DECLARE s1 VARCHAR(40)character set utf8;
  DECLARE s2 VARCHAR(20) character set utf8;#后面发现中文字符编码出现乱码，这里设置字符集
  SET s2 = " is created";
  SET s1 = CONCAT(NEW.name,s2);     #函数CONCAT可以将字符串连接
  INSERT INTO logs(log) values(s1);
  END $
  DELIMITER ;
  
  delimiter ##
  
-- 创建触发器，查询结果赋值到变量
create trigger before_insert_order before insert on orders for each row
begin
    -- 取出 goods 表中对应 id 的库存
    -- new 代表 orders 表中新增的数据
    select goods_num from goods where id = new.goods_id into @num;
    
    -- 用即将插入的 orders 表中的库存和 goods 表中的库存进行比较
    -- 如果库存不够，中断操作
    if @num < new.goods_num then
        -- 中断操作：暴力解决，主动出错
        insert into xxx values(xxx);
    end if;
end
##
delimiter ;

  -- 查看所有触发器
  SELECT * FROM information_schema.triggers;
  -- 删除触发器
  drop trigger  user_log;
~~~

