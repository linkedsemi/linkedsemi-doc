.. _sha_ref:

SHA
==============

一、SHA简介
--------------
SHA-2,名称来自于安全散列算法2(英语:Secure Hash Algorithm 2)的缩写,一种密码散列函数算法标准,由美国国家安全局研发,属于SHA算法之一,是SHA-1的后继者。
SHA-2下又可再分为六个不同的算法标准,包括了:SHA-224、SHA-256、SHA-384、SHA-512、SHA-512/224、SHA-512/256。这些变体除了生成摘要的长度 、循环运行的次数等一些微小差异外，算法的基本结构是一致的。

SHA-256
    对于任意长度的消息,SHA256都会产生一个256位的哈希值,称作消息摘要。这个摘要相当于是个长度为32个字节的数组,通常用一个长度为64的十六进制字符串来表示.
SHA-224
    对于任意长度的消息,SHA224都会产生一个224位的哈希值.
SM3
    对于任意长度的消息,SM3都会产生一个256位的哈希值.

以SHA256为例,算法过程如下:

消息填充
    在将数据输入HASH计算之前,必须根据SHA或SM3算法填充数据流。
    假设消息M的二进制编码长度为L位,首先在消息末尾补上一位"1", 然后再补上K个"0",其K为下列方程的最小非负整数:
    L + 1 + K ≡ 448 mod 512
    然后,再在上述字符串后面补上L的二进制表示形式(64位)
    填充操作生成长度为L + 1 + K + 64的填充消息,其结构是512位的整数倍。
    
消息摘要计算
    假设消息M可以被分解为n个块,于是整个算法需要做的就是完成n次迭代,n次迭代的结果就是最终的哈希值，即256bit的数字摘要。
    一个256-bit的摘要的初始值H0,经过第一个数据块进行运算,得到H1,即完成了第一次迭代
    H1经过第二个数据块得到H2,……,依次处理,最后得到Hn,Hn即为最终的256-bit消息摘要。

二、SHA接口介绍
----------------------
2.1 初始化:
----------------------
首先需要进行SHA模块初始化。

.. code ::

    HAL_SHA_Init(void);

2.2 SHA计算:
------------------

**参数描述**

.. note ::

    #. data:明文数据，没有经过加密过后的数据。
    #. length:明文的大小
    #. sha256[SHA256_WORDS_NUM]: 存放哈希值的地址

2.2.1 轮询模式
......................

.. code ::

    HAL_LSSHA_SHA256(const uint8_t *data,uint32_t length,uint32_t sha256[SHA256_WORDS_NUM]);
    HAL_LSSHA_SHA224(const uint8_t *data,uint32_t length,uint32_t sha224[SHA224_WORDS_NUM]);
    HAL_LSSHA_SM3(const uint8_t *data,uint32_t length,uint32_t sm3[SM3_WORDS_NUM]);

2.2.2 中断模式
......................
对于每个块，HASH将触发一个“等待数据中断”，通知软件将数据块提供给HASH处理器。在512位的数据块准备好之后，HASH处理器开始计算。
当完成当前512位块的计算后，HASH处理器将触发另一个“等待数据中断”，通知软件输入下一个数据块。 
当HASH处理器处理完由寄存器calc_len定义的一些数据块后，HASH处理器将不再请求更多的数据块，并触发“完成中断”。

.. code ::

    HAL_LSSHA_SHA256_IT(const uint8_t *data,uint32_t length,uint32_t sha256[SHA256_WORDS_NUM]);
    HAL_LSSHA_SHA224_IT(const uint8_t *data,uint32_t length,uint32_t sha224[SHA224_WORDS_NUM]);
    HAL_LSSHA_SM3_IT(const uint8_t *data,uint32_t length,uint32_t sm3[SM3_WORDS_NUM]);

2.2.3 回调函数 
......................
.. code ::

    HAL_LSSHA_SHA256_Complete_Callback(void);
    HAL_LSSHA_SHA224_Complete_Callback(void);
    HAL_LSSHA_SM3_Complete_Callback(void);

.. note ::
    
    | 当计算完成后，会将消息摘要存放到相应的buffer中，然后调用相应的回调函数进行数据比较操作。不同SHA算法对应不同的回调函数。
    | 这个函数属于弱定义，用户可以自行定义，并完成相应的逻辑处理。

2.3 反初始化
---------------

反初始化SHA模块
.........................

通过反初始化接口,应用程序可以关闭SHA外设,从而在运行BLE的程序的时候,降低系统的功耗。

.. code ::

    HAL_LSSHA_DeInit(void);