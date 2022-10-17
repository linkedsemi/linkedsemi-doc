# AES加解密--填充算法使用示例

例程路径：ls_sdk/examples/peripheral/crypt/ecb_it_padding

## 一  Padding简介

在AES加密与解密过程中，需要对数据进行填充。填充后的数据长度必须为16的整数倍。

padding分为以下几种，不同的padding对加密与解密有影响，所以要保证padding的方式是一致的。

### 1.1  Padding_None

无填充，需要保证明文一定是16的整数倍。

### 1.2  Padding_PKCS7

在PKCS7的方式下，如果一共需要padded多少个字节，所有填充的地方都填充这个值。

例子1： | DD DD DD DD DD DD DD DD | DD DD DD 05 05 05 05 05 |

例子2： | DD DD DD DD DD DD DD DD | DD DD DD DD DD DD DD DD | 10 10 10 10 10 10 10 10 | 10 10 10 10 10 10 10 10 |

### 1.3  Padding_ANSIX923

在ANSIX923的方式下，先填充00，最后一个字节填充padded的字节个数。

例子1： | DD DD DD DD DD DD DD DD | DD DD DD 00 00 00 00 05 |

例子2： | DD DD DD DD DD DD DD DD | DD DD DD DD DD DD DD DD | 00 00 00 00 00 00 00 00 | 00 00 00 00 00 00 00 10 |

### 1.4  Padding_ISO7816

在ISO7816方式下，第一个填充的字节是80，后面的都填充00。

例子1： | DD DD DD DD DD DD DD DD | DD DD DD 80 00 00 00 00 |

例子2： | DD DD DD DD DD DD DD DD | DD DD DD DD DD DD DD DD | 80 00 00 00 00 00 00 00 | 00 00 00 00 00 00 00 00 |

### 1.5  Padding_ISO10126

在ISO10126的方式下，先是填充随机值，最后一个字节填充padded的字节个数。

例子1： | DD DD DD DD DD DD DD DD | DD DD DD xx xx xx xx 05 |

例子2： | DD DD DD DD DD DD DD DD | DD DD DD DD DD DD DD DD | xx xx xx xx xx xx xx xx | xx xx xx xx xx xx xx 10 |

## 二  程序基本配置及说明

1.初始化操作：sys_init_none(); HAL_LSCRYPT_Init();

2.选择不同padding模式，对数据进行加解密

```c
    mode = Padding_None;
    crypt_cbc_ecb_test_128_None();
    mode = Padding_PKCS7;
    crypt_cbc_ecb_test_128_PKCS7();
    mode = Padding_ANSIX923;
    crypt_cbc_ecb_test_128_ANSIX923();
    mode = Padding_ISO7816;
    crypt_cbc_ecb_test_128_ISO7816();
    mode = Padding_ISO10126;
    crypt_cbc_ecb_test_128_ISO10126();
```

3.AES加解密测试

ecb_it_padding示例程序，在IT模式下实现以上5种padding。

测试明文有两种：1. cbc_ecb_plaintext_aligned ：明文是16的整数倍；2. cbc_ecb_plaintext_unaligned :明文不是16的整数倍

## 三  操作步骤及结果

### 3.1 操作步骤

1 设置填充模式

```c
mode = Padding_ISO7816;
HAL_LSCRYPT_Block_Padding_Mode_Set(mode);
```

2 选择128位的密钥

```c
HAL_LSCRYPT_AES_Key_Config(cbc_ecb_key_128, AES_KEY_128);  
```

3.选择16个字节对齐的明文进行加密

```c
HAL_LSCRYPT_AES_ECB_Encrypt_IT(cbc_ecb_plaintext_aligned, sizeof(cbc_ecb_plaintext_aligned), ciphertext_buff);  
```

4.对第3步加密后的密文进行解密

```c
HAL_LSCRYPT_AES_ECB_Decrypt_IT(ecb_ciphertext_128_ISO7816_aligned, sizeof(ecb_ciphertext_128_ISO7816_aligned), plaintext_buff); 
 ```

### 3.2 测试结果

```c
00> I/NO_TAG:CRYPT_AES_ECB_ENCRYPT_Padding_None_TEST_SUCCESS!
00> I/NO_TAG:CRYPT_AES_ECB_DECRYPT_Padding_None_TEST_SUCCESS!
00> I/NO_TAG:CRYPT_AES_ECB_ENCRYPT_Padding_PKCS7_TEST_SUCCESS!
00> I/NO_TAG:CRYPT_AES_ECB_DECRYPT_Padding_PKCS7_TEST_SUCCESS!
00> I/NO_TAG:CRYPT_AES_ECB_ENCRYPT_Padding_ANSIX923_TEST_SUCCESS!
00> I/NO_TAG:CRYPT_AES_ECB_DECRYPT_Padding_ANSIX923_TEST_SUCCESS!
00> I/NO_TAG:CRYPT_AES_ECB_ENCRYPT_Padding_ISO7816_TEST_SUCCESS!
00> I/NO_TAG:CRYPT_AES_ECB_DECRYPT_Padding_ISO7816_TEST_SUCCESS!
00> I/NO_TAG:CRYPT_AES_ECB_ENCRYPT_Padding_ISO10126_TEST_SUCCESS!
00> I/NO_TAG:CRYPT_AES_ECB_DECRYPT_Padding_ISO10126_TEST_SUCCESS!
```
  