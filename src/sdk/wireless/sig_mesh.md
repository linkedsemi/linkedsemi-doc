# sig mesh provee工作流程

## 1. sig mesh 初始化

```{mermaid}
	    sequenceDiagram
	    participant app as application
	    participant stack as stack layer
        
	    app ->> app: sys_init_app
	    app ->> stack: mesh_stack_data_bss_init
	    app ->> app: tinyfs_mkdir(refer module tinyfs)
	    app ->> app: user app code init
	    
	    rect rgb(135, 206, 235)
            app ->> stack:ble_init
            app ->> app:auto_check_unbind<br/>(fast power up and down 5 times to unbind the node)

            app ->> stack:dev_manager_init(dev_manager_callback)
            app ->> stack:gap_manager_init(gap_manager_callback)
            app ->> stack:gatt_manager_init(gatt_manager_callback)
	    end
	    
	    loop ble_loop
	        stack -->> +app: stack_init
	        app ->> -stack:ble_stack_cfg(user define mac addr)
	        note over app,stack:get mac address
	        stack -->> app:stack ready
	    end    
	    
	    rect rgb(189, 252, 201)
	 		opt mesh application
	 		activate app
	 		note over app,stack:Add feature for sig mesh
	 		app ->> +stack : dev_manager_prf_ls_sig_mesh_add
	 		deactivate app
	 		stack -->> -app:profile mesh added
	 		app ->> stack:prf_ls_sig_mesh_callback_init(mesh_manager_callback)
	 		note over app,stack:resgister all models
	 		app ->> stack:ls_sig_mesh_init(&model_env)
	 		end
	 	end 
	    
```

## 2. Register Model

```{mermaid}
    sequenceDiagram
        Title:  
        participant app as application
	    participant stack as stack layer
	    
	    note over app,stack:All models have been registered
		app ->> stack: ls_sig_mesh_init(&model_env)
		
		rect rgb(189, 252, 201)
            opt mesh_auto_prov(unprov)
            note left of app:App_defined:<br/>unicast_address<br/>group_address<br/>app_key<br/>net_key<br/>iv
            note left of app:App_defined:<br/>Server Model<br/>Client Model
            stack -->> app:MESH_ACTIVE_AUTO_PROV
            app ->> stack: ls_sig_mesh_auto_prov_handler
            end
	    end
	    
	    note left of app:Node_Get_Proved_State:<br/>UNPROVISIONED_KO<br/>PROVISIONING<br/>PROVISIONED_OK
	    stack -->> app: MESH_ACTIVE_STORAGE_LOAD
	    stack -->> app:MESH_ACTIVE_ENABLE
	    
	    
	    rect rgb(135, 206, 235)
	      par clear power up num
			  app ->> app : Clear number of the power up after 3 Seconds
		  and proving_param_req
		  note left of app:App_Set_Prov_Param:    <br/>devuuid/UriHash/OobInfo/PubKeyOob<br/>StaticOob/OutOobSize/InOobSize<br/>OutOobAction/InOobAction/Info
			  stack -->> app :MESH_GET_PROV_INFO
		  note over app,stack:The device waits to be provisioned
			  app ->> stack : set_prov_param
	      end
	    end
	  

```

## 3. Provisioning Procedures

```{mermaid}
	  sequenceDiagram
	    participant app as application
	    participant stack as stack layer
	    
	    note over app, stack:The device is provisioning
	    rect rgb(189, 252, 201)
	      opt No OOB
	    app ->> stack:MESH_GET_PROV_AUTH_INFO 
	    note left of app:App_defined: Auth_data
	    app ->> stack:set_prov_auth_info
	      end
	    end
	    
	    note right of stack:PROV_STARTED<br/>PROV_SUCCEED<br/>PROV_FAILED
	    stack -->> app:MESH_REPOPT_PROV_RESULT
	    note left of app:App_defined:app_store<br/> Provisoning result
	    
	    rect rgb(244, 164, 96)
            alt Prov_Successed
            note right of stack:all models local index<br/>app key local index
            stack -->> app:MESH_ACTIVE_REGISTER_MODEL
            note left of app:app_store:<br/>all models local index<br/> app key local index
            else Prov_Failed
            note over app,stack:The device is provisioning
            end
        end

	    
```

