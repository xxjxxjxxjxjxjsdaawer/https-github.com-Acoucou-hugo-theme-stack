+++
author = "coucou"
title = "操作系统——imx6ull"
date = "2023-08-01"
description = "操作系统专题之imx6ull"
categories = [
    "操作系统"
]
tags = [
    "操作系统","imx6ull"
]
+++
![](1.jpg)


## Linux学习（基于正点原子I.MX6U）

### 常用linux命令

```shell
lsmod
rmmod
depmod	# 一次加载驱动的时候需要运行此命令
modprobe led.ko  # 加载驱动
mknod /dev/led c 200 0  #创建节点、
rmmod led.ko  # 卸载驱动
ls /dev/newchrled -l  # 查看设备

cd proc/device-tree  #查看设备树节点信息
cd /sys/bus/platform/drivers/  # 查看platform节点
```

### 系统启动方式

>I.MX6U 支持多种启动方式以及启动设备，比如可以从 SD/EMMC、NAND Flash、QSPI Flash 等启动。
>
>用户可以根据实际情况，选择合适的启动设备。不同的启动方式其启动方式和启动要求也不一样，比如上一章中的从 SD 卡启动就需要在 bin 文件前面添加一个数据头，其它的启 动设备也是需要这个数据头的。
>
>**启动**
>
>1 2 3 4 5 6 7 8 启动设备
>
>0 1 x x x x x x 串行下载，可以通过 USB 烧写镜像文件
>
>1 0 0 0 0 0 1 0 SD 卡启动
>
>1 0 1 0 0 1 1 0 EMMC 启动      
>
>1 0 0 0 1 0 0 1 NAND FLASH 启动

```basic
setenv bootcmd 'fatload mmc 1:1 80800000 zImage;fatload mmc 1:1 83000000 imx6ull-14x14-emmc-7-1024x600-c.dtb;bootz 80800000 - 83000000;'

setenv bootargs 'console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw'
```

### Linux系统

#### uboot

```shell
1、将 imxdownload 软件拷贝到 uboot 源码根目录下，然后使用 imxdownload 软件将 u-boot.bin
烧写到 SD 卡中，烧写命令如下：
    chmod 777 imxdownload //给予 imxdownload 可执行权限
    ./imxdownload u-boot.bin /dev/sdd //烧写到 SD 卡中，不能烧写到/dev/sda 或 sda1 里面
2、烧写完成以后将 SD 卡插入 I.MX6U-ALPHA 开发板的 TF 卡槽中，最后设置开发板从 SD卡启动。
```

##### 常用命令

```shell
bdinfo # 查看板子信息
printenv # 输出环境变量
version # 版本

# 环境变量操作
setenv bootdelay 5  # 设置环境变量
saveenv  # 保存
setenv bootargs 'console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw'
saveenv

# 内存操作命令
md # 显示内存值
nm # 修改指定地址的内存值
mm # 修改指定地址内存值
mw # 使用一个指定的数据填充一段内存
cp # 将 DRAM 中的数据从一段内存拷贝到另一段内存中
cmp # 比较两段内存的数据是否相等

# 网络操作命令
ping  192.168.1.253
setenv ipaddr 192.168.1.50 # 开发板 ip 地址
setenv ethaddr b8:ae:1d:01:00:00  # mac地址
setenv gatewayip 192.168.1.1
setenv netmask 255.255.255.0
setenv serverip 192.168.1.253 # 服务器 IP 地址
saveenv
dhcp # 从路由器获取 IP 地址
nfs 80800000 192.168.1.253:/home/zuozhongkai/linux/nfs/zImage # 将 zImage 下载到开发板 DRAM 的 0X80800000 地址处

# EMMC 和 SD 卡操作命令
mmc info 输出 MMC 设备信息
mmc read 读取 MMC 中的数据
……

#  NAND 操作命令
nand info 命令
……

# BOOT 操作命令
tftp 80800000 zImage
tftp 83000000 imx6ull-alientek-emmc.dtb
bootz 80800000 - 83000000  # 启动 zImage 镜像文件

setenv bootcmd 'tftp 80800000 zImage; tftp 83000000 imx6ull-alientek-emmc.dtb;'
bootz 80800000 - 83000000
setenv bootargs 'console=ttymxc0,115200 root=/dev/nfs nfsroot=192.168.2.30:/home/coucou/nfs/rootfs,proto=tcp rw ip=192.168.2.10:192.168.2.30:192.168.2.1:255.255.255.0::eth0:off'

saveenv
boot  # 来启动 Linux 系统

# 其他常用命令
reset  # 重启
go  # 跳到指定的地址处执行应用
run  # 用于运行环境变量中定义的命令
```

#### ZImage和设备树

```
使用tftp服务器挂载测试
```

#### rootfs根文件系统

```
编译busybox
```

#### 烧写系统到emmc

```
使用MfgTool工具
```

### linux驱动开发

#### 点灯

>**地址映射**
>
>物理内存和虚拟内存映射    **ioremap()**映射函数  和  **ioremap()**取消映射函数

##### API

