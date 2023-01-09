# TRNG_POLLING 使用示例

例程路径： ls_sdk/examples/peripheral/trng/trng_polling

## 一、程序基本配置及说明：
- 本例程演示了如何使用TRNG单个模块在轮询模式下生成随机数，以供参考

## 二、操作步骤及结果：
### 2.1 初始化TRNG模块
```c
HAL_TRNG_Init();
```
### 2.2 生成随机数
- 例程以一秒为周期调用HAL_TRNG_GenerateRandomNumber()函数生成随机数并通过log输出
```c
static uint32_t random32bit;

while (1)
{
    HAL_TRNG_GenerateRandomNumber(&random32bit);
    LOG_I("RandomNumber: %x", random32bit);
    DELAY_US(1000 * 1000);
}
```
### 2.3 测试结果
结果(不唯一)如下：
```
I/NO_TAG:RandomNumber: 3dc15174
I/NO_TAG:RandomNumber: c5ce9979
I/NO_TAG:RandomNumber: 740b0126
I/NO_TAG:RandomNumber: 0d2456b0
I/NO_TAG:RandomNumber: 773214ab
I/NO_TAG:RandomNumber: b08b836b
I/NO_TAG:RandomNumber: 9ec23c59
I/NO_TAG:RandomNumber: 13fdc118
I/NO_TAG:RandomNumber: e9c99642
I/NO_TAG:RandomNumber: 7b03a4da
...
```