## 4. mesh manager callback

下面的时序图展示了mesh  callback回调函数中的各种event以及收到event后应用层所做的处理，这些event是通过回调函数从协议栈传上来的。图中从callback函数到stack layer的注释对应的是event在协议栈内部的处理程序，从application到callback的注释指的是该event发生时application部分所做的处理。



```{mermaid}
sequenceDiagram
    	participant app as application
	    participant cb as mesh manager callback
	    participant stack as stack layer
	note left of cb:(funtion of added mesh profile)
	app ->>cb:prf_ls_sig_mesh_callback_init(mesh_manager_callback)
	
    rect rgb(189, 252, 201)
        alt MESH_ACTIVE_ENABLE
        note over cb, stack:Enable mesh profile 
        note over app, cb:Set mesh timer
        cb -->> app: TIMER_Set(2, 3000)

        else MESH_ACTIVE_DISABLE
        note over cb, stack:Disable mesh profile
        note over app, cb: Clear all provisioned information to Ubind
        cb -->> app:  SIGMESH_UnbindAll()<br/>platform_reset(0)

        else MESH_ACTIVE_REGISTER_MODEL
        note over cb, stack:Register model instance
        note over app, cb: Save the returned local index of Register model
        cb -->> app: 

        else MESH_ACTIVE_MODEL_PUBLISH
        note over cb, stack:Publish new message 
        note over app, cb: Save the returned information of publish
        cb -->> app: 

        else MESH_ACTIVE_LPN_START
        note over cb, stack:Register a low-power feature node
        note over app, cb: Set low power properties (such as timeout, interval, previous_address, RX window)
        cb -->> app: 
        else MESH_ACTIVE_LPN_OFFER
        note over cb, stack:Indicate mesh Friend Offer reception
        note over app, cb: Report nearby friend-supporting attributes and select a friend node
        cb -->> app: 

        else MESH_ACTIVE_LPN_STATUS
        note over cb, stack:Indicate mesh Friendship status
        note over app, cb: Send a status message of LPN,such as Friendship status/address
        cb -->> app: 

        else MESH_ACTIVE_STORAGE_LOAD
        note over cb, stack:Load stored information 
        note over app, cb:Report whether the device was a node
        cb -->> app: 

        else MESH_GET_PROV_INFO
        note over cb, stack:Get provisioning information 
        note over app, cb:Set device properties (such as uuid, urihash, oob) 
        cb -->> app: 

        else MESH_GET_PROV_AUTH_INFO
        note over cb, stack:Get provisioning authentication information 
        note over app, cb:Set device authentication parameter
        cb -->> app: 

        else MESH_REPOPT_PROV_RESULT
        note over cb, stack:Report provisioning result 
        note over app, cb:Indicte provisioning state
        cb -->> app: 

        else MESH_ACCEPT_MODEL_INFO
        note over cb, stack:Accept vendor model information
        note over app, cb:Indicte vendor model information and send message to other nodes
        cb -->> app: 

        else MESH_STATE_UPD_IND
        note over cb, stack:Indicate to update model state
        note over app, cb:Update model state behavior
        cb -->> app: 

        else MESH_REPORT_TIMER_STATE
        note over cb, stack:Report mesh timer state
        note over app, cb:Perform power-on count clearing when the power-on timer timed out
        cb -->> app: 

        else MESH_ADV_REPORT
        note over cb, stack:Report advertisement message
        note over app, cb:Indicate advertisement information(such as device address, adv data)
        cb -->> app: 
        
            opt MESH_ACTIVE_AUTO_PROV
            note over cb, stack: Enable automatic provisioning 
            note over app, cb:Configure auto provisioning parameter and start auto Provisioning process
            cb -->> app: 
            end
        end
    end
    
    app ->> stack:ls_sig_mesh_init(&model_env)
```

