# TRNG

## 一、TRNG 简介
  真随机数发生器（TRNG）可生成 16 位并行真随机数

## 二、TRNG 接口介绍
### 2.1 初始化
首先需要进行 TRNG 模块初始化
```c
HAL_StatusTypeDef HAL_TRNG_Init(void);
```

### 2.2 随机数生成
```c
HAL_StatusTypeDef HAL_TRNG_GenerateRandomNumber(uint32_t *random32bit);

HAL_StatusTypeDef HAL_TRNG_GenerateRandomNumber_IT(void);

void HAL_TRNG_ReadyDataCallback(uint32_t random32bit);
```   
- HAL_TRNG_GenerateRandomNumber是轮询模式的API，生成的随机数写入 *random32bit
- HAL_TRNG_GenerateRandomNumber_IT是中断模式的API，生成的随机数作为回调函数的参数提供给用户
- HAL_TRNG_ReadyDataCallback属于弱定义，用户可以自行定义，并完成相应的逻辑处理，参数random32bit是中断模式下生成的随机数

### 2.3 反初始化
通过反初始化接口，应用程序可以关闭 TRNG 外设，从而在运行BLE的程序的时候，降低系统的功耗
```c
HAL_StatusTypeDef HAL_TRNG_DeInit(void);
```