---
updated: '2018-04-10 00:00:00'
categories: code
excerpt: 在分析 H264 数据时碰到的问题，在此进行记录
date: '2018-04-10 00:00:00'
tags:
  - C++
urlname: bit-field-order
title: 字序对 bit field 的影响
---

# 问题


下面是在进行 H264 数据分析时碰到的问题


H264 nal indecator 是在 00 00 00 01 后面的一字节，其字段内容为： * forbidden_zero_bit(1) * nal_ref_idc(2) * nal_unit_type(5)


比如对 0x67 / 0b01100111，各字段为： * forbidden_zero_bit = 0 * nal_ref_idc = 0b11 = 3 * nal_unit_type = 0b111 = 7


常规的提取方式如下：


```text
uint8_t a = 0x67;
uint8_t f    = (a & 0b10000000) >> 7;
uint8_t nri  = (a & 0b01100000) >> 5;
uint8_t type = (a & 0b00011111);

```


f = 0, nri = 3, type = 7 结果没有问题。


但是如果采用 bit field 进行数据拆解则获取不到正确的结果：


```text
#include <stdio.h>
#include <inttypes.h>

struct NalIndicator {
    uint8_t f : 1;
    uint8_t nri : 2;
    uint8_t type : 5;
};

int main()
{
    NalIndicator * n;

    uint8_t a = 0x67;
    n = (NalIndicator *)(&a);

    uint8_t f    = (a & 0b10000000) >> 7;
    uint8_t nri  = (a & 0b01100000) >> 5;
    uint8_t type = (a & 0b00011111);

    printf("sizeof NalIndicator %d\n", sizeof(NalIndicator));
    printf("f    : %d, %d\n", n->f, f);
    printf("nri  : %d, %d\n", n->nri, nri);
    printf("type : %d, %d\n", n->type, type);

    return 0;
}

```


输出 > sizeof NalIndicator 1 f : 1, 0 nri : 3, 3 type : 12, 7 请按任意键继续…


# 分析


经过分析，发现是字序影响到了 bit field 的顺序而导致的。


按照以上代码的定义，实际上的各字段是按照从右往左排列的，也就是说：


```text
0x67 = 01100111 = 01100，11，1

```


其中 f = 1， nri = 0b11 = 3, type = 0b0110 = 12。


没想到字序除了会决定字节的顺序，还会对 bit field 的顺序造成影响。


# 解决方式


如果要编写跨平台的代码，需要针对不同字序的平台定义不同的 struct，并通过预处理器进行定义的切换，用户需要定义类似 BIG_ENDIAN / LITTLE_ENDIAN 的宏，比如 SDL 的 SDL_endian.h


```text
/**
 *  \name The two types of endianness
 */
/* @{ */
#define SDL_LIL_ENDIAN  1234
#define SDL_BIG_ENDIAN  4321
/* @} */

#ifndef SDL_BYTEORDER           /* Not defined in SDL_config.h? */
#ifdef __linux__
#include <endian.h>
#define SDL_BYTEORDER  __BYTE_ORDER
#else /* __linux__ */
#if defined(__hppa__) || \
    defined(__m68k__) || defined(mc68000) || defined(_M_M68K) || \
    (defined(__MIPS__) && defined(__MISPEB__)) || \
    defined(__ppc__) || defined(__POWERPC__) || defined(_M_PPC) || \
    defined(__sparc__)
#define SDL_BYTEORDER   SDL_BIG_ENDIAN
#else
#define SDL_BYTEORDER   SDL_LIL_ENDIAN
#endif
#endif /* __linux__ */
#endif /* !SDL_BYTEORDER */

```


想要在编译阶段自动检测字序是不可能的，无论什么方式都需要在程序执行阶段才能对字序及进行判断


```text
#include <stdint .h>
#define IS_BIG_ENDIAN (*(uint16_t *)"\0\xff" < 0x100)

```


不过我们可以针对用户通过宏定义的字序进行检查，并在运行时抛出异常：


```text
inline int IsBigEndian()
{
    int i=1;
    return ! *((char *)&i);
}

/* ... */

#ifdef BIG_ENDIAN
assert(IsBigEndian());
#elif LITTLE_ENDIAN
assert(!IsBigEndian());
#else
#error "No endianness macro defined"
#endif

```


还是乖乖用位运算吧


参考资料


http://mjfrazer.org/mjfrazer/bitfields/


https://yumichan.net/video-processing/video-compression/introduction-to-h264-nal-unit/

