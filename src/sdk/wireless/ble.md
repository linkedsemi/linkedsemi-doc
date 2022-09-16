# BLE
## 设备初始化
```{mermaid} 
sequenceDiagram
title: BLE DEVICE SETUP TYPICAL MESSAGE FLOW
App->>Stack: dev_manager_init
App->>Stack: gap_manager_init
App->>Stack: gatt_manager_init
Stack-->>App: dev event: STACK_INIT
App->>+Stack: dev_manager_stack_init
Stack-->>-App: dev event:STACK_READY

opt ADD SERVICE
App->>+Stack:dev_manager_add_service   
Stack-->>-App: dev_event: SERVICE_ADDED
App->>Stack: gatt_manager_svc_register
end 

opt ADD PROFILE
App->>+Stack: dev_manager_prf_xxx_add
Stack-->>-App: dev_event: PROFILE_ADDED
App->>Stack: prf_xxx_callback_init
end

note over App,Stack: DEVICE INITIALIZED
```
## GAP
```{mermaid}
sequenceDiagram 
title: BLE GAP TYPICAL MESSAGE FLOW
participant Slave App
participant  Slave Stack
participant  Master Stack
participant Master App
note over  Slave App,Master App:DEVICE INITIALIZED
Slave App->>+Slave Stack:dev_manager_create_legacy_adv_object
Slave Stack-->>-Slave App:dev event: ADV_OBJ_CREATED
Slave App->>+Slave Stack:dev_manager_start_adv

opt SCAN
    Master App->>+Master Stack:dev_manager_create_scan_object
    Master Stack-->>-Master App:dev event: SCAN_OBJ_CREATED
    Master App->>+Master Stack:dev_manager_start_scan

    loop several times
        Master Stack-->>Master App:dev event: ADV_REPORT
    end

    opt STOP SCAN MANUALLY
        Master App->>Master Stack:dev_manager_stop_scan
    end
    Master Stack-->>-Master App: dev event: SCAN_STOPPED
end 

Master App->>+Master Stack:dev_manager_create_init_object
Master Stack-->>-Master App: dev event: INIT_OBJ_CREATED
Master App->>+Master Stack:dev_manager_start_init

Master Stack->>Slave Stack:CONNECT REQUEST
note over Slave App,Master App:CONNECTION ESTABLISHED
Master Stack-->>-Master App: dev event: INIT_STOPPED
Slave Stack-->>-Slave App:dev event: ADV_STOPPED
Master Stack-->>Master App:gap event: CONNECTED
Slave Stack-->>Slave App:gap event: CONNECTED

opt BOND PROCEDURE

    opt SLAVE INITIATE SECURITY PROCEDURE
        Slave App->>Slave Stack:gap_manager_slave_security_req
        Slave Stack->>Master Stack:SECURITY REQUEST
        Master Stack-->>Master App:gap event: SLAVE_SECURITY_REQ
    end
    Master App->>Master Stack:gap_manager_master_bond
    Master Stack->>Slave Stack:PAIR REQUEST
    Slave Stack-->>Slave App:gap event: MASTER_PAIR_REQ
    Slave App->>Slave Stack:gap_manager_slave_pair_response_send
    Slave Stack->>Master Stack:PAIR RESPONSE

    alt Just Works
    else Passkey Entry :responder displays,initiator inputs
        Slave Stack-->>Slave App:gap event: DISPLAY_PASSKEY
        Master Stack-->>+Master App:gap event: REQUEST_PASSKEY
        Master App->>-Master Stack:gap_manager_passkey_input
    else Passkey Entry:initiator displays,responder inputs
        Master Stack-->>Master App:gap event: DISPLAY_PASSKEY
        Slave Stack-->>+Slave App:gap event: REQUEST_PASSKEY
        Slave App->>-Slave Stack:gap_manager_passkey_input
    else Passkey Entry:initiator and responder inputs
        Slave Stack-->>+Slave App:gap event: REQUEST_PASSKEY
        Master Stack-->>+Master App:gap event: REQUEST_PASSKEY
        Slave App->>-Slave Stack:gap_manager_passkey_input
        Master App->>-Master Stack:gap_manager_passkey_input
    else Numeric Comparison
        Slave Stack-->>+Slave App:gap event:NUMERIC_COMPARE
        Master Stack-->>+Master App:gap event:NUMERIC_COMPARE
        Slave App->>-Slave Stack:gap_manager_numeric_compare_set
        Master App->>-Master Stack:gap_manager_numeric_compare_set
    else OOB (Legacy)
        Slave Stack-->>+Slave App:gap event: Legacy OOB
        Master Stack-->>+Master App:gap event: Legacy OOB
        Slave App->>-Slave Stack: gap_manager_TK_set
        Master App->>-Master Stack: gap_manager_TK_set
    else OOB (Secure Connection)
        Slave Stack-->>+Slave App:gap event:  Secure OOB
        Master Stack-->>+Master App:gap event: Secure OOB
        Slave App->>-Slave Stack: gap_manager_sc_oob_set(Master Confirmation code,Master rand code)
        Master App->>-Master Stack: gap_manager_sc_oob_set(Slave Confirmation code,Slave rand code)
    end

    note over Slave Stack,Master Stack: Authentication and Encryption
    Slave Stack-->>Slave App:gap event: PAIR_DONE
    Master Stack-->>Master App: PAIR_DONE
end 

Master App->>Master Stack:gap_manager_disconnect
Master Stack->>Slave Stack:DISCONNECT
note over Slave App,Master App:DISCONNECTION
Master Stack-->>Master App:gap event:DISCONNECTED
Slave Stack-->>Slave App:gap event:DISCONNECTED
Slave App->>+Slave Stack:dev_manager_start_adv
Master App->>+Master Stack:dev_manager_start_init
Master Stack->>Slave Stack:CONNECT REQUEST
note over Slave App,Master App:CONNECTION ESTABLISHED
Master Stack-->>-Master App:dev event: INIT_STOPPED
Slave Stack-->>-Slave App:dev event: ADV_STOPPED
Master Stack-->>Master App:gap event: CONNECTED
Slave Stack-->>Slave App:gap event: CONNECTED

opt ENCRYPT PROCEDURE

    opt SLAVE INITIAE SECURITY PROCEDURE
        Slave App->>Slave Stack:gap_manager_slave_security_req
        Slave Stack->>Master Stack:SECURITY REQUEST
        Master Stack-->>Master App:gap event: SLAVE_SECURITY_REQ
    end

    Master Stack->>Slave Stack:ENCRYPTION REQUEST
    Slave Stack->>Master Stack:ENCRYPTION RESPONSE
    Master Stack->>Slave Stack:START ENCRYPTION REQUEST
    Slave Stack->>Master Stack:START ENCRYPTION RESPONSE
    Slave Stack-->>Slave App:gap event: ENCRYPT_DONE
    Master Stack-->>Master App:gap event:ENCRYPT_DONE
end

Master App->>Master Stack:gap_manager_disconnnect
Master Stack->>Slave Stack:DISCONNECT
note over Slave App,Master App:DISCONNECTION
Master Stack-->>Master App:gap event: DISCONNECTED
Slave Stack-->>Slave App:gap event: DISCONNECTED
```
## GATT
```{mermaid}
sequenceDiagram
title: BLE GATT TYPICAL MESSAGE FLOW
participant Server App
participant Server Stack
participant Client Stack
participant Client App
opt Exchange MTU
    Client App->>+Client Stack:gatt_manager_client_mtu_exch_send
    Client Stack->>Server Stack:Exchange MTU Resquest
    Server Stack->>Client Stack:Exchange MTU Response
    Server Stack-->>Server App:gatt_event:MTU_CHANGED_INDICATION
    Client Stack-->>-Client App:gatt_event:MTU_CHANGED_INDICATION
end
note over Client Stack,Client App:Service discovery
Client App->>+Client Stack:gatt_manager_client_svc_discover_by_uuid
Client Stack-->>-Client App:gatt_event:CLIENT_PRIMARY_SVC_DIS_IND
Client App->>+Client Stack:gatt_manager_client_char_discover_by_uuid
Client Stack-->>-Client App:gatt_event:CLIENT_CHAR_DIS_BY_UUID_IND
Client App->>+Client Stack:gatt_manager_client_desc_char_discover
Client Stack-->>-Client App:gatt_event:CLIENT_CHAR_DESC_DIS_BY_UUID_IND
note over Server App,Client App:Client Initiated Operation
opt Write Without Response
    Client App->>+Client Stack:gatt_manager_client_write_no_rsp
    Client Stack-->>-Client App:gatt_event:CLIENT_WRITE_NO_RSP_DONE
    Client Stack->>Server Stack:Write command
    Server Stack-->>Server App:gatt_event:SERVER_WRITE_REQ
end
opt Write Characteristic Value
    Client App->>+Client Stack:gatt_manager_client_write_with_rsp
    Client Stack->>Server Stack:Write Request
    Server Stack-->>Server App:gatt_event:SERVER_WRITE_REQ
    Server Stack->>Client Stack:Write Response
    Client Stack-->>-Client App:gatt_event:CLIENT_WRITE_WITH_RSP_DONE
end
opt Read Characteristic Value
    Client App->>+Client Stack:gatt_manager_client_read
    Client Stack->>Server Stack:Read Request
    Server Stack-->>Server App:gatt_event:SERVER_READ_REQ
    Server App->>Server Stack:gatt_event_server_read_req_reply
    Server Stack->>Client Stack:Read Response
    Client Stack-->>-Client App:gatt_event:CLIENT_RD_CHAR_VAL_BY_UUID_IND
end
note over Server App,Client App:Server Initiated Operation
opt Notification
    Server App->>+Server Stack:gatt_manager_server_send_notification
    Server Stack-->>-Server App:gatt_event:SERVER_NOTIFICATION_DONE
    Server Stack->>Client Stack:Handle Value Notification
    Client Stack-->>Client App:gatt_event:CLENT_RECV_NOTIFICATION
end
opt Indication
    Server App->>+Server Stack:gatt_manager_server_send_indication
    Server Stack->>Client Stack:Handle Value Indication
    Client Stack-->>+Client App:gatt_event:CLENT_RECV_INDICATION
    Client App->>-Client Stack:gatt_manager_client_indication_confirm
    Client Stack->>Server Stack:Handle Value Confirmation
    Server Stack-->>-Server App:gatt_event:SERVER_INDICATION_DONE
end
```