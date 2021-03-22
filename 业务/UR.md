UR

审核账号13122474147

密码 1234abcd







- refundRechargeCard 退款申请接口
  1. 调之前必须先修改卡的状态， 编程card_status = 5 退款申请中
  2. 写表refund_record 退款记录表的时候会把审核人也写进去，examine_id, examine_name
  3. 如果走页面退款流程，
- modifyRechargeCardOrderNo 修改流水号接口
  1. 查t_cus_ur_recharge_card_record表，order_no，record_no确认唯一记录
  2. 如果线上存在订单号，线下没有订单号，往线下传两条记录，orderNo, newOrderNo, 产生两条记录
  3. 如果线上存在订单号，线下有老的订单号，只往线下写1条新的记录
- modifyRefundStatus 退款申请状态接口
  1. 修改状态 正常 1 -> 编程退款申请中 0， 改表里两个字段card_status, refund_status
  2. 连接器写线下修改卡状态不可用
  3. 退款申请中时，可以查询卡状态，可调整卡金额，不能消费，不能充钱

