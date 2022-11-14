# ADC 多通道中断采样

例程路径：ls_sdk\examples\peripheral\adcv2\adc_multi_channel

## 一、功能描述

- 应用上面在使用中断触发ADC采样功能时，当ADC转换完成后，会根据应用配置上相应转换完成中断。
- 使用循环通道方式时，应用上面触发一次后，当转换完循环组最后一个通道时，会自动重新开始下一轮的转换，直到应用上面去停止采样为止。

## 二、软件配置

### 2.1 基本配置

- 基本ADC采样配置和通道配置与轮询采样中介绍的一致，不再赘述。

### 2.2 事件触发和中断回调

- 当使用规则组时

  ```
  // 开始中断采集
  HAL_StatusTypeDef HAL_ADC_Start_IT(ADC_HandleTypeDef *hadc);
  
  // 采集完成后的中断回调，需要在回调函数里面去获取ADC的值
  void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc);
  ```

- 当使用注入组时

  ```
  // 开始中断采集
  HAL_StatusTypeDef HAL_ADCEx_InjectedStart_IT(ADC_HandleTypeDef* hadc);
  
  // 采集完成后的中断回调，需要在回调函数里面去获取ADC的值
  void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc);
  ```

- 当使用循环组时

  ```
  HAL_StatusTypeDef HAL_ADCx_LoopStart_IT(ADC_HandleTypeDef *hadc);
  ```

  

## 三、下载验证

将开发板的PA01、PA02、PA03输入电压，运行程序，打开RTT LOG查看打印信息，三个通道的AD值，依次用规则通道、注入通道、轮询通道的方式捕获到并且打印出来。