```c
// 将用户空间的数据复制到 writebuf 内核空间中
copy_from_user();

// 申请设备号
int alloc_chrdev_region(dev_t *dev, unsigned baseminor, unsigned count, const char *name)
// 注册设备号
int register_chrdev_region(dev_t from, unsigned count, const char *name)
// 释放设备号
void unregister_chrdev_region(dev_t from, unsigned count)
    
// cdev 结构体表示一个字符设备
struct cdev {
    struct kobject kobj;
    struct module *owner;
    const struct file_operations *ops;  // 文件操作
    struct list_head list;
    dev_t dev;  // 设备号
    unsigned int count;
};
// cdev初始化和添加设备
void cdev_init(struct cdev *cdev, const struct file_operations *fops)
int cdev_add(struct cdev *p, dev_t dev, unsigned count)
void cdev_del(struct cdev *p)  // 删除设备
    
// 自动创建节点
struct class *class_create (struct module *owner, const char *name)  // 创建类
void class_destroy(struct class *cls);  // 删除类
struct device *device_create(  // 创建设备
    struct class *class, 
    struct device *parent,
    dev_t devt, 
    void *drvdata, 
    const char *fmt, ...
)
void device_destroy(struct class *class, dev_t devt)  // 删除设备

// 加载驱动和卸载驱动
module_init();
module_exit();
```

##### 示例

