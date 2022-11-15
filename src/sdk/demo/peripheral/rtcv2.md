# RTCV2（万年历）使用示例

例程路径： ls_sdk/examples/peripheral/rtcv2

## 一、程序基本配置及说明：
#define SDK_LSI_USED 0 ->表示使用LSE做RTC时钟源，**此时板子上需要外挂32768晶振**
```c
//延时2秒，等待外部晶振稳定
DELAY_US(2000 * 1000);
//RTC模块初始化
HAL_RTC_Init();
```

## 二、操作步骤
### 2.1 设置万年历时间
```c
static void rtc_test_calendar_set(void)
{
   calendar_cal_t calendar_cal;
   calendar_cal.year = 22;
   calendar_cal.mon = 11;
   calendar_cal.week = 4;
   calendar_time_t calendar_time;
   calendar_time.date = 10;
   calendar_time.hour = 22;
   calendar_time.min = 59;
   calendar_time.sec = 50;
   HAL_RTC_CalendarSet(&calendar_cal, &calendar_time);
   LOG_I("rtc test, calendar set done\n\n");
}
```
### 2.2 设置闹钟时间
```c
static void rtc_test_alarm_set(void)
{
   calendar_cal_t calendar_cal;
   calendar_cal.year = 22;
   calendar_cal.mon = 11;
   calendar_cal.week = 4;
   calendar_time_t calendar_time;
   calendar_time.date = 10;
   calendar_time.hour = 23;
   calendar_time.min = 0;
   calendar_time.sec = 0;
   HAL_RTC_AlarmSet(&calendar_cal, &calendar_time);
   LOG_I("rtc test, alarm set done\n\n");
}
```
### 2.3 获取当前时间并输出
```c
static void rtc_test_calendar_get(void)
{
   calendar_cal_t calendar_cal;
   calendar_time_t calendar_time;
   HAL_RTC_CalendarGet(&calendar_cal, &calendar_time);
   if (calendar_time.sec == beforeSec)
      return;
   beforeSec = calendar_time.sec;
   LOG_I("%02d/%02d/%02d, %02d:%02d:%02d, week=%d", calendar_cal.year,
         calendar_cal.mon, calendar_time.date, calendar_time.hour,
         calendar_time.min, calendar_time.sec, calendar_cal.week);
}
```
### 2.4 定义闹钟回调函数
```c
void HAL_RTC_alarm_callback(void)
{
   LOG_I("====== %s ======", __func__);
}
```
### 2.5 定义参数配置函数（参考，测试例程中没有定义）
```c
HAL_StatusTypeDef HAL_RTC_Cycle_Config(void)
{
   rtc_Cycle_Config(SDK_LCLK_HZ, 100, true);
   return HAL_OK;
}
```
- SDK_LCLK_HZ：计数器在第 SDK_LCLK_HZ 个周期溢出1次（增加一秒）
- 100：计数器每60秒校准一次，以补偿系统低速时钟的频率偏差。启用校准功能后，则每分钟，计数器在第 SDK_LCLK_HZ*59 个周期累计溢出 59 次，然后在第 SDK_LCLK_HZ+100 个周期溢出计数器1次
- true：开启校准功能

## 三 、测试结果
```c
I/NO_TAG:rtc test, calendar set done
I/NO_TAG:rtc test, alarm set done
I/NO_TAG:22/11/10, 22:59:50, week=4
I/NO_TAG:22/11/10, 22:59:51, week=4
I/NO_TAG:22/11/10, 22:59:52, week=4
I/NO_TAG:22/11/10, 22:59:53, week=4       
I/NO_TAG:22/11/10, 22:59:54, week=4    
I/NO_TAG:22/11/10, 22:59:55, week=4
I/NO_TAG:22/11/10, 22:59:56, week=4
I/NO_TAG:22/11/10, 22:59:57, week=4
I/NO_TAG:22/11/10, 22:59:58, week=4
I/NO_TAG:22/11/10, 22:59:59, week=4
I/NO_TAG:====== HAL_RTC_alarm_callback ======
I/NO_TAG:22/11/10, 22:00:00, week=4
I/NO_TAG:22/11/10, 23:00:01, week=4
......
```