# 全流程调度
## Azkaban部署

### 单机部署

```shell
[wesharecn@hadoop ~]$ vi azkaban-solo-server-0.1.0/conf/azkaban.properties
												default.timezone.id=Asia/Shanghai
[wesharecn@hadoop azkaban-solo-server-0.1.0]$ bin/start-solo.sh
```



## MySQL建表

```sql
CREATE DATABASE `gmall_report`;
USE DATABASE `gmall_report`;

#（1）访客统计
DROP TABLE IF EXISTS ads_visit_stats;
CREATE TABLE `ads_visit_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `is_new` VARCHAR(255) NOT NULL COMMENT '新老标识,1:新,0:老',
  `recent_days` INT NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `channel` VARCHAR(255) NOT NULL COMMENT '渠道',
  `uv_count` BIGINT(20) DEFAULT NULL COMMENT '日活(访问人数)',
  `duration_sec` BIGINT(20) DEFAULT NULL COMMENT '页面停留总时长',
  `avg_duration_sec` BIGINT(20)  DEFAULT NULL COMMENT '一次会话，页面停留平均时长',
  `page_count` BIGINT(20) DEFAULT NULL COMMENT '页面总浏览数',
  `avg_page_count` BIGINT(20) DEFAULT NULL COMMENT '一次会话，页面平均浏览数',
  `sv_count` BIGINT(20) DEFAULT NULL COMMENT '会话次数',
  `bounce_count` BIGINT(20) DEFAULT NULL COMMENT '跳出数',
  `bounce_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '跳出率',
  PRIMARY KEY (`dt`,`recent_days`,`is_new`,`channel`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;

#（2）页面路径分析
DROP TABLE IF EXISTS ads_page_path;
CREATE TABLE `ads_page_path` (      
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `source` VARCHAR(255) DEFAULT NULL COMMENT '跳转起始页面',
  `target` VARCHAR(255) DEFAULT NULL COMMENT '跳转终到页面',
  `path_count` BIGINT(255) DEFAULT NULL COMMENT '跳转次数',
  UNIQUE KEY (`dt`,`recent_days`,`source`,`target`) USING BTREE     
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

#（3）用户统计
DROP TABLE IF EXISTS ads_user_total;
CREATE TABLE `ads_user_total` (          
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,0:累积值,1:最近1天,7:最近7天,30:最近30天',
  `new_user_count` BIGINT(20) DEFAULT NULL COMMENT '新注册用户数',
  `new_order_user_count` BIGINT(20) DEFAULT NULL COMMENT '新增下单用户数',
  `order_final_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '下单总金额',
  `order_user_count` BIGINT(20) DEFAULT NULL COMMENT '下单用户数',
  `no_order_user_count` BIGINT(20) DEFAULT NULL COMMENT '未下单用户数(具体指活跃用户中未下单用户)',
  PRIMARY KEY (`dt`,`recent_days`)           
) ENGINE=INNODB DEFAULT CHARSET=utf8;