```c
#include <linux/types.h>
#include <linux/kernel.h>
#include <linux/delay.h>
#include <linux/ide.h>
#include <linux/init.h>
#include <linux/module.h>
#include <linux/errno.h>
#include <linux/gpio.h>
#include <linux/cdev.h>
#include <linux/device.h>

#include <asm/mach/map.h>
#include <asm/uaccess.h>
#include <asm/io.h>

/***************************************************************
Copyright © ALIENTEK Co., Ltd. 1998-2029. All rights reserved.
文件名		: newchrled.c
作者	  	: 左忠凯
版本	   	: V1.0
描述	   	: LED驱动文件。
其他	   	: 无
论坛 	   	: www.openedv.com
日志	   	: 初版V1.0 2019/6/27 左忠凯创建
***************************************************************/
#define NEWCHRLED_CNT			1		  	/* 设备号个数 */
#define NEWCHRLED_NAME			"newchrled"	/* 名字 */
#define LEDOFF 					0			/* 关灯 */
#define LEDON 					1			/* 开灯 */
 
/* 寄存器物理地址 */
#define CCM_CCGR1_BASE				(0X020C406C)	
#define SW_MUX_GPIO1_IO03_BASE		(0X020E0068)
#define SW_PAD_GPIO1_IO03_BASE		(0X020E02F4)
#define GPIO1_DR_BASE				(0X0209C000)
#define GPIO1_GDIR_BASE				(0X0209C004)

/* 映射后的寄存器虚拟地址指针 */
static void __iomem *IMX6U_CCM_CCGR1;
static void __iomem *SW_MUX_GPIO1_IO03;
static void __iomem *SW_PAD_GPIO1_IO03;
static void __iomem *GPIO1_DR;
static void __iomem *GPIO1_GDIR;

/* newchrled设备结构体 */
struct newchrled_dev{
	dev_t devid;			/* 设备号 	 */
	struct cdev cdev;		/* cdev 	*/
	struct class *class;		/* 类 		*/
	struct device *device;	/* 设备 	 */
	int major;				/* 主设备号	  */
	int minor;				/* 次设备号   */
};

struct newchrled_dev newchrled;	/* led设备 */

/*
 * @description		: LED打开/关闭
 * @param - sta 	: LEDON(0) 打开LED，LEDOFF(1) 关闭LED
 * @return 			: 无
 */
void led_switch(u8 sta)
{
	u32 val = 0;
	if(sta == LEDON) {
		val = readl(GPIO1_DR);
		val &= ~(1 << 3);	
		writel(val, GPIO1_DR);
	}else if(sta == LEDOFF) {
		val = readl(GPIO1_DR);
		val|= (1 << 3);	
		writel(val, GPIO1_DR);
	}	
}

/*
 * @description		: 打开设备
 * @param - inode 	: 传递给驱动的inode
 * @param - filp 	: 设备文件，file结构体有个叫做private_data的成员变量
 * 					  一般在open的时候将private_data指向设备结构体。
 * @return 			: 0 成功;其他 失败
 */
static int led_open(struct inode *inode, struct file *filp)
{
	filp->private_data = &newchrled; /* 设置私有数据 */
	return 0;
}

/*
 * @description		: 从设备读取数据 
 * @param - filp 	: 要打开的设备文件(文件描述符)
 * @param - buf 	: 返回给用户空间的数据缓冲区
 * @param - cnt 	: 要读取的数据长度
 * @param - offt 	: 相对于文件首地址的偏移
 * @return 			: 读取的字节数，如果为负值，表示读取失败
 */
static ssize_t led_read(struct file *filp, char __user *buf, size_t cnt, loff_t *offt)
{
	return 0;
}

/*
 * @description		: 向设备写数据 
 * @param - filp 	: 设备文件，表示打开的文件描述符
 * @param - buf 	: 要写给设备写入的数据
 * @param - cnt 	: 要写入的数据长度
 * @param - offt 	: 相对于文件首地址的偏移
 * @return 			: 写入的字节数，如果为负值，表示写入失败
 */
static ssize_t led_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *offt)
{
	int retvalue;
	unsigned char databuf[1];
	unsigned char ledstat;

	retvalue = copy_from_user(databuf, buf, cnt);
	if(retvalue < 0) {
		printk("kernel write failed!\r\n");
		return -EFAULT;
	}

	ledstat = databuf[0];		/* 获取状态值 */

	if(ledstat == LEDON) {	
		led_switch(LEDON);		/* 打开LED灯 */
	} else if(ledstat == LEDOFF) {
		led_switch(LEDOFF);	/* 关闭LED灯 */
	}
	return 0;
}

/*
 * @description		: 关闭/释放设备
 * @param - filp 	: 要关闭的设备文件(文件描述符)
 * @return 			: 0 成功;其他 失败
 */
static int led_release(struct inode *inode, struct file *filp)
{
	return 0;
}

/* 设备操作函数 */
static struct file_operations newchrled_fops = {
	.owner = THIS_MODULE,
	.open = led_open,
	.read = led_read,
	.write = led_write,
	.release = 	led_release,
};

/*
 * @description	: 驱动出口函数
 * @param 		: 无
 * @return 		: 无
 */
static int __init led_init(void)
{
	u32 val = 0;

	/* 初始化LED */
	/* 1、寄存器地址映射 */
  	IMX6U_CCM_CCGR1 = ioremap(CCM_CCGR1_BASE, 4);
	SW_MUX_GPIO1_IO03 = ioremap(SW_MUX_GPIO1_IO03_BASE, 4);
  	SW_PAD_GPIO1_IO03 = ioremap(SW_PAD_GPIO1_IO03_BASE, 4);
	GPIO1_DR = ioremap(GPIO1_DR_BASE, 4);
	GPIO1_GDIR = ioremap(GPIO1_GDIR_BASE, 4);

	/* 2、使能GPIO1时钟 */
	val = readl(IMX6U_CCM_CCGR1);
	val &= ~(3 << 26);	/* 清楚以前的设置 */
	val |= (3 << 26);	/* 设置新值 */
	writel(val, IMX6U_CCM_CCGR1);

	/* 3、设置GPIO1_IO03的复用功能，将其复用为
	 *    GPIO1_IO03，最后设置IO属性。
	 */
	writel(5, SW_MUX_GPIO1_IO03);
	
	/*寄存器SW_PAD_GPIO1_IO03设置IO属性
	 *bit 16:0 HYS关闭
	 *bit [15:14]: 00 默认下拉
     *bit [13]: 0 kepper功能
     *bit [12]: 1 pull/keeper使能
     *bit [11]: 0 关闭开路输出
     *bit [7:6]: 10 速度100Mhz
     *bit [5:3]: 110 R0/6驱动能力
     *bit [0]: 0 低转换率
	 */
	writel(0x10B0, SW_PAD_GPIO1_IO03);

	/* 4、设置GPIO1_IO03为输出功能 */
	val = readl(GPIO1_GDIR);
	val &= ~(1 << 3);	/* 清除以前的设置 */
	val |= (1 << 3);	/* 设置为输出 */
	writel(val, GPIO1_GDIR);

	/* 5、默认关闭LED */
	val = readl(GPIO1_DR);
	val |= (1 << 3);	
	writel(val, GPIO1_DR);

	/* 注册字符设备驱动 */
	/* 1、创建设备号 */
	if (newchrled.major) {		/*  定义了设备号 */
		newchrled.devid = MKDEV(newchrled.major, 0);
		register_chrdev_region(newchrled.devid, NEWCHRLED_CNT, NEWCHRLED_NAME);
	} else {						/* 没有定义设备号 */
		alloc_chrdev_region(&newchrled.devid, 0, NEWCHRLED_CNT, NEWCHRLED_NAME);	/* 申请设备号 */
		newchrled.major = MAJOR(newchrled.devid);	/* 获取分配号的主设备号 */
		newchrled.minor = MINOR(newchrled.devid);	/* 获取分配号的次设备号 */
	}
	printk("newcheled major=%d,minor=%d\r\n",newchrled.major, newchrled.minor);	
	
	/* 2、初始化cdev */
	newchrled.cdev.owner = THIS_MODULE;
	cdev_init(&newchrled.cdev, &newchrled_fops);
	
	/* 3、添加一个cdev */
	cdev_add(&newchrled.cdev, newchrled.devid, NEWCHRLED_CNT);

	/* 4、创建类 */
	newchrled.class = class_create(THIS_MODULE, NEWCHRLED_NAME);
	if (IS_ERR(newchrled.class)) {
		return PTR_ERR(newchrled.class);
	}

	/* 5、创建设备 */
	newchrled.device = device_create(newchrled.class, NULL, newchrled.devid, NULL, NEWCHRLED_NAME);
	if (IS_ERR(newchrled.device)) {
		return PTR_ERR(newchrled.device);
	}
	
	return 0;
}

/*
 * @description	: 驱动出口函数
 * @param 		: 无
 * @return 		: 无
 */
static void __exit led_exit(void)
{
	/* 取消映射 */
	iounmap(IMX6U_CCM_CCGR1);
	iounmap(SW_MUX_GPIO1_IO03);
	iounmap(SW_PAD_GPIO1_IO03);
	iounmap(GPIO1_DR);
	iounmap(GPIO1_GDIR);

	/* 注销字符设备驱动 */
	cdev_del(&newchrled.cdev);/*  删除cdev */
	unregister_chrdev_region(newchrled.devid, NEWCHRLED_CNT); /* 注销设备号 */

	device_destroy(newchrled.class, newchrled.devid);
	class_destroy(newchrled.class);
}

module_init(led_init);
module_exit(led_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("zuozhongkai");
```

