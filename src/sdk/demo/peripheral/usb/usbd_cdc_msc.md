# usbd_cdc_msc 使用示例

例程路径： ls_sdk/examples/peripheral/usb/usb_cdc_msc

本例程演示了如何使用usbd_cdc_msc运行usbd msc(mass storage controller)实例。

> 注：出于对后续更新tinyusb仓库的考虑，为尽量保证tinyusb仓库的完整性，SDK的examples路径下只有app_config.h和SConstruct，demo的源代码在tinyusb的examples下

## 一、初始化及board配置

board相关的初始化代码如下：

```c
void board_init(void)
{
    sys_init_none();
    pinmux_usb_init();
}
```

pinmux_usb_init会使用默认的DP/DM作为USB通信的两根IO

除此之外，为了适配tinyusb应用需求，需要实现board_millis()函数接口：

```c
uint32_t board_millis(void)
{
    return systick_get_value();
}
```

## 二、测试准备：

### 2.1 硬件准备

使用Gemini demo板，确保DP(PA12)有1.5KΩ上拉电阻到VDD33

### 2.2 软件准备

建议安装bushound软件

## 三、操作步骤及结果：

编译生成usbd_cdc_msc.hex文件，下载到demo板中，运行起来之后，插入USB线到PC，能在PC端看到识别的U盘，打开之后能看到一个txt文本文件，可以修改txt文本内容。修改后保存，拔掉USB再插入（确保没有断电），再次读取该文本文件，能看到修改后的内容

操作前可以打开bushound，抓取测试流程中所有USB通信数据

