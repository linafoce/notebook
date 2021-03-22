### SQL验证

#### 店铺

##### 店铺name

select * from ALI_CRM_INITDATA_STORE_VG where store_name is null;

select * from ALI_CRM_INITDATA_STORE_VG where store_name = '';


select * from ALI_CRM_INITDATA_STORE_TW where store_name is null;

select * from ALI_CRM_INITDATA_STORE_TW where store_name = '';

##### 店铺code

select * from ALI_CRM_INITDATA_STORE_TW where sys_store_offline_code is null;

select * from ALI_CRM_INITDATA_STORE_VG where sys_store_offline_code is null;



select sys_store_offline_code
from ALI_CRM_INITDATA_STORE_TW
group by sys_store_offline_code
having count(sys_store_offline_code) > 1;

select sys_store_offline_code
from ALI_CRM_INITDATA_STORE_VG
group by sys_store_offline_code
having count(sys_store_offline_code) > 1;

--------



### 员工

#### 员工姓名校验

select * from ALI_CRM_INITDATA_EMP_TW where staffname is null 

select * from ALI_CRM_INITDATA_EMP_VG where staffname is null 

#### 员工code校验

select * from ALI_CRM_INITDATA_EMP_TW where staffcode is null; 

select * from ALI_CRM_INITDATA_EMP_VG where staffcode is null; 



select staffcode
from ALI_CRM_INITDATA_EMP_TW
group by staffcode
having count(staffcode) > 1;



select staffcode
from ALI_CRM_INITDATA_EMP_VG
group by staffcode
having count(staffcode) > 1;

#### 员工职位

##### 职位是不符合店长和导购的

select count(*)
from ALI_CRM_INITDATA_EMP_VG
where  position != '店长' and position != '导购'

select *
from ALI_CRM_INITDATA_EMP_VG
where position is null 



select count(*)
from ALI_CRM_INITDATA_EMP_TW
where  position != '店长' and position != '导购'

select *
from ALI_CRM_INITDATA_EMP_TW
where position is null 

#### 员工手机号

select count(*)
from ALI_CRM_INITDATA_EMP_TW
where phone is null 

select *
from ALI_CRM_INITDATA_EMP_VG
where phone is null 

select * 
from ALI_CRM_INITDATA_EMP_TW
where phone in (
select phone 
from ALI_CRM_INITDATA_EMP_TW
group by phone
having count(phone) > 1
) 

select * 
from ALI_CRM_INITDATA_EMP_VG
where phone in (
select phone 
from ALI_CRM_INITDATA_EMP_VG
group by phone
having count(phone) > 1
) 

---------

### 会员

#### 开卡日期

select count(*)
from ALI_CRM_INITDATA_VIP_TW
where opencardtime is null 

select count(*)
from ALI_CRM_INITDATA_VIP_VG
where opencardtime is null 

select count(*)
from ALI_CRM_INITDATA_VIP_TW
where opencardtime > to_char(sysdate,'yyyy-mm-dd HH24:MI:SS')

select count(*)
from ALI_CRM_INITDATA_VIP_VG
where opencardtime > to_char(sysdate,'yyyy-mm-dd HH24:MI:SS')

select count(to_date(opencardtime,'yyyy-mm-dd HH24:MI:SS'))
from ALI_CRM_INITDATA_VIP_VG



#### 会员手机号

select count(*)
from ALI_CRM_INITDATA_VIP_TW
where phone is null

select phone
from ALI_CRM_INITDATA_VIP_VG
where phone is null 

select *
from ALI_CRM_INITDATA_VIP_TW
where LENGTH(PHONE) <> 11

select ERPID,phone
from ALI_CRM_INITDATA_VIP_VG
where LENGTH(PHONE) <> 11

select * 
from ALI_CRM_INITDATA_VIP_VG
where phone in (
select phone 
from ALI_CRM_INITDATA_VIP_VG
group by phone 
having count(phone) > 1 
)

#### 卡等级

select *
from ALI_CRM_INITDATA_VIP_VG
where LEVELCODE is null 

select *
from ALI_CRM_INITDATA_VIP_TW
where LEVELCODE is null 



#### 开卡门店

select *
from ALI_CRM_INITDATA_VIP_VG
where OPENCARDSTORECODE is null 

select count(*)
from ALI_CRM_INITDATA_VIP_TW
where OPENCARDSTORECODE is null 

select * from ALI_CRM_INITDATA_VIP_VG
where OPENCARDSTORECODE in (
　SELECT OPENCARDSTORECODE FROM ALI_CRM_INITDATA_VIP_VG
　　MINUS  
　　SELECT storeCode FROM ALI_CRM_INITDATA_STORE_VG
);