**note:**上面的event顺序是按照程序中switch--case语句的出现顺序编写的，不表示event发生的先后顺序，图中的 **alt (alternative )** 也说明了这一点。

### 4.1 广播和mesh profile

MESH_ADV_REPORT是Scan中的event, 在mesh和ble应用中均存在，当扫描到广播后会触发这个事件，应用程序可以获取adv_report_evt结构体中的Advertising address、adv data、rssi等信息。由于扫描到ble和mesh广播后都会进入这个事件，同时空间中可能存在大量的ble和mesh设备，这个事件触发会很频繁。如果直接获取adv_report_evt中的信息很可能得不到正确的信息，这是由于大量的信息不断进来导致之前的信息被覆盖，因此需要对广播信息进行一些过滤，例程中使用两个字节的MAC地址进行了过滤。

```c
case MESH_ADV_REPORT:
    {
        // if(evt->adv_report.adv_addr->addr[5] == 0x20 && evt->adv_report.adv_addr->addr[4] == 0x17)
        // {
        //     LOG_I("dev addr: %02x:%02x:%02x:%02x:%02x:%02x",evt->adv_report.adv_addr->addr[5],
        //                                                     evt->adv_report.adv_addr->addr[4],
        //                                                     evt->adv_report.adv_addr->addr[3],
        //                                                     evt->adv_report.adv_addr->addr[2],
        //                                                     evt->adv_report.adv_addr->addr[1],
        //                                                     evt->adv_report.adv_addr->addr[0]);
        //     //LOG_HEX(evt->adv_report.adv_addr.addr,6);
        //     LOG_HEX(evt->adv_report.data,evt->adv_report.length);
        // }
        
    }
break;
```

### 4.2  节点解绑

#### 4.2.1  定时累计上电次数触发解绑

MESH_ACTIVE_ENABLE发生后，应用程序使用TIMER_Set设置了一个3秒的定时器，它与MESH_REPORT_TIMER_STATE相关。3秒为timeout时间，超过3秒将触发MESH_REPORT_TIMER_STATE事件，会对clear_power_on_num清零。

```c
 case MESH_ACTIVE_ENABLE:
    {
        TIMER_Set(2, 3000); //clear power up num
    }
 break;
```

```c
 case MESH_REPORT_TIMER_STATE:
    {
        if (2 == evt->mesh_timer_state.timer_id)
        {
            uint8_t clear_power_on_num = 0;
            TIMER_Cancel(2);
            tinyfs_write(ls_sigmesh_dir, RECORD_KEY1, &clear_power_on_num, sizeof(clear_power_on_num));
            tinyfs_write_through();
        }
    }
    break;
```

例程中提供了一种解绑方式（可查看auto_check_unbind函数）：在3秒内节点累计大于4次快速上下电，node重新变为device(未入网的设备称为device)；超过3秒节点仍未完成累计大于4次快速上下电，则认为节点不需要解除绑定，会将coutinu_power_up_num清零（这里的coutinu_power_up_num和之前的clear_power_on_num实际上是同一个值，在tinyfs中使用同一内存地址），用户可以修改应用程序实现其他解绑方式。

```c
void auto_check_unbind(void)
{
    uint16_t length = 1;
    uint8_t coutinu_power_up_num = 0;
    tinyfs_read(ls_sigmesh_dir, RECORD_KEY1, &coutinu_power_up_num, &length);
    LOG_I("coutinu_power_up_num:%d", coutinu_power_up_num);

    if (coutinu_power_up_num > 4)
    {
        coutinu_power_up_num = 0;
        tinyfs_write(ls_sigmesh_dir, RECORD_KEY1, &coutinu_power_up_num, sizeof(coutinu_power_up_num));
        tinyfs_write_through();
        SIGMESH_UnbindAll();
    }
    else
    {
        coutinu_power_up_num++;
        tinyfs_write(ls_sigmesh_dir, RECORD_KEY1, &coutinu_power_up_num, sizeof(coutinu_power_up_num));
        tinyfs_write_through();
    }
}
```