#### 设备树点灯

>**路径** ： /arch/arm/boot/dts
>
>**设备树节点命名**  **node-name@unit-address**
>
>​	“unit-address”一般表示设备的地址或寄存器首地址
>
>**节点属性**
>
>1. **compatible** 属性：“兼容性”属性，用于将设备和驱动绑定起来
>
>2. **model** 属性：描述设备模块信息
>
>3. **status** 属性：设备状态
>
>4. **\#address-cells** 和  **#size-cells** 属性：表明子节点应该如何编写 reg 属性值
>
>5. **reg** 属性：用于描述设备地址空间资源信息，一般都是某个外设的寄存器地址范围信息
>6. **ranges** 属性：ranges 是一个地址映射/转换表，ranges 属性每个项目由子地址、父地址和地址空间长度组成
>7. **name** 属性：用于记录节点名字
>8. **device_type** 属性：用于描述设备的 FCode
>
>**节点内容追加**
>
>&i2c1{ 
>
>​		/* 要追加或修改的内容 */ 
>
>};    // 在I2C节点下追加内容
>
>**单独编译设备树**：make dtbs
>
>

##### API

```c
// 通过节点名字查找指定的节点
struct device_node *of_find_node_by_name(struct device_node *from, const char *name);
// 通过 device_type 属性查找指定的节点
struct device_node *of_find_node_by_type(struct device_node *from, const char *type)
// device_type 和 compatible 这两个属性查找指定的节点
struct device_node *of_find_compatible_node(struct device_node *from, const char *type, const char *compatible)
// 通过 of_device_id 匹配表来查找指定的节点
struct device_node *of_find_matching_node_and_match(
    struct device_node *from,
    const struct of_device_id *matches,
 	const struct of_device_id **match)
// 通过路径来查找指定的节点
inline struct device_node *of_find_node_by_path(const char *path)

// 获取指定节点的父节点
struct device_node *of_get_parent(const struct device_node *node)
// 查找子节点
struct device_node *of_get_next_child(const struct device_node *node, struct device_node *prev)
    
// 查找指定的属性
property *of_find_property(const struct device_node *np, const char *name, int *lenp)
// 获取属性中元素的数量
int of_property_count_elems_of_size(const struct device_node *np,const char *propname,int elem_size)
// 读取整形值的属性
int of_property_read_u32(const struct device_node *np, const char *propname, u32 *out_value)

// 读取属性中字符串值
int of_property_read_string(struct device_node *np, const char *propname, const char **out_string)
```

##### 示例

