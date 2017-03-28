# Cortex-M 开发中实现stdout(printf, puts, putc)

开发嵌入式程序时，由于没有标准输出设备，通常要把标准库的stdout定向到串口或usb等，用于调试;也就是说，在使用printf，puts,putc
等函数时，要先提供stdout基本功能的实现，标准的printf等会调用用户定义的实现函数作为stdout.

## 使用armcc和Microlib

* 实现`fputc`函数
* armcc armlink armasm参数中添加`--library_type=microlib`,或者使用IDE指定

注意:Microlib不支持p记数法和宽字符，也就是`%lc`,`%ls`, `%a` .  
示例(基于stm32,定向到USART,需要提前配置好外设)：

``` c
#ifdef __CC_ARM
int fputc(int ch, FILE *f)
{
    USART_SendData(USART1,(uint8_t)ch);
    while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
    return ch;
}
#endif
```

## 使用gcc和Newlib-Nano

* 实现`_write`函数
* arm-none-eabi-gcc 参数中添加 `--specs=nano.specs`,或者使用IDE指定

如果要输出浮点数，需要使用`-u _printf_float`
示例(基于stm32,定向到USART,需要提前配置好外设)：

``` c
#ifdef __GNUC__
int _write (int fd, char *pBuffer, int size)
  {
      for (int i = 0; i < size; i++)
      {
        USART_SendData(USART1,pBuffer[i]);
        while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
      }
      return size;
  }
#endif
```

## 参考资料

1. [Tailoring the microlib input/output functions](http://www.keil.com/support/man/docs/armlib/armlib_chr1358938940288.htm)
1. [Redefining target-dependent system I/O functions in the C library](http://www.keil.com/support/man/docs/armlib/armlib_chr1358938932518.htm)
1. [Redefining low-level library functions to enable direct use of high-level library functions in the C library](http://www.keil.com/support/man/docs/armlib/armlib_chr1358938931411.htm)
1. [printf() with newlib-nano vs. newlib / retargeting to UART](http://china.cypress.com/forum/psoc-5-known-problems-and-solutions/printf-newlib-nano-vs-newlib-retargeting-uart)
1. [GNU Tools for ARM Embedded Processors](https://launchpadlibrarian.net/287100883/readme.txt)
1. [Building ARM Projects with Newlib-Nano](http://visualgdb.com/tutorials/arm/newlib-nano/)