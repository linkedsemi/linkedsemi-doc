# Mesh Provisioner指令

指令格式：

AT+“操作符” = “参数1,参数2,参数3...”

## 1.1 启动扫描

| 操作符             | 参数1（s）     |
| ------------------ | -------------- |
| START_SCAN         | LSB-MSB        |
| 示例               |                |
| AT+START_SCAN=0500 | 扫描持续时间5s |

## 1.2 停止扫描

| 操作符       |
| ------------ |
| STOP_SCAN    |
| 示例         |
| AT+STOP_SCAN |

## 2.1 发起入网

| 操作符                                                | 参数1（节点主单播地址） | 参数2（UUID）                                |
| ----------------------------------------------------- | ----------------------- | -------------------------------------------- |
| START_INVITE                                          | LSB-MSB                 | LSB-MSB                                      |
| 示例                                                  |                         |                                              |
| AT+START_INVITE=0600,1111111111C08F45B2DF93EE71127C68 | Unicast_Address= 0x0006 | DEV_UUID =0x687c1271ee93dfb2458fc01111111111 |

* 注： 节点的主单播地址应该大于0x0001，其中Provisioner的单播地址为0x0001。

## 2.2 配置节点

| 操作符                           | 参数1（单播地址）       | 参数2（netkey_lid） | 参数3(devkey_lid) |
| -------------------------------- | ----------------------- | ------------------- | ----------------- |
| SET_CONFIG_DEV                   | LSB-MSB                 | LSB-MSB             | LSB-MSB           |
| 示例                             |                         |                     |                   |
| AT+SET_CONFIG_DEV=0400,2000,0300 | Unicast_Address= 0x0004 | netkey_lid = 0x0020 | devkey_lid=0x0003 |

* 注：netkey_lid和devkey_lid是节点在入网时，由Mesh协议分配的lid

## 2.3 解绑节点

| 操作符          |
| --------------- |
| SET_RST_NODE    |
| 示例            |
| AT+SET_RST_NODE |

* 注：解绑一个节点，Provisioner端需要先设定要解绑设备的单播地址、网络密钥协议栈分配的ID号以及节点的设备密钥协议栈分配的ID号。所以在解绑节点操作前，先执行配置节点操作。

## 3.1 节点Mesh模型绑定APP密钥

| 操作符                             | 参数1（元素的单播地址）         | 参数2（appkey_id） | 参数3(Model_id)     |
| ---------------------------------- | ------------------------------- | ------------------ | ------------------- |
| BIND_MDL_APP                       | LSB-MSB                         | LSB-MSB            | LSB-MSB             |
| 示例                               |                                 |                    |                     |
| AT+BIND_MDL_APP=0200,0000,00100000 | Element_Unicast_Address= 0x0002 | appkey_id = 0x0000 | Model_id=0x00001000 |

* 注：对于每个节点(Node)，可以有多个元素(Element)，每个元素可以有多个模型(Model)
* 注：appkey_id和model_id都是应用层分配给Mesh协议的两个参数。其中model_id可以是标准的sig Model也可以是用户自定义的Vendor Model

## 3.2 节点Mesh模型解绑APP密钥

| 操作符                               | 参数1（目标单播地址）   | 参数2（Appkey_id） | 参数3(Model_id)     |
| ------------------------------------ | ----------------------- | ------------------ | ------------------- |
| UNBIND_MDL_APP                       | LSB-MSB                 | LSB-MSB            | LSB-MSB             |
| 示例                                 |                         |                    |                     |
| AT+UNBIND_MDL_APP=0200,0000,00100000 | Unicast_Address= 0x0002 | appkey_id= 0x0000  | Model_id=0x00001000 |

## 4.1 节点Mesh服务端模型订阅组播地址

| 操作符                                | 参数1（目标单播地址）   | 参数2（Model_id）   | 参数3（订阅地址类型） | 参数4(Group_address) |
| ------------------------------------- | ----------------------- | ------------------- | --------------------- | -------------------- |
| ACT_MDL_SUBS                          | LSB-MSB                 | LSB-MSB             | LSB-MSB               | LSB-MSB              |
| 示例                                  |                         |                     |                       |                      |
| AT+ACT_MDL_SUBS=0200,00100000,00,00C0 | Unicast_Address= 0x0002 | Model_id=0x00001000 | addr_type=0x00        | Group_address=0xC000 |

* 注：订阅地址类型，0x00表示订阅地址为组播地址，0x01表示订阅地址为128bits的虚拟地址

## 4.2 节点Mesh客户端模型发布组播地址