```c
// 再该文件 imx6ull-alientek-emmc.dts 下添加
alphaled {
                #address-cells = <1>;
                #size-cells = <1>;
                compatible = "atkalpha-led";
                status = "okay";
                reg = < 0X020C406C 0X04         		/* CCM_CCGR1_BAE                        */
                        0X020E0068 0X04         /* SW_MUX_GPIO1_IO03_BASE       */
                        0X020E02F4 0X04         /* SW_PAD_GPIO1_IO03_BASE       */
                        0X0209C000 0X04         /* GPIO1_DR_BASE                        */
                        0X0209C004 0X04>;       /* GPIO1_GDIR_BASE                      */
        };

// 驱动代码
#include <linux/types.h>
#include <linux/kernel.h>
#include <linux/delay.h>
#include <linux/ide.h>
#include <linux/init.h>
#include <linux/module.h>
#include <linux/errno.h>
#include <linux/gpio.h>
#include <linux/cdev.h>
#include <linux/device.h>
#include <linux/of.h>
#include <linux/of_address.h>
#include <asm/mach/map.h>
#include <asm/uaccess.h>
#include <asm/io.h>

/***************************************************************
Copyright © ALIENTEK Co., Ltd. 1998-2029. All rights reserved.
文件名		: dtsled.c
作者	  	: 左忠凯
版本	   	: V1.0
描述	   	: LED驱动文件。
其他	   	: 无
论坛 	   	: www.openedv.com
日志	   	: 初版V1.0 2019/7/9 左忠凯创建
***************************************************************/
#define DTSLED_CNT			1		  	/* 设备号个数 */
#define DTSLED_NAME			"dtsled"	/* 名字 */
#define LEDOFF 					0			/* 关灯 */
#define LEDON 					1			/* 开灯 */

/* 映射后的寄存器虚拟地址指针 */
static void __iomem *IMX6U_CCM_CCGR1;
static void __iomem *SW_MUX_GPIO1_IO03;
static void __iomem *SW_PAD_GPIO1_IO03;
static void __iomem *GPIO1_DR;
static void __iomem *GPIO1_GDIR;

/* dtsled设备结构体 */
struct dtsled_dev{
	dev_t devid;			/* 设备号 	 */
	struct cdev cdev;		/* cdev 	*/
	struct class *class;		/* 类 		*/
	struct device *device;	/* 设备 	 */
	int major;				/* 主设备号	  */
	int minor;				/* 次设备号   */
	struct device_node	*nd; /* 设备节点 */
};

struct dtsled_dev dtsled;	/* led设备 */

/*
 * @description		: LED打开/关闭
 * @param - sta 	: LEDON(0) 打开LED，LEDOFF(1) 关闭LED
 * @return 			: 无
 */
void led_switch(u8 sta)
{
	u32 val = 0;
	if(sta == LEDON) {
		val = readl(GPIO1_DR);
		val &= ~(1 << 3);	
		writel(val, GPIO1_DR);
	}else if(sta == LEDOFF) {
		val = readl(GPIO1_DR);
		val|= (1 << 3);	
		writel(val, GPIO1_DR);
	}	
}

/*
 * @description		: 打开设备
 * @param - inode 	: 传递给驱动的inode
 * @param - filp 	: 设备文件，file结构体有个叫做private_data的成员变量
 * 					  一般在open的时候将private_data指向设备结构体。
 * @return 			: 0 成功;其他 失败
 */
static int led_open(struct inode *inode, struct file *filp)
{
	filp->private_data = &dtsled; /* 设置私有数据 */
	return 0;
}

/*
 * @description		: 从设备读取数据 
 * @param - filp 	: 要打开的设备文件(文件描述符)
 * @param - buf 	: 返回给用户空间的数据缓冲区
 * @param - cnt 	: 要读取的数据长度
 * @param - offt 	: 相对于文件首地址的偏移
 * @return 			: 读取的字节数，如果为负值，表示读取失败
 */
static ssize_t led_read(struct file *filp, char __user *buf, size_t cnt, loff_t *offt)
{
	return 0;
}

/*
 * @description		: 向设备写数据 
 * @param - filp 	: 设备文件，表示打开的文件描述符
 * @param - buf 	: 要写给设备写入的数据
 * @param - cnt 	: 要写入的数据长度
 * @param - offt 	: 相对于文件首地址的偏移
 * @return 			: 写入的字节数，如果为负值，表示写入失败
 */
static ssize_t led_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *offt)
{
	int retvalue;
	unsigned char databuf[1];
	unsigned char ledstat;

	retvalue = copy_from_user(databuf, buf, cnt);
	if(retvalue < 0) {
		printk("kernel write failed!\r\n");
		return -EFAULT;
	}

	ledstat = databuf[0];		/* 获取状态值 */

	if(ledstat == LEDON) {	
		led_switch(LEDON);		/* 打开LED灯 */
	} else if(ledstat == LEDOFF) {
		led_switch(LEDOFF);	/* 关闭LED灯 */
	}
	return 0;
}

/*
 * @description		: 关闭/释放设备
 * @param - filp 	: 要关闭的设备文件(文件描述符)
 * @return 			: 0 成功;其他 失败
 */
static int led_release(struct inode *inode, struct file *filp)
{
	return 0;
}

/* 设备操作函数 */
static struct file_operations dtsled_fops = {
	.owner = THIS_MODULE,
	.open = led_open,
	.read = led_read,
	.write = led_write,
	.release = 	led_release,
};

/*
 * @description	: 驱动出口函数
 * @param 		: 无
 * @return 		: 无
 */
static int __init led_init(void)
{
	u32 val = 0;
	int ret;
	u32 regdata[14];
	const char *str;
	struct property *proper;

	/* 获取设备树中的属性数据 */
	/* 1、获取设备节点：alphaled */
	dtsled.nd = of_find_node_by_path("/alphaled");
	if(dtsled.nd == NULL) {
		printk("alphaled node nost find!\r\n");
		return -EINVAL;
	} else {
		printk("alphaled node find!\r\n");
	}

	/* 2、获取compatible属性内容 */
	proper = of_find_property(dtsled.nd, "compatible", NULL);
	if(proper == NULL) {
		printk("compatible property find failed\r\n");
	} else {
		printk("compatible = %s\r\n", (char*)proper->value);
	}

	/* 3、获取status属性内容 */
	ret = of_property_read_string(dtsled.nd, "status", &str);
	if(ret < 0){
		printk("status read failed!\r\n");
	} else {
		printk("status = %s\r\n",str);
	}

	/* 4、获取reg属性内容 */
	ret = of_property_read_u32_array(dtsled.nd, "reg", regdata, 10);
	if(ret < 0) {
		printk("reg property read failed!\r\n");
	} else {
		u8 i = 0;
		printk("reg data:\r\n");
		for(i = 0; i < 10; i++)
			printk("%#X ", regdata[i]);
		printk("\r\n");
	}

	/* 初始化LED */
#if 0
	/* 1、寄存器地址映射 */
	IMX6U_CCM_CCGR1 = ioremap(regdata[0], regdata[1]);
	SW_MUX_GPIO1_IO03 = ioremap(regdata[2], regdata[3]);
  	SW_PAD_GPIO1_IO03 = ioremap(regdata[4], regdata[5]);
	GPIO1_DR = ioremap(regdata[6], regdata[7]);
	GPIO1_GDIR = ioremap(regdata[8], regdata[9]);
#else
	IMX6U_CCM_CCGR1 = of_iomap(dtsled.nd, 0);
	SW_MUX_GPIO1_IO03 = of_iomap(dtsled.nd, 1);
  	SW_PAD_GPIO1_IO03 = of_iomap(dtsled.nd, 2);
	GPIO1_DR = of_iomap(dtsled.nd, 3);
	GPIO1_GDIR = of_iomap(dtsled.nd, 4);
#endif

	/* 2、使能GPIO1时钟 */
	val = readl(IMX6U_CCM_CCGR1);
	val &= ~(3 << 26);	/* 清楚以前的设置 */
	val |= (3 << 26);	/* 设置新值 */
	writel(val, IMX6U_CCM_CCGR1);

	/* 3、设置GPIO1_IO03的复用功能，将其复用为
	 *    GPIO1_IO03，最后设置IO属性。
	 */
	writel(5, SW_MUX_GPIO1_IO03);
	
	/*寄存器SW_PAD_GPIO1_IO03设置IO属性
	 *bit 16:0 HYS关闭
	 *bit [15:14]: 00 默认下拉
     *bit [13]: 0 kepper功能
     *bit [12]: 1 pull/keeper使能
     *bit [11]: 0 关闭开路输出
     *bit [7:6]: 10 速度100Mhz
     *bit [5:3]: 110 R0/6驱动能力
     *bit [0]: 0 低转换率
	 */
	writel(0x10B0, SW_PAD_GPIO1_IO03);

	/* 4、设置GPIO1_IO03为输出功能 */
	val = readl(GPIO1_GDIR);
	val &= ~(1 << 3);	/* 清除以前的设置 */
	val |= (1 << 3);	/* 设置为输出 */
	writel(val, GPIO1_GDIR);

	/* 5、默认关闭LED */
	val = readl(GPIO1_DR);
	val |= (1 << 3);	
	writel(val, GPIO1_DR);

	/* 注册字符设备驱动 */
	/* 1、创建设备号 */
	if (dtsled.major) {		/*  定义了设备号 */
		dtsled.devid = MKDEV(dtsled.major, 0);
		register_chrdev_region(dtsled.devid, DTSLED_CNT, DTSLED_NAME);
	} else {						/* 没有定义设备号 */
		alloc_chrdev_region(&dtsled.devid, 0, DTSLED_CNT, DTSLED_NAME);	/* 申请设备号 */
		dtsled.major = MAJOR(dtsled.devid);	/* 获取分配号的主设备号 */
		dtsled.minor = MINOR(dtsled.devid);	/* 获取分配号的次设备号 */
	}
	printk("dtsled major=%d,minor=%d\r\n",dtsled.major, dtsled.minor);	
	
	/* 2、初始化cdev */
	dtsled.cdev.owner = THIS_MODULE;
	cdev_init(&dtsled.cdev, &dtsled_fops);
	
	/* 3、添加一个cdev */
	cdev_add(&dtsled.cdev, dtsled.devid, DTSLED_CNT);

	/* 4、创建类 */
	dtsled.class = class_create(THIS_MODULE, DTSLED_NAME);
	if (IS_ERR(dtsled.class)) {
		return PTR_ERR(dtsled.class);
	}

	/* 5、创建设备 */
	dtsled.device = device_create(dtsled.class, NULL, dtsled.devid, NULL, DTSLED_NAME);
	if (IS_ERR(dtsled.device)) {
		return PTR_ERR(dtsled.device);
	}
	
	return 0;
}

/*
 * @description	: 驱动出口函数
 * @param 		: 无
 * @return 		: 无
 */
static void __exit led_exit(void)
{
	/* 取消映射 */
	iounmap(IMX6U_CCM_CCGR1);
	iounmap(SW_MUX_GPIO1_IO03);
	iounmap(SW_PAD_GPIO1_IO03);
	iounmap(GPIO1_DR);
	iounmap(GPIO1_GDIR);

	/* 注销字符设备驱动 */
	cdev_del(&dtsled.cdev);/*  删除cdev */
	unregister_chrdev_region(dtsled.devid, DTSLED_CNT); /* 注销设备号 */

	device_destroy(dtsled.class, dtsled.devid);
	class_destroy(dtsled.class);
}

module_init(led_init);
module_exit(led_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("zuozhongkai");
```