#### 4.2.2  Mesh协议执行复位指令触发解绑

MESH_ACTIVE_DISABLE表示mesh proflie disable,触发后应用程序会调用SIGMESH_UnbindAll清除该节点所有的provisioned information，node重新变为device。用户可以修改应用程序在MESH_ACTIVE_DISABLE出发后不执行解绑操作，实现其他逻辑。

```c
 case MESH_ACTIVE_DISABLE:
    {
        SIGMESH_UnbindAll();
        platform_reset(0);
    }
 break;
```

### 4.3.友邻节点

MESH_ACTIVE_LPN_START、MESH_ACTIVE_LPN_OFFER、MESH_ACTIVE_LPN_STATUS三个事件是low power node和friend node相关的特性，如果该节点不是low power node，则不会触发这些事件。

```c
 case MESH_ACTIVE_LPN_START:
    {
        struct start_lpn_info param;
        param.poll_timeout_100ms = 20; //timeout min 1s
        param.poll_intv_ms = 1000;
        param.previous_addr = 0x2013;
        param.rx_delay_ms = 10; //rx delay min 10ms
        param.rx_window_factor = LPN_RX_WINDOW_FACTOR_2_5;
        param.min_queue_size_log = FRIEND_NODE_MIN_QUEUE_SIZE_LOG_N16;
        start_lpn_handler(&param);
    }
    break;
    case MESH_ACTIVE_LPN_OFFER:
    {
        friendship_env.friend_addr = evt->lpn_offer_info.friend_addr;
        friendship_env.friend_rx_window = evt->lpn_offer_info.friend_rx_window;
        friendship_env.friend_queue_size = evt->lpn_offer_info.friend_queue_size;
        friendship_env.friend_subs_list_size = evt->lpn_offer_info.friend_subs_list_size;
        friendship_env.friend_rssi = evt->lpn_offer_info.friend_rssi;
        lnp_select_friend_handler(friendship_env.friend_addr);
    }
    break;
    case MESH_ACTIVE_LPN_STATUS:
    {
        local_lpn_env.lpn_status = evt->lpn_status_info.lpn_status;
        local_lpn_env.friend_addr = evt->lpn_status_info.friend_addr;
    }
    break;
```

当low power node被注册后会触发MESH_ACTIVE_LPN_START事件，然后应用程序设置实现low power所需的参数；附近的friend node若支持low power参数中特定的要求，将发送“Friend Offer”消息给LPN，此时将触发MESH_ACTIVE_LPN_OFFER事件，low power node收到Friend Offer消息后将根据friend_addr选择一个friend节点；然后在MESH_ACTIVE_LPN_STATUS事件发生后，在应用程序中发送LPN status message。

下图展示了Friendship establish过程：

```{mermaid}
 sequenceDiagram
	    participant lpn as Low Power node
	    participant fdn as Friend node
	   
	    lpn ->> fdn: Friend Request
	    fdn ->> lpn: Friend Offer
	    lpn ->> fdn:Friend Poll
	    fdn ->> lpn:Friend Update
	    note over lpn,fdn:Friendship established
          
```



### 4.4. 入网配置信息

 在provisioning时会触发MESH_GET_PROV_INFO事件，应用程序将设置设备配网所需的参数；双方交换publish key后会来到Authentication阶段，此时会触发MESH_GET_PROV_AUTH_INFO事件，应用程序将设置Authentication所需的参数，之后会在MESH_REPOPT_PROV_RESULT中指示配网是否成功。

