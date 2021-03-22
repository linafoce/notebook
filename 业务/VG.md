### VG 中控

http://qacrmwx.vgrass.com/centercontrol.web/index.html#/

http://qacrmwx.vgrass.com/etlservice/topicDirectoryNew?companyCode=P40002

12312345678/12345678



#### VG Kibana

kibana: [10.8.1.212](10.8.1.212):5601

用户: read-only

密码: V66WrN15uCH5CX9x



## VG etl

etl地址: http://10.8.1.231:8080/

账号: admin

密码：bizvane&BIZVANE##SF$$etl2018



### VG HUE

hue地址: http://10.8.1.223:8888

账号: jiangyujun

密码: ixQl6UDm5KItXrSf



账号: developer

密码: MlmIx4ZtXwxcnO4b



## VG服务器

IP                  10.8.1.232
HostName    crm-etl-prd
Domain        idc.vgrass.com

密码: Jhcrm@prd2021!



### VG vpn账号

地址：https://sslvpn.vgrass.com:4433/

账号： crmuser01 或者 crmuser02 

密码：Jinhong@crmvpn2021!!

### 伯俊数据库地址

数据库：oracle 

IP：10.8.5.189 

端⼝：1521 

实例：tbosdb

**应该是只有查询权限**

用户：alicrm_bj_initdata

密码：bj_pw123

-----------------------------

**这个权限高一点，能改表数据**

用户： bosnds3

密码： abc123



### 对接中产生的线上问题，要业务调整的

1. 订单表中的vouchers_no 字段，如何存放多张券

2. 订单表中的original_orderno, 可能要调整取数程序，消费程序，订单的判断逻辑如何根据这个字段来判断退货单，换货单

3. 核销券的数据，取ipos使用券，之前的表全部作废，初始化数据只取未核销的，且数据有效期在3月15号还可以使用的,那么新的表还要加什么字段吗

4. 初始化合并的会员卡号规则，TW VG下都有会员，卡号生成规则

5. 一个导购服务于多家门店，在商秀使用的时候会来回调店

6. 职务我们只支持店长和导购，其他的职务比如, **历史数据非高级顾问,见习店长,高级顾问,其他 **，如何刷数据

7. 会员发券伯俊会加字段，领取后多少天内有效，目前不支持我们的线上场景

8. 会员手机号为空（过不了线上程序验证）

9. 会员手机号重复（数据一般为同一个会员有两条不同的数据，所属店铺，卡等级，开卡店铺都不同）,**这种情况要看取哪一条有效的**

10. 会员手机号格式不对（正常应该是11位，但实际有12位，或者10位的），**这种会员线上小程序不能授权**

11. 开卡店铺，开卡导购为空,  **会赋值线上的默认店铺，默认导购**

12. 订单erpid关联不到会员，这种**订单会被算成非会销**

    



#### VG openapi接口文档，增量同步用的

https://www.yuque.com/rsebsg/ui5yxe/bdbuvn







#### 优惠券



#### VG挖坑

测openapi 订单时，要注意 商城订单是双写，写到俱乐部 和集团各一份

没有门店的



VG接口目前问题

1. 门店调用不通
   1. 门店要不要写集团卡
2. 员工调用不通
   1. 员工要不要写集团卡，集团下是否有员工信息

1. 会员查询没问题，返回值要改

2. 会员开卡 VG_CLUB 俱乐部品牌，没写id_card字段，集团卡写了, level code 集团卡和俱乐部到底如何区分品牌,

   没有返回erp_id

   1. 卡等级目前是，先匹配俱乐部的卡等级，集团卡的卡等级给默认的
   2. 开卡门店，开卡导购，服务门店，服务导购,如果不传每个品牌是不是都给默认值

3. 修改会员卡卡等级的接口 changeJHCardLevel，404

   1. 会员的卡等级变更，只修改了卡等级level_id，没有修改level_code

4. 新增积分，积分流水的接口，问题系统报错，changeDate的日期格式转不出来，trace: 4c6d6063ebb9a11a

   1. 集团会不会传积分过来

5. 查询积分流水

   1. 根据会员卡card_no 查询不到
   2. 根据积分流水id查询报错
   3. 积分流水是不是一式两份
   4. 全品牌的积分是不是要统一更新

6. 券

   1. 券同步接口 /request/openApiService/receiveJHCoupons，目前调不同
   2. 集团商城同步的券要写几分，别的品牌应该用不了这个接口，券定义是如何操作的

7. 订单

   1. 目前两个接口都调不通，只进了队列
   2. 会员没有erpId，怎么过serviceOrder
   3. 如果是退单要传原订单号，原订单号有什么作用， 只是记录吗
   4. 集团商城要通过订单算积分,通缩get_integral?
   5. 同步接口不能修改



1. 重新开卡积分同步，开卡之后推消息，处理优惠券，积分
2. 不同品牌插入相同的线下积分流水要不要拦截
3. 查询Vg
4. 开卡门店，开卡导购

https://quicka-shenzhen.aliyun.com (用这个做测试）

[test1@1866616984236757.onaliyun.com](mailto:test1@1866616984236757.onaliyun.com) 40e7a2dd892e4b26



