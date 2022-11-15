.. _rtcv2_ref:

RTCV2
==============

一、RTCV2简介
--------------

rtcv2模块是一个独立的计时器。RTC提供了一组连续运行的计数器，以提供时钟日历功能。可以写入计数器值来设置系统的当前时间/日期。
#define SDK_LSI_USED (0) 表示启用外部低速时钟，**此时板子上需要外挂32768晶振**。

二、RTCV2接口介绍
----------------------
2.1 初始化:
++++++++++++++++++++++++++++++
首先需要进行rtcv2模块初始化。

.. code ::

    HAL_RTC_Init(void);   

2.2 参数配置:
++++++++++++++++++++++++++++++
.. code ::

    rtc_Cycle_Config(uint32_t cyc_1hz, uint32_t calib_cyc, bool calib_en);
    HAL_RTC_Cycle_Config(void);
.. note ::

    | cyc_1hz: 计数器在第 cyc_1hz 个周期溢出1次（增加一秒）
    | calib_cyc: 计数器每60秒校准一次，以补偿系统低速时钟的频率偏差。启用校准功能后，则每分钟，计数器在第 cyc_1hz*59 个周期累计溢出 59 次，然后在第 cyc_1hz+calib_cyc 个周期溢出计数器
    | calib_en: 是否开启校准功能   
    | HAL_RTC_Cycle_Config 这个函数属于弱定义，在 HAL_RTC_Init() 中被调用。用户可以自行定义，调用rtc_Cycle_Config()实现对不同时钟频率下参数的配置

2.3 万年历时间:
++++++++++++++++++++++++++++++
万年历当前时间的设置及获取。

.. code ::

    HAL_RTC_CalendarSet(calendar_cal_t *calendar_cal, calendar_time_t *calendar_time);
    HAL_RTC_CalendarGet(calendar_cal_t *calendar_cal, calendar_time_t *calendar_time);
.. note ::

    | calendar_cal: 设置、获取的日期（年、月、周）
    | calendar_time: 设置、获取的时间（日、时、分、秒）

2.4 闹钟:
++++++++++++++++++++++++++++++
闹钟的时间设置，以及关闭。

.. code ::

    HAL_RTC_AlarmSet(calendar_cal_t *calendar_cal, calendar_time_t *calendar_time);
    HAL_RTC_alarm_callback(void);
    HAL_RTC_AlarmClear(void);
.. note ::

    | HAL_RTC_alarm_callback于弱定义，用户可以自行定义，并完成相应的逻辑处理
    | 当前时间等于闹钟设置时间时，则会触发中断，调用 HAL_RTC_alarm_callback()

2.5 反初始化:
++++++++++++++++++++++++++++++
反初始化rtcv2模块。

通过反初始化接口，应用程序可以关闭rtcv2外设，从而在运行BLE的程序的时候，降低系统的功耗。

.. code ::

    HAL_RTC_DeInit(void);