```c
    case MESH_GET_PROV_INFO:
    {
        struct mesh_prov_info param;
        memcpy(&param.DevUuid[0],&dev_uuid[0],16);
        param.UriHash = 0x00000000;
        param.OobInfo = 0x0000;
        param.PubKeyOob = 0x00;
        param.StaticOob = 0x00;
        param.OutOobSize = 0x00;
        param.InOobSize = 0x00;
        param.OutOobAction = 0x0000;
        param.InOobAction = 0x0000;
        param.Info = 0x00;
        set_prov_param(&param);
    }
    break;
    case MESH_GET_PROV_AUTH_INFO:
    {
        struct mesh_prov_auth_info param;
        param.Adopt = PROV_AUTH_ACCEPT;
        memcpy(&param.AuthBuffer[0], &auth_data[0], 16);
        param.AuthSize = 16;
        set_prov_auth_info(&param);
    }
    break;
    case MESH_REPOPT_PROV_RESULT:
    {
        if(evt->prov_rslt_sate.state == MESH_PROV_STARTED)
        {
            LOG_I("prov started");
        }
        else if(evt->prov_rslt_sate.state == MESH_PROV_SUCCEED)
        {
            LOG_I("prov succeed");
            mesh_node_prov_state = true;
            ls_mesh_light_set_lightness(0xffff,LIGHT_LED_2);
            ls_mesh_light_set_lightness(0xffff,LIGHT_LED_3);
        }
       else if(evt->prov_rslt_sate.state == MESH_PROV_FAILED)
        {
            LOG_I("prov failled:%d",evt->prov_rslt_sate.status);
        }
    }
    break;
```

### 4.5  上电加载配置信息

每次设备上电或复位后会触发MESH_ACTIVE_STORAGE_LOAD，应用程序从flash中加载信息判断设备的状态。

```c
 case MESH_ACTIVE_STORAGE_LOAD:
    {
        Node_Get_Proved_State = evt->st_proved.proved_state;
        if (Node_Get_Proved_State == PROVISIONED_OK)
        {
            uint16_t length = sizeof(provisioner_unicast_addr);
            LOG_I("The node is provisioned");
            ls_mesh_light_set_lightness(0xffff, LIGHT_LED_2);
            ls_mesh_light_set_lightness(0xffff, LIGHT_LED_3);
            tinyfs_read(ls_sigmesh_dir, RECORD_KEY2, (uint8_t *)&provisioner_unicast_addr, &length);
            mesh_node_prov_state = true;
        }
        else
        {
            LOG_I("The node is not provisioned");
            mesh_node_prov_state = false;
        }
        
    }
    break;
```

### 4.6 Mesh应用层消息处理

#### 4.6.1 自定义Vendor Model交互信息

当节点中所有model都注册完成后会触发MESH_ACTIVE_REGISTER_MODEL事件，应用程序将保存返回的model 和 app key的local index。当client model使能了publish funtion后将触发MESH_ACTIVE_MODEL_PUBLISH事件，应用程序将保存返回的publish信息。   收到来自vendor model的消息后会触发MESH_ACCEPT_MODEL_INFO事件，应用程序将指示model information并发送message给source节点。

```c
  case MESH_ACCEPT_MODEL_INFO:
    {
         LOG_I("vendor info");
         LOG_I("model_lid=%x,opcode=%x",evt->rx_msg.ModelHandle,evt->rx_msg.opcode);
         LOG_HEX(&evt->rx_msg.info[0],evt->rx_msg.rx_info_len);
        if (evt->rx_msg.opcode == APP_LS_SIG_MESH_VENDOR_SET)
         {

            ls_mesh_light_set_onoff(evt->rx_msg.info[0], LIGHT_LED_1); 

            app_generic_vendor_report(evt->rx_msg.source_addr);
         } 
    }
    break;
```

#### 4.6.2 标准Sig Model的状态信息

