## VG接口文档

注：postman 环境，其实我之前分享给很多人了，如果还要我可以重新发一下

### 1.  验签（除非自己写代码，这个我有python代码，这一块我先不写了）



### 2. 会员

  1. 会员查询

     条件三选一，phone如果写了phone这个字段，那么他就会校验里面的值

     {

     ​    "brandCode": "VG-CLUB",

     ​    "phone":"18953270148",

     ​    "cardNo": "VG210304000013",

     ​    "unionId": ""

     }

  2. 会员开卡

     **主要问题： 会员存两份，会员卡卡号根据当天开卡的会员数据顺序生成, 如 VG2021030300001**

     **会员的开卡门店，服务门店有自己的生成规则**

     1.开卡门店，开卡导购不隔离，也就是说集团卡的门店，导购可以是俱乐部的
     2.服务门店，服务导购，俱乐部下会隔离

     

     1.集团卡 开卡门店 = 服务门店， 开卡导购 = 服务导购
     2.俱乐部 开卡门店 = 集团卡开卡门店 ，开卡导购 = 集团卡开卡导购
     服务门店 = 校验俱乐部下是否存在，不存在则是俱乐部默认门店
     服务导购 = 校验俱乐部下是否存在，不存在则是俱乐部默认门店

     

     锦泓集团卡 和 俱乐部的开卡门店，开卡导购是同一个

     如果带参开卡 ，集团卡的开卡门店 = 服务门店，开卡导购 = 服务导购，会使用参数在企业下查询

     如果带参入会，俱乐部的开卡门店，开卡导购会使用集团卡参数，服务门店，服务导购会去俱乐部下去查，没找到就是默认

     

     {

     ​    "brandCode": "VG-CLUB",

     ​    "phone": "18953270148",

     ​    "name": "相霆威",

     ​    "sex": "2",

     ​    "province": "省",

     ​    "city": "城市",

     ​    "county": "区域",

     ​    "address": "啦啦啦",

     ​    "email": "1067655669@qq.com",

     ​    "idCard": "14234324234234",

     ​    "birthday": "2020-12-01",

     ​    "storeCode": "VG001",

     ​    "guideCode": "VG001",

     ​    "levelCode": "01",

     ​    "source": 1,

     ​    "unionId": "",

     ​    "serviceGuideCode": "VG001",

     ​    "serviceStoreCode": "VG001",

     ​    "openCardTime": "2021-01-01 00:00:00",

     ​    "remark": "无"

     }

  3. 会员修改卡等级( 没有什么要说的)

     {

     ​    "brandCode": "VG-CLUB",

     ​    "cardNo": "VG210304000013",

     ​    "oldLevelCode": "VG01",

     ​    "newLevelCode": "VG17",

     ​    "detail": "明细"

     }

### 积分 

1. **最主要的问题，我们取到的线下积分流水ID，有可能不同品牌的是重复的，所以我们制定了规则(接口文档yuque上有，我这里再写一遍)**

   VG_POS_#{INTEGRAL_ID}, 线下是这么传上来的，自己造的时候最好也这样

2. **查询一个会员的积分接口，查询的是所有品牌下的积分流水，为的是能根据所有的积分流水（TW,VG都包含），对应上他的会员总积分**

3. **积分流水同步的时候，会一起算会员总积分countIntegral字段，并且t_mbr_integral_record插入记录**



###   优惠券（目前只有核销接口，其他的是跟VG商城做同步用的，可以先不看）

1. **上游发券规则**

   1. 如果我创建的券是在集团里面，那么这张券只有集团能发，俱乐部一个道理

   2. 新增了一个字段，适用俱乐部字段, applicable_brand_codes

      ![image-20210308151832685](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20210308151832685.png)

   3. 核销的时候要校验券的品牌，是不是JH集团发的，或者是不是VG-CLUB发的，根据brand_code校验

   4. 接口的适用品牌字段，不做校验

   

   

   ### 订单接口

   1. 目前只有pos订单接口
   2. 订单接口中的coupon对象，payments对象，其实业务逻辑不明朗（没啥用）
   3. 主要看一下订单的字段,订单表 billtime,placeordertime,paytime, tradeamount,commodity_amount，erp_id, membercode (**主要大数据算业绩**)
   4. 订单明细表主要子段, service_guide_ids, product_guide_code, product_guide_name, trade_amount_detail, tag_price,sku (可能有打错的)
   5. **订单接口的消费问题不知道结没解决，影响营销模块触发活动，message模块发短信**
   6. 明细表中 productNo，每买一个商品会写商品表的offline_product_code

   

   ### 商品同步接口

   1. 新增字段 offline_product_code, 拼接方式 #{brand_code}_#{product_code}，程序里拼接的
   2. 颜色，尺寸，接口不传，用的是specification_json中的值，拆开了往表里写

   

   

   

   

   

   

   

   



​		