#### pinctl 和 gpio 控制点灯

> **使用须知！！**
>
> 检查 PIN 是否被其他外设使用
>
> **pinctrl 子系统**
>
> 传统的配置 pin 的方式就是直接操作相应的寄存器，但是这种配置 方式比较繁琐、而且容易出问题(比如 pin 功能冲突)。pinctrl 子系统就是为了解决这个问题而引 入的，pinctrl 子系统主要工作内容如下： 
>
> ①、获取设备树中 pin 信息。 
>
> ②、根据获取到的 pin 信息来设置 pin 的复用功能 
>
> ③、根据获取到的 pin 信息来设置 pin 的电气特性，比如上/下拉、速度、驱动能力等。
>
> **gpio 子系统**
>
> gpio 子系统顾名思义，就是用于初始化 GPIO 并且提供相应的 API 函数，比如设置 GPIO 为输入输出，读取 GPIO 的值等

##### API

```c
/*
    打开 imx6ull-alientek-emmc.dts，在 iomuxc 节点中的“imx6ul-evk”子节点下添加“pinctrl_test”节点
    MX6UL_PAD_GPIO1_IO00__GPIO1_IO00  是设置这个 PIN 的复用功能
    config 是设置一个 IO 的上/下拉、驱动能力和速度等
*/
pinctrl_test: testgrp {
    fsl,pins = <
    MX6UL_PAD_GPIO1_IO00__GPIO1_IO00 config   
>;
    
int gpio_request(unsigned gpio, const char *label)  // 申请一个 GPIO 管脚
void gpio_free(unsigned gpio)  // 释放
int gpio_direction_input(unsigned gpio)  // 设置某个 GPIO 为输入
int gpio_direction_output(unsigned gpio, int value)  // 设置某个 GPIO 为输出
int __gpio_get_value(unsigned gpio)  // 获取某个 GPIO 的值(0 或 1)
void __gpio_set_value(unsigned gpio, int value)  // 设置某个 GPIO 的值
    
/* 
	在根节点“/”下创建 test 设备子节点
*/
test {
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_test>;
	gpio = <&gpio1 0 GPIO_ACTIVE_LOW>;
}

// 获取 GPIO 编号
int of_get_named_gpio(struct device_node *np, const char *propname, int index)
// 获取设备树某个属性里面定义了几个 GPIO 信息
int of_gpio_named_count(struct device_node *np, const char *propname)
```