当model状态需要更新时将触发MESH_STATE_UPD_IND事件，应用程序将根据具体的state id来设置state。在MESH_STATE_UPD_IND处理SIG Mesh中标准model的message,上面的MESH_ACCEPT_MODEL_INFO处理vendor model的message。

```c
 case MESH_STATE_UPD_IND:
    {
        sig_mesh_mdl_state_upd_hdl(&evt->mdl_state_upd_ind);
    }
    break;
```

### 4.7 自动入网模式的节点配置

使用自动配网例程sig_mesh_provee_auto_prov时，mesh manager callback增加了一个MESH_ACTIVE_AUTO_PROV事件，设备上电后会从flash中加载信息，如果设备处于unprovisioned状态将触发这个事件。应用程序将设置配网所需的参数，如单播地址、组播地址、model参数以及app key和net key等,然后开始自动配网处理。

```c
     case MESH_ACTIVE_AUTO_PROV:
    {
            struct mesh_auto_prov_info init_param;
            init_param.model_nb = model_env.nb_model;
			memcpy((uint8_t *)&init_param.unicast_addr,&dev_uuid[0],2); 
            init_param.unicast_addr = NODE_UNICAST_ADDR; // unicast_addr >= 0x0002
            init_param.group_addr = GROUP_ADDR;//+group_para_handle();   
                                              // group_addr >=  0xC000
            init_param.ttl = PUBLISH_TTL;
            init_param.model_info[MODEL0_GENERIC_ONOFF_SVC].model_id = 
            model_env.info[MODEL0_GENERIC_ONOFF_SVC].model_id;
            init_param.model_info[MODEL0_GENERIC_ONOFF_SVC].publish_flag = false;
            init_param.model_info[MODEL0_GENERIC_ONOFF_SVC].subs_flag = true;

            init_param.model_info[MODEL1_GENERIC_LEVEL_SVC].model_id =        
            model_env.info[MODEL1_GENERIC_LEVEL_SVC].model_id;
            init_param.model_info[MODEL1_GENERIC_LEVEL_SVC].publish_flag = false;
            init_param.model_info[MODEL1_GENERIC_LEVEL_SVC].subs_flag = true;

            init_param.model_info[MODEL2_VENDOR_MODEL_SVC].model_id = 
            model_env.info[MODEL2_VENDOR_MODEL_SVC].model_id;
            init_param.model_info[MODEL2_VENDOR_MODEL_SVC].publish_flag = false;
            init_param.model_info[MODEL2_VENDOR_MODEL_SVC].subs_flag = true;

            init_param.model_info[MODEL0_GENERIC_ONOFF_CLI].model_id = 
            model_env.info[MODEL0_GENERIC_ONOFF_CLI].model_id;
            init_param.model_info[MODEL0_GENERIC_ONOFF_CLI].publish_flag = true;
            init_param.model_info[MODEL0_GENERIC_ONOFF_CLI].subs_flag = true;

            init_param.model_info[MODEL1_GENERIC_LEVEL_CLI].model_id = 
            model_env.info[MODEL1_GENERIC_LEVEL_CLI].model_id;
            init_param.model_info[MODEL1_GENERIC_LEVEL_CLI].publish_flag = true;
            init_param.model_info[MODEL1_GENERIC_LEVEL_CLI].subs_flag = true;

            init_param.model_info[MODEL2_VENDOR_MODEL_CLI].model_id = 
            model_env.info[MODEL2_VENDOR_MODEL_CLI].model_id;
            init_param.model_info[MODEL2_VENDOR_MODEL_CLI].publish_flag = true;
            init_param.model_info[MODEL2_VENDOR_MODEL_CLI].subs_flag = true;
            memcpy(&init_param.app_key[0], &app_key[0], 16);
            memcpy(&init_param.net_key[0], &net_key[0], 16);

            ls_sig_mesh_auto_prov_handler(&init_param, true);
    }
    break;
```

