.. _SM4_ref:

SM4
==============

一、SM4简介
--------------

硬件加密模块主要用于由硬件对数据进行密钥扩展，加密或解密操作。SM4.0（原名SMS4.0）由国家密码管理局于2012年3月21日发布。相关标准为 GM/T 0002-2012《SM4分组密码算法》（原SMS4分组密码算法）。与DES和AES算法类似，SM4算法是一种对称性的分组密码算法，其分组长度与密钥长度均为128bit。

SM4加密: SM4的分组长度为4字，因此，其输入是4字的明文(X1,X2,X3,X4)（其中Xi表示一个32位的字），经过加密后，得到32个字的密文(X5,X6,X7,...,X35)，将迭代最后得到的四个字进行反序，得到最终的密文(Y1,Y2,Y3,Y4)=(X35,X34,X33,X32)。这个加密过程分为两步，由32次轮迭代和1次反序变换组成。每一轮轮迭代都需要一个1字的轮密钥，总共需要32个轮密钥，记为(RK0,RK1,RK2,...,RK31)。

SM4解密: SM4的解密过程与加密过程完全相同，也包括32轮迭代和一次反序变换。只是在轮迭代的时候，需要将轮密钥逆序使用。(RK31,RK30,RK29,...,RK0)。

SM4密钥扩展: SM4的密钥扩展算法与加密算法相同。输入是4字的密钥(MK1,MK2,MK3,MK4)，经过扩展运算后，生成32个字的轮密钥(RK0,RK1,RK2,...,RK31)。

二、SM4接口介绍
----------------------
2.1 初始化:
++++++++++++++++++++++++++++++
首先需要进行SM4模块初始化。


.. code ::

    HAL_LSSM4_Init(void);
    
由于在进行加密解密的过程中需要使用到一个相同的key，
这个key用来加密明文的密码，在对称加密算法中，加密与解密的密钥是相同的。
密钥为接收方与发送方协商产生，但不可以直接在网络上传输，否则会导致密钥泄漏，最终导致明文被还原。
所以还需要输入一个key。如下代码所示：

.. code ::

    /*等待密钥扩展结束再返回*/
    HAL_StatusTypeDef HAL_SM4_KeyExpansion(const uint8_t *key);
    /*开始密钥扩展，通过触发中断调用回调函数*/
    HAL_StatusTypeDef HAL_SM4_KeyExpansion_IT(const uint8_t *key);
    void HAL_SM4_KeyExpansion_Complete_Callback();

2.2 SM4加解密:
++++++++++++++++++++++++++++++

**参数描述**

.. note ::

    #. data:待加密的明文数据。
    #. result:加密完成的密文数据。
    #. length:待加密的明文数据长度。

2.2.1 轮询模式
......................

.. code ::

    HAL_StatusTypeDef HAL_SM4_Encrypt(const uint8_t *data, uint8_t *result, uint32_t length);
    HAL_StatusTypeDef HAL_SM4_Decrypt(const uint8_t *data, uint8_t *result, uint32_t length);

2.2.2 中断模式
......................

.. code ::

    HAL_StatusTypeDef HAL_SM4_Encrypt_IT(const uint8_t *data, uint8_t *result, uint32_t length);
    HAL_StatusTypeDef HAL_SM4_Decrypt_IT(const uint8_t *data, uint8_t *result, uint32_t length);

2.2.3 回调函数 
......................
.. code ::

    void HAL_SM4_Calculation_Complete_Callback(bool Encrypt)
    {

    }
.. note ::
    
    | 函数中的“Encrypt”为true时选择的是加密模式，当为false时选择的是解密模式。
    | 这个函数属于弱定义，用户可以自行定义，并完成相应的逻辑处理。

2.3 反初始化
++++++++++++++++++++++++++++++

反初始化SM4模块
.........................

通过反初始化接口，应用程序可以关闭SM4外设，从而在运行BLE的程序的时候，降低系统的功耗。

.. code ::

    HAL_LSSM4_DeInit(void);