##### 示例

```c
// 创建 GPIO1_IO03 pinctrl 节点
pinctrl_led: ledgrp {
    fsl,pins = <
    	MX6UL_PAD_GPIO1_IO03__GPIO1_IO03 0x10B0 /* LED0 */
    >;
};
// 添加 LED 设备节点
gpioled {
	#address-cells = <1>;
	#size-cells = <1>;
	compatible = "atkalpha-gpioled";
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_led>;
	led-gpio = <&gpio1 3 GPIO_ACTIVE_LOW>;
	status = "okay";
};

// 驱动程序
#include <linux/types.h>
#include <linux/kernel.h>
#include <linux/delay.h>
#include <linux/ide.h>
#include <linux/init.h>
#include <linux/module.h>
#include <linux/errno.h>
#include <linux/gpio.h>
#include <linux/cdev.h>
#include <linux/device.h>
#include <linux/of.h>
#include <linux/of_address.h>
#include <linux/of_gpio.h>
#include <asm/mach/map.h>
#include <asm/uaccess.h>
#include <asm/io.h>
/***************************************************************
Copyright © ALIENTEK Co., Ltd. 1998-2029. All rights reserved.
文件名		: gpioled.c
作者	  	: 左忠凯
版本	   	: V1.0
描述	   	: 采用pinctrl和gpio子系统驱动LED灯。
其他	   	: 无
论坛 	   	: www.openedv.com
日志	   	: 初版V1.0 2019/7/13 左忠凯创建
***************************************************************/
#define GPIOLED_CNT			1		  	/* 设备号个数 */
#define GPIOLED_NAME		"gpioled"	/* 名字 */
#define LEDOFF 				0			/* 关灯 */
#define LEDON 				1			/* 开灯 */

/* gpioled设备结构体 */
struct gpioled_dev{
	dev_t devid;			/* 设备号 	 */
	struct cdev cdev;		/* cdev 	*/
	struct class *class;	/* 类 		*/
	struct device *device;	/* 设备 	 */
	int major;				/* 主设备号	  */
	int minor;				/* 次设备号   */
	struct device_node	*nd; /* 设备节点 */
	int led_gpio;			/* led所使用的GPIO编号		*/
};

struct gpioled_dev gpioled;	/* led设备 */

/*
 * @description		: 打开设备
 * @param - inode 	: 传递给驱动的inode
 * @param - filp 	: 设备文件，file结构体有个叫做private_data的成员变量
 * 					  一般在open的时候将private_data指向设备结构体。
 * @return 			: 0 成功;其他 失败
 */
static int led_open(struct inode *inode, struct file *filp)
{
	filp->private_data = &gpioled; /* 设置私有数据 */
	return 0;
}

/*
 * @description		: 从设备读取数据 
 * @param - filp 	: 要打开的设备文件(文件描述符)
 * @param - buf 	: 返回给用户空间的数据缓冲区
 * @param - cnt 	: 要读取的数据长度
 * @param - offt 	: 相对于文件首地址的偏移
 * @return 			: 读取的字节数，如果为负值，表示读取失败
 */
static ssize_t led_read(struct file *filp, char __user *buf, size_t cnt, loff_t *offt)
{
	return 0;
}

/*
 * @description		: 向设备写数据 
 * @param - filp 	: 设备文件，表示打开的文件描述符
 * @param - buf 	: 要写给设备写入的数据
 * @param - cnt 	: 要写入的数据长度
 * @param - offt 	: 相对于文件首地址的偏移
 * @return 			: 写入的字节数，如果为负值，表示写入失败
 */
static ssize_t led_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *offt)
{
	int retvalue;
	unsigned char databuf[1];
	unsigned char ledstat;
	struct gpioled_dev *dev = filp->private_data;

	retvalue = copy_from_user(databuf, buf, cnt);
	if(retvalue < 0) {
		printk("kernel write failed!\r\n");
		return -EFAULT;
	}

	ledstat = databuf[0];		/* 获取状态值 */

	if(ledstat == LEDON) {	
		gpio_set_value(dev->led_gpio, 0);	/* 打开LED灯 */
	} else if(ledstat == LEDOFF) {
		gpio_set_value(dev->led_gpio, 1);	/* 关闭LED灯 */
	}
	return 0;
}

/*
 * @description		: 关闭/释放设备
 * @param - filp 	: 要关闭的设备文件(文件描述符)
 * @return 			: 0 成功;其他 失败
 */
static int led_release(struct inode *inode, struct file *filp)
{
	return 0;
}

/* 设备操作函数 */
static struct file_operations gpioled_fops = {
	.owner = THIS_MODULE,
	.open = led_open,
	.read = led_read,
	.write = led_write,
	.release = 	led_release,
};

/*
 * @description	: 驱动出口函数
 * @param 		: 无
 * @return 		: 无
 */
static int __init led_init(void)
{
	int ret = 0;

	/* 设置LED所使用的GPIO */
	/* 1、获取设备节点：gpioled */
	gpioled.nd = of_find_node_by_path("/gpioled");
	if(gpioled.nd == NULL) {
		printk("gpioled node not find!\r\n");
		return -EINVAL;
	} else {
		printk("gpioled node find!\r\n");
	}

	/* 2、 获取设备树中的gpio属性，得到LED所使用的LED编号 */
	gpioled.led_gpio = of_get_named_gpio(gpioled.nd, "led-gpio", 0);
	if(gpioled.led_gpio < 0) {
		printk("can't get led-gpio");
		return -EINVAL;
	}
	printk("led-gpio num = %d\r\n", gpioled.led_gpio);

	/* 3、设置GPIO1_IO03为输出，并且输出高电平，默认关闭LED灯 */
	ret = gpio_direction_output(gpioled.led_gpio, 1);
	if(ret < 0) {
		printk("can't set gpio!\r\n");
	}

	/* 注册字符设备驱动 */
	/* 1、创建设备号 */
	if (gpioled.major) {		/*  定义了设备号 */
		gpioled.devid = MKDEV(gpioled.major, 0);
		register_chrdev_region(gpioled.devid, GPIOLED_CNT, GPIOLED_NAME);
	} else {						/* 没有定义设备号 */
		alloc_chrdev_region(&gpioled.devid, 0, GPIOLED_CNT, GPIOLED_NAME);	/* 申请设备号 */
		gpioled.major = MAJOR(gpioled.devid);	/* 获取分配号的主设备号 */
		gpioled.minor = MINOR(gpioled.devid);	/* 获取分配号的次设备号 */
	}
	printk("gpioled major=%d,minor=%d\r\n",gpioled.major, gpioled.minor);	
	
	/* 2、初始化cdev */
	gpioled.cdev.owner = THIS_MODULE;
	cdev_init(&gpioled.cdev, &gpioled_fops);
	
	/* 3、添加一个cdev */
	cdev_add(&gpioled.cdev, gpioled.devid, GPIOLED_CNT);

	/* 4、创建类 */
	gpioled.class = class_create(THIS_MODULE, GPIOLED_NAME);
	if (IS_ERR(gpioled.class)) {
		return PTR_ERR(gpioled.class);
	}

	/* 5、创建设备 */
	gpioled.device = device_create(gpioled.class, NULL, gpioled.devid, NULL, GPIOLED_NAME);
	if (IS_ERR(gpioled.device)) {
		return PTR_ERR(gpioled.device);
	}
	return 0;
}

/*
 * @description	: 驱动出口函数
 * @param 		: 无
 * @return 		: 无
 */
static void __exit led_exit(void)
{
	/* 注销字符设备驱动 */
	cdev_del(&gpioled.cdev);/*  删除cdev */
	unregister_chrdev_region(gpioled.devid, GPIOLED_CNT); /* 注销设备号 */

	device_destroy(gpioled.class, gpioled.devid);
	class_destroy(gpioled.class);
}

module_init(led_init);
module_exit(led_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("zuozhongkai");
```

