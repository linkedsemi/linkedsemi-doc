# 裸机应用

## 异步函数调用

在BLE裸机应用中，外设中断处理常常需要触发BLE行为，而BLE API要求不能在中断上下文调用。因此需要有一个通过中断抛出消息，并在大循环中执行的机制。

```c
#include "ls_sys.h"
/** \brief Post a function to be called in the main loop with specific parameter
 *  \param[in] func The pointer to the function
 *  \param[in] param The pointer of the object with global lifecycle that will be passed to func
 */
void func_post(void (*func)(void *),void *param);
```

func_post函数是中断安全、可重入的。多次以相同参数(func,param)调用func_post，最终func(param)也会执行同样次数。

## 软件定时器

参见[软件定时器](../module/software_timer)
