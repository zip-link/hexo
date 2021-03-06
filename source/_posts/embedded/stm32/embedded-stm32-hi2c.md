---
title: stm32硬件I2C实现问题
categories:
  - 嵌入式
  - stm32
tags:
  - stm32
  - I2C
  - 单片机
date: 2020-01-31 07:34:33
---
虽然软件可实现I2C读取三轴传感器数据，但I2C作为一种重要的通信协议是一定要搞清楚问题所在的，SO继续研究之前的问题。（网上传言STM32硬件I2C有问题，但仍然有人实现出来）
再次启动程序，依旧是停在原来的位置
<!-- more -->
![这是一个图片](https://img-blog.csdn.net/20180814174512882?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMjgxNjAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

等待EV6，网上搜索相关问题好多人都停在了等待EV5上。分析EV5等待问题，主机发送起始信号，没能接受从设备发送的应答，

或者可能都没有发送。此问题应该是接线或IIC初始化代码的问题。

而我此时停在等待EV6，说明已检测到该设备。换句话说从设备已经知道了主设备的存在。却在主设备发送设备地址之后，接受不到从设备的应答信号，自己分析有两种可能，一是设备地址错误，从设备接受到不是自身的设备而地址自然不会应答。二是从设备已应答，而并接受到。在设备地址正确的前提下（前面已经通过例程验证过，前面的文章），思索第二种问题。

网上类似问题网友回应也是繁多，如RCC时钟初始化的问题，检查代码RCC_APB1Periph_I2C1，RCC_APB2Periph_GPIOB均已使能。又如可能i2c1默认引脚被复用，或该引脚没接上拉电阻，而不能开漏输出。将默认PIN6.PIN7重映射后依旧老样子排除该问题。

（重映射函数

```c
GPIO_PinRemapConfig(u32 GPIO_Remap, FunctionalState NewState)
```

）

再一次想到时钟问题，将IIC初始化结构体中的速率该低

原来是400000，也不行

直到在某论坛看到据说可以直接用的程序，打开发现代码并无差别，唯一不同的是，RCC初始化，其代码在主函数开始就初始化了所有的需要用到的时钟包括

,只是初始化位置不同。。。。

终于豁然开朗。

数据读取准确无误。

之后又将初始化程序恢复到原位置发现，程序仍可正确运行，不知何解。

尝试恢复速率发现，恢复成400000后不能读取，程序停滞在等待EV6

2000000等待EV5，1000000等待EV5

只能还设为100000，可以正常工作。

至此可确定STM32硬件I2C真的有问题！！！！

程序只能在10000输出，仅供参考

.h

```c
#ifndef __I2C_H__
#define __I2C_H__
 
#include "stm32f10x.h"
 
void I2C_GPIO_Config(void);
void I2C1_Init(void);
void I2C1_Write(u8 addr, u8 data);
u8 I2C1_Read(u8 nAddr);
#endif
```

.c

```c
#include "tb_delay.h"
#include "i2c.h"
void I2C_GPIO_Config()
{
GPIO_InitTypeDef GPIO_InitStructure;
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6 | GPIO_Pin_7;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
GPIO_Init(GPIOB, &GPIO_InitStructure);
}
 
void I2C1_Init()
{
I2C_InitTypeDef I2C_InitStructure;
RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C1, ENABLE);
I2C_DeInit(I2C1);
I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;
I2C_InitStructure.I2C_DutyCycle = I2C_DutyCycle_2;
I2C_InitStructure.I2C_OwnAddress1 = 0x77;
I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;
I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;
I2C_InitStructure.I2C_ClockSpeed = 10000;
I2C_Init(I2C1, &I2C_InitStructure);
I2C_Cmd(I2C1, ENABLE);
}
 
void I2C1_Write(u8 addr, u8 data)
{
I2C_AcknowledgeConfig(I2C1,ENABLE); 
I2C_GenerateSTART(I2C1,ENABLE); 
while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_MODE_SELECT)){;}//EV5
I2C_Send7bitAddress(I2C1,0x3a,I2C_Direction_Transmitter); 
while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED)){;} //EV6
I2C_SendData(I2C1,addr); 
while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED)){;} //EV8
I2C_SendData(I2C1,data);
while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED)){;} 
I2C_GenerateSTOP(I2C1,ENABLE); 
}
 
u8 I2C1_Read(u8 nAddr)
{
I2C_AcknowledgeConfig(I2C1,ENABLE); //????
I2C_GenerateSTART(I2C1,ENABLE); //???????
while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_MODE_SELECT)){;} //??EV5
I2C_Send7bitAddress(I2C1,0x3a,I2C_Direction_Transmitter); //????????
while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED)){;}//??EV6
I2C_SendData(I2C1,nAddr);//?????
while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_TRANSMITTED)){;} //??EV8
 
I2C_GenerateSTART(I2C1,ENABLE); //???????
while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_MODE_SELECT)){;} //??EV5
I2C_Send7bitAddress(I2C1,0x3a,I2C_Direction_Receiver); //???????
while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED)){;} //??EV6
I2C_AcknowledgeConfig(I2C1,DISABLE); //??????
I2C_GenerateSTOP(I2C1,ENABLE); //???????
while(!I2C_CheckEvent(I2C1,I2C_EVENT_MASTER_BYTE_RECEIVED)){;} //??EV7
return I2C_ReceiveData(I2C1); //???????
}
```