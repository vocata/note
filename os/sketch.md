# 主引导扇区

## 主引导扇区结构

- 代码：446B
- 硬盘分区表：64B
- 魔数：0x55 0xaa

## 主引导扇区功能

负责加载内核加载器，并执行。

# 硬盘读写 

## IDE/ATA PIO Mode

### 硬盘接口标准

- IDE（Integrated Drive Electronics）：IDE是早期的硬盘接口标准，最早出现在1986年。它使用40根线缆连接主板和硬盘驱动器，支持传输速率为8.3MB/s。IDE标准规定了硬盘驱动器的电路、信号传输和寻址方式等，并且将控制器和硬盘驱动器集成在一起，简化了硬盘的安装和配置。
- ATA（Advanced Technology Attachment）：ATA是IDE的升级版，最早出现在1994年。ATA标准提高了传输速率和数据容量，支持传输速率为16.6MB/s，并增加了多字节扇区传输和自动寻道等功能。ATA标准还引入了**PIO**（Programmed Input/Output）和**DMA**（Direct Memory Access）传输方式，使数据传输更加高效和可靠。
- PATA（Parallel ATA）：以并行传输方式实现的ATA接口标准，最早的ATA就是以这种方式实现的，也就是IDE。
- SATA（Serial ATA）：以串行传输方式实现的ATA接口标准，最早出现在2003年。相比于早期的并行实现IDE/ATA/PATA，其主要又是在于传输速率更高，电缆长度更长，兼容性更好，并且还支持热拔插。

关系： IDE是最早期的硬盘接口标准，使用了并行的传输方式。直到1994年，ANSI才发布ATA接口标准，所以早期的IDE也成为original ATA或者PATA。通常来说IDE=PATA=ATAT，而SATA则是基于串行传输方式的ATA，区别与前面三者。

### 数据传输方式

- PIO（Programmed Input/Output）：PIO是一种基于CPU的数据传输方式，每次传输数据时需要CPU的参与。在PIO模式下，CPU从设备控制器中读取数据，然后将数据写入内存或从内存中读取数据，再将数据写入设备控制器中。PIO模式的传输速率通常较慢，且会占用大量的CPU资源，因此在现代计算机系统中较少使用。
- DMA（Direct Memory Access）：DMA是一种基于设备控制器的数据传输方式，它可以在不需要CPU参与的情况下直接将数据传输到内存或从内存中读取数据。在DMA模式下，设备控制器直接访问内存中的数据，通过DMA控制器来控制数据传输。DMA模式的传输速率通常较快，且不会占用CPU资源，因此在现代计算机系统中广泛使用。

> 不同模式下一次性读取的扇区大小是有限制的。在PIO模式下，一次性最多读取256个扇区（$256*512=128$ KB），而DMA模式下可以读取更多。

关系： 两者都是ATA接口协议的一部分，定义了数据的传输方式。总的来说，PIO需要CPU参与数据传输过程，而DMA则直接交由设备来控制传输过程，因此DMA的传输效率更高，且节省CPU资源。

## 硬盘读写

### 硬盘访问模式

- CHS（Cylinder-Head-Sector）模式：使用原生的硬盘坐标（磁头，磁道，扇区）来访问硬盘数据。

    硬盘由柱面-磁头-扇区（CHS）组成，为了减少磁头的寻道次数，数据的读写顺序设计为扇区-柱面-磁头。另外，注意扇区的起始编号是从1开始，且最多只能读取256个扇区。

    参考：https://en.wikipedia.org/wiki/Cylinder-head-sector

- LBA模式（Logical Block Address）模式：将硬盘上的所有扇区地址看作一个连续的地址空间，从0开始编址，每个扇区被赋予一个唯一的逻辑块地址（LBA），程序可以直接使用LBA来访问硬盘数据而无需关心底层实际物理扇区的位置。

    常用的LBA模式有LBA28和LBA48，LAB28模式最大支持的硬盘容量为128GB（${(2^{28}-1)}\times{512}$ B），LBA28由于只能支持的存储空间过小，目前已逐渐被淘汰。现代计算机一般都是用LBA48的寻址模式，最大支持的硬盘容量为128PB。

### 硬盘控制端口

IDE控制器一般只支持2个IDE通道（primary和secondary），每个通道最多支持2个IDE设备（硬盘或光驱），因此IDE控制器最多只支持4个IDE设备。

以下是IDE控制器支持的I/O端口

| Primary | Secondary | In | Out | Bit |
| :-: | :-: | :-: | :-: | :-: |
| 0x1F0 | 0x170 | Data | Data | 16 |
| 0x1F1 | 0x171 | Error | Features | 8 |
| 0x1F2 | 0x172 | Sector Count | Sector Count | 8 |
| 0x1F3 | 0x173 | LBA Low | LBA Low | 8 |
| 0x1F4 | 0x174 | LBA Mid | LBA Mid | 8 |
| 0x1F5 | 0x175 | LBA High | LBA High | 8 |
| 0x1F6 | 0x176 | Device | Device | 8 |
| 0x1F7 | 0x177 | Status | Command | 8 |

- 0x1F0：读写数据
- 0x1F1：检测前一个指令的错误
- 0x1F2：指定读写的扇区数量
- 0x1F3：起始扇区的0 ~ 7位
- 0x1F4：起始扇区的8 ~ 15位
- 0x1F5：起始扇区的16 ~ 23位
- 0x1F6：
    - bit 0 ~ 3：起始扇区的24 ~ 27位
    - bit 4：0 master，1 slaver
    - bit 6：0 CHS，1 LBA
    - bit 5，7：固定为1
- 0x1F7：
    - out：0xEC 识别硬盘，0x20 读硬盘，0x30 写硬盘
    - in：
        - bit 0 ERR
        - bit 3 DRQ 数据准备完毕