### platform设备驱动框架

>**驱动的分离与分层**

##### 示例

```c
//  platform 驱动框架
/* 设备结构体 */
struct xxx_dev{
    struct cdev cdev;
    /* 设备结构体其他具体内容 */
};

struct xxx_dev xxxdev; /* 定义个设备结构体变量 */

static int xxx_open(struct inode *inode, struct file *filp)
{ 
     /* 函数具体内容 */
     return 0;
 }

 static ssize_t xxx_write(struct file *filp, const char __user *buf,size_t cnt, loff_t *offt)
 {
     /* 函数具体内容 */
     return 0;
 }
static struct file_operations xxx_fops = {
     .owner = THIS_MODULE,
     .open = xxx_open,
     .write = xxx_write,
 };

 /*
 * platform 驱动的 probe 函数
 * 驱动与设备匹配成功以后此函数就会执行
 */
 static int xxx_probe(struct platform_device *dev)
 { 
     ......
     cdev_init(&xxxdev.cdev, &xxx_fops); /* 注册字符设备驱动 */
     /* 函数具体内容 */
     return 0;
 }

 static int xxx_remove(struct platform_device *dev)
 {
     ......
     cdev_del(&xxxdev.cdev);/* 删除 cdev */
     /* 函数具体内容 */
     return 0;
 }

 /* 匹配列表 */
 static const struct of_device_id xxx_of_match[] = {
     { .compatible = "xxx-gpio" },
     { /* Sentinel */ }
 };

 /* 
 * platform 平台驱动结构体
 */
 static struct platform_driver xxx_driver = {
     .driver = {
         .name = "xxx",
         .of_match_table = xxx_of_match,
     },
     .probe = xxx_probe,
    .remove = xxx_remove,
 };
 
 /* 驱动模块加载 */
 static int __init xxxdriver_init(void)
 {
	return platform_driver_register(&xxx_driver);
 }

 /* 驱动模块卸载 */
 static void __exit xxxdriver_exit(void)
 {
 	platform_driver_unregister(&xxx_driver);
 }
```

### 驱动测试

```c
// led测试
depmod
modporobe gpioled.ko
./ledApp /deb/gpioled 1
rmmod gpioled.ko
    
    
```

### linux应用开发



