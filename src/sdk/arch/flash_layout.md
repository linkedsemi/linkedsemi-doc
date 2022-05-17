# FLASH布局

## LE501X

BLE Application

| Region                               | Address Range (512KB)   | Size  | Address Range (1MB)     | Size  |
| ------------------------------------ | ----------------------- | ----- | ----------------------- | ----- |
| OTA Settings                         | 0x1807F000 - 0x1807FFFF | 4KB   | 0x180FF000 - 0x180FFFFF | 4KB   |
| TinyFS Data Storage                  | 0x1807C000 - 0x1807EFFF | 12KB  | 0x180FC000 - 0x180FEFFF | 12KB  |
| App                                  | 0x18034000 - 0x1807BFFF | 288KB | 0x18034000 - 0x180FBFFF | 800KB |
| BLE Host & Controller Protocol Stack | 0x18002000 - 0x18033FFF | 200KB | 0x18002000 - 0x18033FFF | 200KB |
| Info Page & Second Bootloader        | 0x18000000 - 0x18001FFF | 8KB   | 0x18000000 - 0x18001FFF | 8KB   |



MESH Application

| Region                               | Address Range (512KB)   | Size  | Address Range (1MB)     | Size  |
| ------------------------------------ | ----------------------- | ----- | ----------------------- | ----- |
| OTA Settings                         | 0x1807F000 - 0x1807FFFF | 4KB   | 0x180FF000 - 0x180FFFFF | 4KB   |
| TinyFS Data Storage                  | 0x1807C000 - 0x1807EFFF | 12KB  | 0x180FC000 - 0x180FEFFF | 12KB  |
| App                                  | 0x18056000 - 0x1807BFFF | 152KB | 0x18056000 - 0x180FBFFF | 664KB |
| BLE MESH Protocol Stack              | 0x18033XXX - 0x18055FFF | 136KB | 0x18033XXX - 0x18055FFF | 136KB |
| BLE Host & Controller Protocol Stack | 0x18002000 - 0x18033XXX | 200KB | 0x18002000 - 0x18033XXX | 200KB |
| Info Page & Second Bootloader        | 0x18000000 - 0x18001FFF | 8KB   | 0x18000000 - 0x18001FFF | 8KB   |

***For 1MB Flash, refer to [1MB LE5010注意事项](../notice_le5010.md) to modify the base address of TinyFS Data Storage Area***

MCU Application （without BLE&MESH Protocol Stack）

| Region                        | Address Range (512KB)   | Size  | Address Range (1MB)     | Size  |
| ----------------------------- | ----------------------- | ----- | ----------------------- | ----- |
| OTA Settings                  | 0x1807F000 - 0x1807FFFF | 4KB   | 0x180FF000 - 0x180FFFFF | 4KB   |
| App                           | 0x18002000 - 0x1807EFFF | 500KB | 0x18002000 - 0x180FEFFF | 1012KB |
| Info Page & Second Bootloader | 0x18000000 - 0x18001FFF | 8KB   | 0x18000000 - 0x18001FFF | 8KB   |