select count(*) from ALI_CRM_INITDATA_VIP_VG
where OPENCARDSTORECODE  not in (
　　SELECT sys_store_offline_code FROM ALI_CRM_INITDATA_STORE_VG
);



#### 开卡导购

select *
from ALI_CRM_INITDATA_VIP_VG
where OPENCARDGUIDECODE is null 

select count(*)
from ALI_CRM_INITDATA_VIP_TW
where OPENCARDGUIDECODE is null 

select * from ALI_CRM_INITDATA_VIP_TW
where OPENCARDGUIDECODE  not in (
　　SELECT staffcode FROM ALI_CRM_INITDATA_EMP_TW
);

select count(*) from ALI_CRM_INITDATA_VIP_VG
where OPENCARDGUIDECODE  not in (
　　SELECT staffcode FROM ALI_CRM_INITDATA_EMP_VG
);

#### 服务门店

select *
from ALI_CRM_INITDATA_VIP_VG
where SERVICECARDSTORECODE is null 

select count(*)
from ALI_CRM_INITDATA_VIP_TW
where SERVICECARDSTORECODE is null 



#### 服务导购

select erpid
from ALI_CRM_INITDATA_VIP_VG
where SERVICEGUIDECODE is null



select erpid
from ALI_CRM_INITDATA_VIP_TW
where SERVICEGUIDECODE is null

--------------

### 订单数据

#### erpId两张表有差异的

select count(*) from ALI_CRM_INITDATA_ORDER_VG
where erpId not in (
　SELECT erpId FROM ALI_CRM_INITDATA_VIP_VG
);

select count(*) from ALI_CRM_INITDATA_ORDER_TW
where erpId not in (
　SELECT erpId FROM ALI_CRM_INITDATA_VIP_TW
);

#### 订单编号，订单明细编号

select orderno
from ALI_CRM_INITDATA_ORDER_VG
group by orderno 
having count(ORDERNO) > 1



select orderno
from ALI_CRM_INITDATA_ORDER_TW
group by orderno 
having count(ORDERNO) > 1;

#### 商品数量

select * 
from ALI_CRM_INITDATA_ORDER_VG a 
left join (select ORDERNO,sum(QUANTITY) QUANTITY from ALI_CRM_INITDATA_ORDERITEM_VG group by ORDERNO) b on a.ORDERNO = b.ORDERNO
where  PRODUCTCOUNT <> quantity;

select a.orderno, a.PRODUCTCOUNT, b.quantity
from ALI_CRM_INITDATA_ORDER_TW a 
left join (select ORDERNO,sum(QUANTITY) QUANTITY from ALI_CRM_INITDATA_ORDERITEM_TW group by ORDERNO) b on a.ORDERNO = b.ORDERNO
where  PRODUCTCOUNT <> quantity;



#### 所属店铺

select orderno, servicestorecode
from ALI_CRM_INITDATA_ORDER_VG
where servicestorecode not in (
　　SELECT sys_store_offline_Code FROM ALI_CRM_INITDATA_store_VG
);



#### 所属导购

select * from ALI_CRM_INITDATA_ORDER_VG
where OFFLINESERVICEGUIDECODE in (
　SELECT OFFLINESERVICEGUIDECODE FROM ALI_CRM_INITDATA_ORDER_VG
　　MINUS  
　　SELECT staffcode FROM ALI_CRM_INITDATA_EMP_VG
);



#### 导购业绩拆分比例

select * from ALI_CRM_INITDATA_ORDERITEM_VG
where service_guide_porprotion is null



select * from ALI_CRM_INITDATA_ORDERITEM_TW
where service_guide_porprotion is null



-----------------

### 会员积分不一致

select count(*)
from ALI_CRM_INITDATA_INTL_VG a 
inner join (
select erpId,sum(CHANGEINTEGRAL) integral 
from ALI_CRM_INITDATA_INTLFTP_VG 
group by ERPID) b on a.erpid = b.ERPID
where a.COUNTINTEGRAL <> b.INTEGRAL



select count(*)
from ALI_CRM_INITDATA_INTL_TW a 
inner join (
select erpId,sum(CHANGEINTEGRAL) integral 
from ALI_CRM_INITDATA_INTLFTP_TW 
group by ERPID) b on a.erpid = b.ERPID
where a.COUNTINTEGRAL <> b.INTEGRAL



#### 集团卡合并存储过程（不成立）

create or replace PROCEDURE combineMemberInfo() 
as
DECLARE
  cursor opencardCursor is select count(1) from xtwtest group by SUBSTR(opencardtime, 0, 10);
  pemp myemp%rowtype;
	i number;
begin 
	for my_cur in opencardCursor loop
		i = 1;
		for i in my_cur loop:
			update rowtype set "VG" || "opencardtime" || "i"
			
end

