# usbd_hid_composite 使用示例

例程路径： ls_sdk/examples/peripheral/usb/usb_hid_composite

本例程演示了如何使用usb_hid_composite运行usbd hid(keyboard/mouse/gamepad)实例。

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

以及board_button_read()函数接口：

```c
uint32_t board_button_read(void)
{
    static uint32_t button = 0;
    if (button > 4)
    {
        button = 0;
    }
    else
    {
        button++;
    }
    
    return button;
}
```

该函数是用于模拟按键输入

## 二、测试准备：

### 2.1 硬件准备

使用Gemini demo板，确保DP(PA12)有1.5KΩ上拉电阻到VDD33

### 2.2 软件准备

建议安装bushound软件

## 三、操作步骤及结果：

编译生成usbd_hid_composite.hex文件，下载到demo板中，运行起来之后，在PC端打开一个空白文本文档，然后将鼠标移动到屏幕左上位置。之后将demo板USB线插入PC端，能看到有HID设备识别，文本文档不断输入字符"a"，鼠标会持续向屏幕右下方移动，同时电脑音量会一直减小直到静音

操作前可以打开bushound，抓取测试流程中所有USB通信数据