| 操作符                                                   | 参数1               （目标单播地址） | 参数2 （Appkey_id） | 参数3 （Publish_ttl） | 参数4 (Publish_period) | 参数5 （Retx_cnt)） | 参数6 （Retx_intv） | 参数7 （Model_id)）  | 参数8    （发布地址类型） | 参数9             (发布组播地址） |
| -------------------------------------------------------- | ------------------------------------ | ------------------- | --------------------- | ---------------------- | ------------------- | ------------------- | -------------------- | ------------------------- | --------------------------------- |
| SET_MDL_PUBLI                                            | LSB-MSB                              | LSB-MSB             | LSB-MSB               | LSB-MSB                | LSB-MSB             | LSB-MSB             | LSB-MSB              | LSB-MSB                   | LSB-MSB                           |
| 示例                                                     |                                      |                     |                       |                        |                     |                     |                      |                           |                                   |
| AT+SET_MDL_PUBLI= 0200,0000,0b,00,00,00,01100000,00,00C0 | Unicast_Address= 0x0002              | Appkey_id= 0x0000   | Publish_ttl= 0x0b     | Publish_period= 0x00   | Retx_cnt=  0x00     | Retx_intv= 0x00     | Model_id= 0x00001001 | addr_type= 0x00           | Publish_addr= 0xC000              |

## 5.1 Provisioner 用OnOff 模型控制单节点On/Off操作

| 操作符                                  | 参数1（控制操作） | 参数2（目标单播地址）   | 参数3（Model_id）   |
| --------------------------------------- | ----------------- | ----------------------- | ------------------- |
| ONOFF_MDL_APP                           | LSB-MSB           | LSB-MSB                 | LSB-MSB             |
| 示例                                    |                   |                         |                     |
| AT+ONOFF_MDL_APP=00000000,0200,01100000 | 0x00000000（关）  | Unicast_Address= 0x0002 | Model_id=0x00001001 |
| AT+ONOFF_MDL_APP=01000000,0200,01100000 | 0x000000001 (开)  | Unicast_Address= 0x0002 | Model_id=0x00001001 |

## 5.2 Provisioner 用OnOff 模型群控节点On/Off操作

| 操作符                                  | 参数1（控制操作） | 参数2（目标组播地址）   | 参数3（Model_id）   |
| --------------------------------------- | ----------------- | ----------------------- | ------------------- |
| ONOFF_MDL_APP                           | LSB-MSB           | LSB-MSB                 | LSB-MSB             |
| 示例                                    |                   |                         |                     |
| AT+ONOFF_MDL_APP=00000000,00C0,01100000 | 0x00000000（关）  | Unicast_Address= 0xC000 | Model_id=0x00001001 |
| AT+ONOFF_MDL_APP=01000000,00C0,01100000 | 0x000000001 (开)  | Unicast_Address= 0xC000 | Model_id=0x00001001 |

## 5. 3 Provisioner 用Vendor 模型传输信息给节点

| 操作符                                                  | 参数1   （Vendor_model_id） | 参数2               （目标单播地址） | 参数3 （Vendor_opcode） | 参数4        (消息长度） | 参数5（消息） |
| ------------------------------------------------------- | --------------------------- | ------------------------------------ | ----------------------- | ------------------------ | ------------- |
| VDR_MDL_TX_APP                                          | LSB-MSB                     | LSB-MSB                              | LSB-MSB                 | LSB-MSB                  | LSB-MSB       |
| 示例                                                    |                             |                                      |                         |                          |               |
| AT+VDR_MDL_TX_APP= 02003a09,0400,d03a0900,05,0102030405 | Model_id= 0x093a0002        | Unicast_Address= 0x0004              | Opcode= 0x00093ad0      | length=0x05              | 0x0504030201  |

## 5.4  Provisioner 用Vendor 模型传输信息给组节点

| 操作符                                                  | 参数1   （Vendor_model_id） | 参数2               （目标单播地址） | 参数3（Vendor_opcode） | 参数4        (消息长度） | 参数5（消息） |
| ------------------------------------------------------- | --------------------------- | ------------------------------------ | ---------------------- | ------------------------ | ------------- |
| VDR_MDL_TX_APP                                          | LSB-MSB                     | LSB-MSB                              | LSB-MSB                | LSB-MSB                  | LSB-MSB       |
| 示例                                                    |                             |                                      |                        |                          |               |
| AT+VDR_MDL_TX_APP= 02003a09,01c0,d03a0900,05,0102030405 | Model_id= 0x093a0002        | Group_Address= 0xc001                | Opcode=  0x00093ad0    | length=0x05              | 0x0504030201  |

