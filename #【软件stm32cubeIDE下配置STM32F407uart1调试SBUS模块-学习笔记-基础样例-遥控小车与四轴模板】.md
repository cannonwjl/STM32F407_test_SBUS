@[TOC](#【软件stm32cubeIDE下配置STM32F407uartt调试SBUS模块-学习笔记-基础样例-遥控小车与四轴模板】)

# 1、前言
最近一段时间在调试飞控遥控器模块，是基于SBUS，自己在裸机上跑通了，很多细节越值的注意，写这边文章也是给自己做个记录，保持初学者之心，这篇其实跟蓝牙那片有相似方式与文章结构。

# 2、实验环境以及器材
本次实验不是只买个蓝牙就能解决的，一般几乎每个人都有一部手机，但不一定没啥事带着一套飞控遥控器，而是市面上遥控器又五花八门，本篇针对特定遥控器开发，最文章最后会附上遥控器链接。

 1. 软件环境：STM32cubeIDE 1.8.0
 2. 硬件环境：STM32F407(正点原子探索者开发板)
 3. 一套遥控：HOTRC HT-6A航模遥控器接收机飞控套装 DIY自制6通道
 4. 下载模块：ST-link下载器 （下载器）
 5. 串口模块：串口转换器  （可用232模块代替）
 6. 硬件其它物品：  三极管8050，两个直插电阻4.7K(可以自行更换其它类型) 
实物照片：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c1ea2dcb101e21df41a1fe6b1547af97.png)

# 3、第一步：初步了解SBUS
###	（1）什么是sbus,简单说说
  翻了一段时间网上的文章后，总结，sbus就是基于串口的一套传输协议，就像我们平时使用9600波特率的串口一样，在使用它时，只是配置上稍微不同。打个比方，就好像铁轨上能跑绿色铁皮车，动车组，同样也能跑高级些的高铁列车，仔细理解这个比喻，铁轨没变，跑的东西，运输变了。
###	（2）硬件取反
这个很多文章都说了，要硬件取反，而且时必须硬件取反，软件只能反向数据位，不能反向停止位啥的，所以不要企图走捷径，还是老老实实去焊接个反相器吧。
###	（3）基本注意点
在使用sbus时候，和普通串口只有一点配置上的区别.

 - 波特率100K(100000)
 - 停止位 2个
 - 校验EVEN
 - sbus数据是：以0F开头，00为结束的。
如下图所示（已反向），串口助手打印出来的数据，以及配置，最好每行25bit,方便看。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d985dd3261f78d397538b29facfbc53.png)


# 4、第二步：制作硬件取反
###	（1）网上硬件图
这部分网上说挺多了，大部分都大同小异，只有电阻阻值的不同，都是用NPN的8050三级管来做，建议直接网上淘个，具体链接也放在后面了。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d85d30ff6738f77296974c7dc8e0abba.png)
###	（2）我的硬件图
以下出自我灵魂的画手，主要需要注意的是，三极管引脚，不同型号NPN三极管理论上都可以，但是买完后，一定要对下引脚图，要不可能就不好使，一脸懵逼，如下图所示。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ea0bf0e0bfa0e12dec29c28883fe1393.png)


# 5、第三步：接上串口试试，先不忙写代码
同之前测试蓝牙一样，焊借好了后，先不着急写代码，先看看有数据过来没有，接上串口，设置好配置，如下图，是实际逻辑分析仪抓取到的。上半部分是反向完成的数据，下部分是为反向数据。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c72b9341c74708e2e2790ed3232cce3d.png)

如下图，是串口抓取到数据。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/43edf90d5b65b0075b00fd80ac8a9c58.png)
你的串口工具基本稳定发送如上数据，那么恭喜你，你基本已经走完一半路程。

# 6、第四步：代码实验
完成第三步，可以进行下一步了，使用编译器写程序。
##### （1）软件基本配置，下载口和时钟
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/36f567b3ed474b853119ab3244fadb4a.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d5e5bed450ac29d9f090cd0d0608ada4.png)

##### （2）uart1的DMA等配置
1）波特率100000  9bit  EVEN  2STOP
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/042b8eb1bd85946ff4ead1e55cbb73b7.png)
2）DMA配置
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/30dc36689887ecf7c9cb6c9f3b5d000d.png)
3）中断配置
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f4c5190b50fc05aa4a0a83c886a61c9e.png)

##### （3）uart4的配置，用作输出显示。
1）这个想配置DMA就配置不想 普普通通也行，默认就行
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8848fc3fe4225a07e0ff0702094bee1a.png)
2）中断配置别忘了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dd8ea7d62ce03d3a52ef388a1aaedc7d.png)

##### （4）时钟配置，然后生成代码
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71fbdc15645a990e5ff3b7ba4fa1aea7.png)

##### （5）加入printf,输出显示
我们需要使用printf,帮我们输出一些信息，加入printf重定向，具体可以看我之前写的文章。
[https://waka-can.blog.csdn.net/article/details/124452661?spm=1001.2014.3001.5502](https://waka-can.blog.csdn.net/article/details/124452661?spm=1001.2014.3001.5502)
简单说，mian.c加入如下代码

```c
#include "stdio.h"
#include "stdint.h"

/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */

/* USER CODE END Includes */
//>>第二步：定义数组
uint8_t u_buf[64];

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */
//>>第三步：定义输出函数printf
#define printf(...)  HAL_UART_Transmit((UART_HandleTypeDef * )&huart4, (uint8_t *)u_buf,\
											sprintf((char *)u_buf,__VA_ARGS__),0x200);
```

##### （6）加入uart1的DMA等回调函数
具体代码我就不一一列举了，如果需要直接去我代码里去，觉得CSDN要积分的话，可以私信我要代码。
（1）初始化声明下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/61b89ce1b98c0e668507d06c53eaa8d7.png)

