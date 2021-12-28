# FLASH布局

## LE501X

BLE Application

| Region                               | Address Range           | Size  |
| ------------------------------------ | ----------------------- | ----- |
| OTA Settings                         | 0x1807F000 - 0x1807FFFF | 4KB   |
| TinyFS Data Storage                  | 0x1807C000 - 0x1807EFFF | 12KB  |
| App                                  | 0x18034000 - 0x1807BFFF | 288KB |
| BLE Host & Controller Protocol Stack | 0x18002000 - 0x18033FFF | 200KB |
| Info Page & Second Bootloader        | 0x18000000 - 0x18001FFF | 8KB   |



MESH Application

| Region                               | Address Range           | Size  |
| ------------------------------------ | ----------------------- | ----- |
| OTA Settings                         | 0x1807F000 - 0x1807FFFF | 4KB   |
| TinyFS Data Storage                  | 0x1807C000 - 0x1807EFFF | 12KB  |
| App                                  | 0x18056000 - 0x1807BFFF | 152KB |
| BLE MESH Protocol Stack              | 0x18033XXX - 0x18055FFF | 136KB |
| BLE Host & Controller Protocol Stack | 0x18002000 - 0x18033XXX | 200KB |
| Info Page & Second Bootloader        | 0x18000000 - 0x18001FFF | 8KB   |



MCU Application （without BLE&MESH Protocol Stack）

| Region                        | Address Range           | Size  |
| ----------------------------- | ----------------------- | ----- |
| OTA Settings                  | 0x1807F000 - 0x1807FFFF | 4KB   |
| App                           | 0x18002000 - 0x1807EFFF | 500KB |
| Info Page & Second Bootloader | 0x18000000 - 0x18001FFF | 8KB   |
