# SHA_IT 使用示例

例程路径: ls_sdk\examples\peripheral\sha\sha_it

## 一、程序基本配置及说明：

```c
/*sha模块初始化*/
HAL_LSSHA_Init();
```

## 二、操作步骤及结果：

### 2.1 操作步骤

说明：确定好要加密的明文之后，使用在线计算工具(https://the-x.cn/cryptography/Aes.aspx)，计算出对应的哈希值(消息摘要)，放入密文buffer.

```c
/*明文：*/
static const uint8_t plaintext[32] = {
    0xA7, 0xFC, 0xFC, 0x6B, 0x52, 0x69, 0xBD, 0xCC, 0xE5, 0x71, 0x79, 0x8D, 0x61, 0x8E, 0xA2, 0x19,
    0xA6, 0x8B, 0x96, 0xCB, 0x87, 0xA0, 0xE2, 0x10, 0x80, 0xC2, 0xE7, 0x58, 0xD2, 0x3E, 0x4C, 0xE9};
```

```c
/*sha256算法产生的消息摘要：*/
static const uint32_t ciphertext_sha256[8] = {
    0x96ada97d, 0x8abbef25, 0xc7703811, 0x7a52a8e4, 0x6a05e8f1, 0xfa553d87, 0x787e05a3, 0x5adccf5b};  
```

```c
/*sha224算法产生的消息摘要：*/
static const uint32_t ciphertext_sha224[7] = {
    0xc3d1a3c4, 0xb7942f35, 0xf2d61b8d, 0x5bbfcad8, 0x65e8269c, 0xbbdc1514, 0x9afec5b};
```

```c
/*sm3算法产生的消息摘要：*/
static const uint32_t ciphertext_sm3[8] = {
    0xd5339620, 0x1e34260d, 0xaa20ce75, 0x4897615c, 0x51ad9f0c, 0xedcf9ea9, 0x63008679, 0x3d914c08};
```

```c
/*存放不同哈希算法计算的消息摘要的buffer：*/
uint32_t cipherbuffer_sha256[8];
uint32_t cipherbuffer_sha224[7];
uint32_t cipherbuffer_sm3[8];
```

```c
/*开始计算哈希值：*/
static void sha_crypt_test()
{
    HAL_LSSHA_SHA256_IT(plaintext, sizeof(plaintext), cipherbuffer_sha256);
    HAL_LSSHA_SHA224_IT(plaintext, sizeof(plaintext), cipherbuffer_sha224);
    HAL_LSSHA_SM3_IT(plaintext, sizeof(plaintext), cipherbuffer_sm3);
}
```

---plaintext：明文的地址
---sizeof(plaintext): 明文的大小
---cipherbuffer_sha256: 存放哈希值的地址

```c
/*计算完成之后进入回调函数，不同的哈希计算对应不同的回调函数，SHA256的回调函数如下：*/
void HAL_LSSHA_SHA256_Complete_Callback()
{
    if (!memcmp(cipherbuffer_sha256, ciphertext_sha256, sizeof(cipherbuffer_sha256)))
    {
        LOG_I("SHA256_ENCRYPT_TEST_SUCCESS!");
    }
    else
    {
        LOG_I("SHA256_ENCRYPT_TEST_FAIL!");
    }
}
```

### 2.2 测试结果

00> I/NO_TAG:SHA256_ENCRYPT_TEST_SUCCESS!
00> I/NO_TAG:SHA224_ENCRYPT_TEST_SUCCESS!
00> I/NO_TAG:SM3_ENCRYPT_TEST_SUCCESS!