（2）回调函数
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9acd8dfc204aed7a270f8a00ac9c91fa.png)
（3）加入中断函数内
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff12a8325455200c066c61207018575f.png)（4）==加入转化函数==：通用代码

```c
void get_sbus_data(uint8_t sbus_data[27], uint16_t sbus_channel[16])
{
		sbus_channel[0]  = ((sbus_data[1]|sbus_data[2]<<8) & 0x07FF);
		sbus_channel[1]  = ((sbus_data[2]>>3 |sbus_data[3]<<5) & 0x07FF);
		sbus_channel[2]  = ((sbus_data[3]>>6 |sbus_data[4]<<2 |sbus_data[5]<<10) & 0x07FF);
		sbus_channel[3]  = ((sbus_data[5]>>1 |sbus_data[6]<<7) & 0x07FF);
		sbus_channel[4]  = ((sbus_data[6]>>4 |sbus_data[7]<<4) & 0x07FF);
		sbus_channel[5]  = ((sbus_data[7]>>7 |sbus_data[8]<<1 |sbus_data[9]<<9) & 0x07FF);
		sbus_channel[6]  = ((sbus_data[9]>>2 |sbus_data[10]<<6) & 0x07FF);
		sbus_channel[7]  = ((sbus_data[10]>>5|sbus_data[11]<<3) & 0x07FF);
		sbus_channel[8]  = ((sbus_data[12]   |sbus_data[13]<<8) & 0x07FF);
		sbus_channel[9]  = ((sbus_data[13]>>3|sbus_data[14]<<5) & 0x07FF);
		sbus_channel[10] = ((sbus_data[14]>>6|sbus_data[15]<<2|sbus_data[16]<<10) & 0x07FF);
		sbus_channel[11] = ((sbus_data[16]>>1|sbus_data[17]<<7) & 0x07FF);
		sbus_channel[12] = ((sbus_data[17]>>4|sbus_data[18]<<4) & 0x07FF);
		sbus_channel[13] = ((sbus_data[18]>>7|sbus_data[19]<<1|sbus_data[20]<<9)& 0x07FF);
		sbus_channel[14] = ((sbus_data[20]>>2|sbus_data[21]<<6) & 0x07FF);
		sbus_channel[15] = ((sbus_data[21]>>5|sbus_data[22]<<3) & 0x07FF);

}
```


##### （7）打印输出显示。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cde243ee8fe7c63da85825ec4f8baedf.png)

# 代码：实验代码连接
具体本次代码连接：[https://download.csdn.net/download/qq_22146161/85826413](https://download.csdn.net/download/qq_22146161/85826413)
# 8、实际效果演示
这个视频还是放在B站了，有兴趣可以看下。
[https://www.bilibili.com/video/BV15G411x7TQ/](https://www.bilibili.com/video/BV15G411x7TQ/)
# 9、后期细节
##	（1）接收长度27问题
查看他人编写的代码时发现，有uart1接收长度为27位，这个应该都行，只要能顺利接收进来就行，也没必要给太长。
##	（2）判断断联问题，数据接收位【23】位
查看资料时，发现有个判断断联标志位，这个设计非常好，如果在四轴上，一旦的断联，基本是损害物品，所以这个位需要注意下，最好查查这个位怎么取，怎么用。
# 10、参考连接
###	（1）硬件取反图，以及基础知识了解
自己开始也是小白，从他人哪里获取知识，当然需要标明从哪里获取。
参考连接：[https://blog.csdn.net/peach_orange/article/details/52958385](https://blog.csdn.net/peach_orange/article/details/52958385)
参考连接：[https://blog.csdn.net/ReadAir/article/details/102631513](https://blog.csdn.net/ReadAir/article/details/102631513)
###	（2）其它人样例代码
如果你想要keil版本的，在正点原子社区有人做了，自己验证过，可以的。
参考连接：[http://www.openedv.com/forum.php?mod=viewthread&tid=332860](http://www.openedv.com/forum.php?mod=viewthread&tid=332860)
# 11、硬件连接
###	（1）三极管以及电阻
网上某宝非常多：[8050三极管等](https://detail.tmall.com/item.htm?spm=a230r.1.14.16.45ff53fbc3cnba&id=14574984671&ns=1&abbucket=4)
###	（2）飞控遥控器
网上某宝非常多：[飞控遥控：HOTRC HT-6A航模遥控器接收机飞控套装 DIY自制6通道](https://item.taobao.com/item.htm?id=665318299342&price=55-178&sourceType=item&sourceType=item&suid=68bc6551-8a70-497d-a77a-8bf05950c695&shareUniqueId=16060481500&ut_sk=1.XhXWjDU7SWEDAFwlErjrw1MX_21646297_1652859151319.Copy.1&un=4312cec1ca329ab08bd7bab98ffeb263&share_crt_v=1&un_site=0&spm=a2159r.13376460.0.0&sp_abtk=common_1_commonInfo&tbSocialPopKey=shareItem&sp_tk=eldKMjJraWIza1g=&cpp=1&shareurl=true&short_name=h.fHrkS3A&bxsign=scdR0qvxl_eVP8BhWnFu6Ue5595nkDjcZtvzoBmk7qXwTdFrTXN7N8y9iAbBlvu_mI15sB-Dk1SCg6cs4H-CfHFXccOqKEI7gTQ0tgpdmL3HIC2AqW8tlg3TzQUYLmfB5u_&tk=zWJ22kib3kX&app=chrome)