#（4）用户变动统计
DROP TABLE IF EXISTS ads_user_change;
CREATE TABLE `ads_user_change` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `user_churn_count` BIGINT(20) DEFAULT NULL  COMMENT '流失用户数',
  `user_back_count` BIGINT(20) DEFAULT NULL  COMMENT '回流用户数',
  PRIMARY KEY (`dt`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;

#（5）用户行为漏斗分析
DROP TABLE IF EXISTS ads_user_action;
CREATE TABLE `ads_user_action` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `home_count` BIGINT(20) DEFAULT NULL COMMENT '浏览首页人数',
  `good_detail_count` BIGINT(20) DEFAULT NULL COMMENT '浏览商品详情页人数',
  `cart_count` BIGINT(20) DEFAULT NULL COMMENT '加入购物车人数',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '下单人数',
  `payment_count` BIGINT(20) DEFAULT NULL COMMENT '支付人数',
  PRIMARY KEY (`dt`,`recent_days`) USING BTREE
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

#（6）用户留存率分析
DROP TABLE IF EXISTS ads_user_retention;
CREATE TABLE `ads_user_retention` (      
  `dt` DATE DEFAULT NULL COMMENT '统计日期',
  `create_date` VARCHAR(255) NOT NULL COMMENT '用户新增日期',
  `retention_day` BIGINT(20) NOT NULL COMMENT '截至当前日期留存天数',
  `retention_count` BIGINT(20) DEFAULT NULL COMMENT '留存用户数量',
  `new_user_count` BIGINT(20) DEFAULT NULL COMMENT '新增用户数量',
  `retention_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '留存率',
  PRIMARY KEY (`create_date`,`retention_day`) USING BTREE        
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

#（7）订单统计
DROP TABLE IF EXISTS ads_order_total;
 CREATE TABLE `ads_order_total` (   
  `dt` DATE NOT NULL COMMENT '统计日期', 
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `order_count` BIGINT(255) DEFAULT NULL COMMENT '订单数', 
  `order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '订单金额', 
  `order_user_count` BIGINT(255) DEFAULT NULL COMMENT '下单人数',
  PRIMARY KEY (`dt`,`recent_days`)  
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

#（8）各省份订单统计
DROP TABLE IF EXISTS ads_order_by_province;
CREATE TABLE `ads_order_by_province` (
  `dt` DATE NOT NULL,
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `province_id` VARCHAR(255) NOT NULL COMMENT '统计日期',
  `province_name` VARCHAR(255) DEFAULT NULL COMMENT '省份名称',
  `area_code` VARCHAR(255) DEFAULT NULL COMMENT '地区编码',
  `iso_code` VARCHAR(255) DEFAULT NULL COMMENT '国际标准地区编码',
  `iso_code_3166_2` VARCHAR(255) DEFAULT NULL COMMENT '国际标准地区编码',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '订单数',
  `order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '订单金额',
  PRIMARY KEY (`dt`, `recent_days` ,`province_id`) USING BTREE       
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

#（9）品牌复购率
DROP TABLE IF EXISTS ads_repeat_purchase;
CREATE TABLE `ads_repeat_purchase` (         
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `tm_id` VARCHAR(255) NOT NULL COMMENT '品牌ID',
  `tm_name` VARCHAR(255) DEFAULT NULL COMMENT '品牌名称',
  `order_repeat_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '复购率',
  PRIMARY KEY (`dt` ,`recent_days`,`tm_id`)          
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

#（10）商品统计
DROP TABLE IF EXISTS ads_order_spu_stats;
CREATE TABLE `ads_order_spu_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `spu_id` VARCHAR(255) NOT NULL COMMENT '商品ID',
  `spu_name` VARCHAR(255) DEFAULT NULL COMMENT '商品名称',
  `tm_id` VARCHAR(255) NOT NULL COMMENT '品牌ID',
  `tm_name` VARCHAR(255) DEFAULT NULL COMMENT '品牌名称',
  `category3_id` VARCHAR(255) NOT NULL COMMENT '三级品类ID',
  `category3_name` VARCHAR(255) DEFAULT NULL COMMENT '三级品类名称',
  `category2_id` VARCHAR(255) NOT NULL COMMENT '二级品类ID',
  `category2_name` VARCHAR(255) DEFAULT NULL COMMENT '二级品类名称',
  `category1_id` VARCHAR(255) NOT NULL COMMENT '一级品类ID',
  `category1_name` VARCHAR(255) NOT NULL COMMENT '一级品类名称',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '订单数',
  `order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '订单金额', 
  PRIMARY KEY (`dt`,`recent_days`,`spu_id`)  
) ENGINE=INNODB DEFAULT CHARSET=utf8;

#（11）活动统计
DROP TABLE IF EXISTS ads_activity_stats;
CREATE TABLE `ads_activity_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `activity_id` VARCHAR(255) NOT NULL COMMENT '活动ID',
  `activity_name` VARCHAR(255) DEFAULT NULL COMMENT '活动名称',
  `start_date` DATE DEFAULT NULL COMMENT '开始日期',
  `order_count` BIGINT(11) DEFAULT NULL COMMENT '参与活动订单数',
  `order_original_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '参与活动订单原始金额',
  `order_final_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '参与活动订单最终金额',
  `reduce_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '优惠金额',
  `reduce_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '补贴率',
  PRIMARY KEY (`dt`,`activity_id` )
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

#（12）优惠券统计
DROP TABLE IF EXISTS ads_coupon_stats;
CREATE TABLE `ads_coupon_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `coupon_id` VARCHAR(255) NOT NULL COMMENT '优惠券ID',
  `coupon_name` VARCHAR(255) DEFAULT NULL COMMENT '优惠券名称',
  `start_date` DATE DEFAULT NULL COMMENT '开始日期',  
  `rule_name`  VARCHAR(200) DEFAULT NULL COMMENT '优惠规则',
  `get_count`  BIGINT(20) DEFAULT NULL COMMENT '领取次数',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '使用(下单)次数',
  `expire_count`  BIGINT(20) DEFAULT NULL COMMENT '过期次数',
  `order_original_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '使用优惠券订单原始金额',
  `order_final_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '使用优惠券订单最终金额',
  `reduce_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '优惠金额',
  `reduce_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '补贴率',
  PRIMARY KEY (`dt`,`coupon_id` )
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

## Sqoop导出

```shell
#!/bin/bash

hive_db_name=gmall
mysql_db_name=gmall_report

export_data() {
/home/wesharecn/sqoop-1.4.7/bin/sqoop export \
--connect "jdbc:mysql://node0:3306/${mysql_db_name}?useUnicode=true&characterEncoding=utf-8"  \
--username root \
--password root123 \
--table $1 \
--num-mappers 1 \
--export-dir /warehouse/$hive_db_name/ads/$1 \
--input-fields-terminated-by "\t" \
--update-mode allowinsert \
--update-key $2 \
--input-null-string '\\N'    \
--input-null-non-string '\\N'
}

case $1 in
  "ads_activity_stats" )
    export_data "ads_activity_stats" "dt,activity_id"
  ;;

  "ads_coupon_stats" )
    export_data "ads_coupon_stats" "dt,coupon_id"
  ;;

  "ads_order_by_province" )
    export_data "ads_order_by_province" "dt,recent_days,province_id"
  ;;

  "ads_order_spu_stats" )
    export_data "ads_order_spu_stats" "dt,recent_days,spu_id"
  ;;

  "ads_order_total" )
    export_data "ads_order_total" "dt,recent_days"
  ;;

  "ads_page_path" )
    export_data "ads_page_path" "dt,recent_days,source,target"
  ;;

  "ads_repeat_purchase" )
    export_data "ads_repeat_purchase" "dt,recent_days,tm_id"
  ;;

  "ads_user_action" )
    export_data "ads_user_action" "dt,recent_days"
  ;;

  "ads_user_change" )
    export_data "ads_user_change" "dt"
  ;;

  "ads_user_retention" )
    export_data "ads_user_retention" "create_date,retention_day"
  ;;

  "ads_user_total" )
    export_data "ads_user_total" "dt,recent_days"
  ;;

  "ads_visit_stats" )
    export_data "ads_visit_stats" "dt,recent_days,is_new,channel"
  ;;
  "all" )
    export_data "ads_activity_stats" "dt,activity_id"
    export_data "ads_coupon_stats" "dt,coupon_id"
    export_data "ads_order_by_province" "dt,recent_days,province_id"
    export_data "ads_order_spu_stats" "dt,recent_days,spu_id"
    export_data "ads_order_total" "dt,recent_days"
    export_data "ads_page_path" "dt,recent_days,source,target"
    export_data "ads_repeat_purchase" "dt,recent_days,tm_id"
    export_data "ads_user_action" "dt,recent_days"
    export_data "ads_user_change" "dt"
    export_data "ads_user_retention" "create_date,retention_day"
    export_data "ads_user_total" "dt,recent_days"
    export_data "ads_visit_stats" "dt,recent_days,is_new,channel"
  ;;
esac
```

>关于导出update还是insert的问题
>
>Ø --update-mode：
>
>updateonly  只更新，无法插入新数据
>
>​    allowinsert  允许新增 
>
>Ø --update-key：允许更新的情况下，指定哪些字段匹配视为同一条数据，进行更新而不增加。多个字段用逗号分隔。
>
>Ø --input-null-string和--input-null-non-string：
>
>分别表示，将字符串列和非字符串列的空串和“null”转义。
>
>Hive中的Null在底层是以“\N”来存储，而MySQL中的Null在底层就是Null，为了保证数据两端的一致性。在导出数据时采用--input-null-string和--input-null-non-string两个参数。导入数据时采用--null-string和--null-non-string。

## 工作流程配置

### azkaban.project

```yaml
azkaban-flow-version: 2.0
```

### gmall.flow

```yaml
nodes:
  - name: mysql_to_hdfs
    type: command
    config:
     command: /home/wesharecn/bin/mysql_to_hdfs.sh all ${dt}
    
  - name: hdfs_to_ods_log
    type: command
    config:
     command: /home/wesharecn/bin/hdfs_to_ods_log.sh ${dt}
     
  - name: hdfs_to_ods_db
    type: command
    dependsOn: 
     - mysql_to_hdfs
    config: 
     command: /home/wesharecn/bin/hdfs_to_ods_db.sh all ${dt}
  
  - name: ods_to_dim_db
    type: command
    dependsOn: 
     - hdfs_to_ods_db
    config: 
     command: /home/wesharecn/bin/ods_to_dim_db.sh all ${dt}

  - name: ods_to_dwd_log
    type: command
    dependsOn: 
     - hdfs_to_ods_log
    config: 
     command: /home/wesharecn/bin/ods_to_dwd_log.sh all ${dt}
    
  - name: ods_to_dwd_db
    type: command
    dependsOn: 
     - hdfs_to_ods_db
    config: 
     command: /home/wesharecn/bin/ods_to_dwd_db.sh all ${dt}
    
  - name: dwd_to_dws
    type: command
    dependsOn:
     - ods_to_dim_db
     - ods_to_dwd_log
     - ods_to_dwd_db
    config:
     command: /home/wesharecn/bin/dwd_to_dws.sh all ${dt}
    
  - name: dws_to_dwt
    type: command
    dependsOn:
     - dwd_to_dws
    config:
     command: /home/wesharecn/bin/dws_to_dwt.sh all ${dt}
    
  - name: dwt_to_ads
    type: command
    dependsOn: 
     - dws_to_dwt
    config:
     command: /home/wesharecn/bin/dwt_to_ads.sh all ${dt}
     
  - name: hdfs_to_mysql
    type: command
    dependsOn:
     - dwt_to_ads
    config:
      command: /home/wesharecn/bin/hdfs_to_mysql.sh all
```

