【MDK-ARM(AC5)】KEIL的MDK工程文件，uVision5版本ARM Compiler5
ARMFLY_STM32H7x_QSPI_W25Q256.FLM -  下载算法

/**********************************   下载案例    **********************************/
https://github.com/cturvey/stm32extldr 算法制作大全
https://forum.anfulai.cn/forum.php?mod=viewthread&tid=101586&highlight=w25q64 案例

/**********************************   下载算法    **********************************/
说明：整个案例是移植安富莱的工程


bsp_qspi_w25q256.c文件中代码改动
-->修改了对应的引脚   需要注意的有两点  
1、复用端口号 AF10还是AF9。
 2、修改完之后 要去修改他的
#define QSPI_CS_GPIO_CLK_ENABLE()       __HAL_RCC_GPIOB_CLK_ENABLE()
所对应的端口引脚使能

bsp_qspi_w25q256.h文件代码改动
---->修改了对应的引脚 （安富莱的工程是w25Q256 ）
我们要根据对应的Q64的数据手册进行更改。
改对应的define

FlashDev.c
修改了对应的FLASH大小 以及编程页大小（和擦除片区有关）
 完成以上即可使用w25Q64的算法FLM

struct FlashDevice const FlashDevice  =  {
    FLASH_DRV_VERS,                   /* 驱动版本，勿修改，这个是MDK定的 */
    "ARMFLY_STM32H7x_QSPI_W25Q64_TEST",   /* 算法名，添加算法到MDK安装目录会显示此名字 */
    EXTSPI,                           /* 设备类型 */
    0x90000000,                       /* Flash起始地址 */
    8 * 1024 * 1024,                 /* Flash大小，8MB */
    4 * 1024,                               /* 编程页大小 */
    0,                                /* 保留，必须为0 */
    0xFF,                             /* 擦除后的数值 */
    1000,                             /* 页编程等待时间 */
    6000,                             /* 扇区擦除等待时间 */
    64 * 1024, 0x000000,              /* 扇区大小，扇区地址 */
    SECTOR_END
};
#endif
				性能和稳定的关键
   			 4 * 1024,                               /* 编程页大小 */



