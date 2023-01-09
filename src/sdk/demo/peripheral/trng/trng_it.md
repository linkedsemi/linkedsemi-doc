# TRNG_IT 使用示例

例程路径： ls_sdk/examples/peripheral/trng/trng_it

## 一、程序基本配置及说明：
- 本例程演示了如何使用TRNG单个模块在中断模式下生成随机数，以供参考

## 二、操作步骤及结果：
### 2.1 初始化TRNG模块
```c
HAL_TRNG_Init();
```
### 2.2 生成随机数
- 例程以一秒为周期调用 HAL_TRNG_GenerateRandomNumber_IT()函数生成随机数
```c
while (1)
{
    HAL_TRNG_GenerateRandomNumber_IT();
    DELAY_US(1000 * 1000);
}
```
### 2.3 定义回调函数
- 将生成的随机数通过log输出
```c
void HAL_TRNG_ReadyDataCallback(uint32_t random32bit)
{
    LOG_I("RandomNumber: %x", random32bit);
}
```
### 2.4 测试结果
结果(不唯一)如下：
```
I/NO_TAG:RandomNumber: 0d6257ae
I/NO_TAG:RandomNumber: 17aeb7b2
I/NO_TAG:RandomNumber: 5e08ac5f
I/NO_TAG:RandomNumber: fb5fd94a
I/NO_TAG:RandomNumber: 0afe8fed
I/NO_TAG:RandomNumber: 46022fb2
I/NO_TAG:RandomNumber: f83d8abf
...
```