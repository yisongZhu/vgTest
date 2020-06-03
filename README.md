# tspvg
vehiclegateway车辆监控网关

## 异常处理策略
* 未鉴权出现协议解析异常断开网络连接
* 未鉴权连接生命时长10s，tbox须在建立连接10s内完成鉴权
* 已鉴权出现协议解析失败，进行负反馈
* 已鉴权负反馈类型
  * 报文转码异常 binary->ascii
  * 报文解密异常
  * 报文长度错误
  * 未知的设备类型
  * 报文验签失败
  * 鉴权失败
    * 车辆未开通blink
    * 车辆imei、vin码不符
* 已鉴权连接生命周期70s，70s收不到正常的上行报文则断开连接

## 系统优化策略
* 重新定义网络通讯框架，按照功能进行分类设计
* 统一异常和负反馈处理，详见异常处理
* 关键网络通讯环节加入application事件触发，业务监听事件进行逻辑处理
* rmi远程服务接口采用方法回调方式来实现异步通知调用方
* 下行指令分级，写指令同一时刻进行互斥处理，需等待tbox ack，读指令可无ack下发，为蓝牙钥匙和ota升级做技术铺垫
* 删除非必要的定时器，交由netty task处理tcp会话超时和下发指令长时间未响应逻辑
* tbox在线状态、rmi服务路由改用redis进行维持
* 高度抽象业务功能性代码，业务实现类只关注业务自身逻辑
* 自定义二进制协议模版解析引擎，解决大部分协议编解码功能逻辑
* 业务实现优化，修该部分数据结构和服务接口结构
* 消息传递去掉现有的rest服务调用，改用mq减少服务依赖
* 加入交互报文日志的hbase存储
* 使用统一的缓存框架解决数据服务的数据缓存问题，减少功能逻辑侵入

## 待改进
* 标准数据结构输出
* 标准化错误码
* TCP服务摘除策略
* 下行服务同步策略，减少polling服务依赖
* session缓存
* 异步队列改用redis
* 设备GPS时间错误处理，由下位机实现较为合理
  * GPS时间翻转鉴别规则 if(abs(平台时间-GPS时间)>1y) gps发生翻转
  * 设备时间修正 平台时间(yyyy-mm-dd)+GPS时间(hh:MM:dd)
  * 修正时间0点问题 if(hour==23 && abs(平台时间-修正时间)>22h) 修正时间=修正时间-1d, if(hour==0 && abs(平台时间-修正时间)>22h) 修正时间=修正时间+1d
* tbox更换服务地址策略
  * tbox固化ip:port（缺省服务地址），ota协议增加配置协议用于平台可以下发服务端ip:port;tbox优先使用配置的地址，如果出现3次conn失败则切换使用固化的地址
  * tbox固化ip:port（导航服务地址），ota协议增加获取平台服务地址协议，tbox每次上电主动通过固化的ip:port进行导航地址查询
  * ota协议加入短信白名单设置，tbox收到短信验证发件人是否在白名单，进行tbox设置
  
## 分支管理
* master 主分支
* dev_store_v1.0.0 向tspvehstore传输serviceData, 数据分类落库hbase,热点数据落库mongo
* dev_ota_v1.0.1 ota升级通道1901、6902
* dev_mcu_v1.0.2 customer系统存储3805中的mcu、mpu、ble等版本信息
* dev_charge_v1.0.3 预约充电需求
* dev_big_data_v1.0.4 大数据上报
* dev_empty_fix_v1.0.5 store空数据修复
* dev_door_fix_v1.0.6 t-box端6304 阻塞 （vgclient - 1.1.3-SNAPSHOT）
* dev_mq_order_dtc_current_v1.0.7 dove分区调整保证同车顺序、dtc 判断当前故障调整
* dev_multi_1304_v1.0.8 多条6304回复1条1304 需求
* dev_ff_v1.0.9 公式ff无效数据，不参与计算修改; dk topic新增imei;
* dev_ota_and_ff_v1.1.0 ota与模板0xff的分支

## tag管理
* tspvg_tag_v1.0.0 vg重构
* tspvg_tag_v1.0.1 重构后，兼容6311 eventId为0、1002变长问题
* tspvg_tag_v1.0.2 新能源监控系统发serviceData、online topic添加能源类型、driveaction topic过滤电车
* tspvg_tag_v1.0.3 车控'无钥匙驾驶'client包 bug fix，更新tukey 修改
* tspvg_tag_v1.0.4 MCU大版本，熄火上传
* tspvg_tag_v1.0.5 振铃后，0x1001与0x1301下行指令冲突bug修复
* tspvg_tag_v1.0.6 t-box端6304 阻塞 单独下行指令 1304
* tspvg_tag_v1.0.7 电动车-充电 需求
* tspvg_tag_v1.0.8 mq同一vin同分区顺序消费、1d00覆盖6d00、dtc当前故障规则调整