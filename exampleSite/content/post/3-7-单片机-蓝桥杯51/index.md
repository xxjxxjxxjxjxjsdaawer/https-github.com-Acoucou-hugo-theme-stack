+++
author = "coucou"
title = "单片机——蓝桥杯51"
date = "2023-08-01"
description = "单片机专题之蓝桥杯51"
categories = [
    "单片机"
]
tags = [
    "单片机","蓝桥杯51"
]
+++

![](1.jpg)

## 速记

### 初始化

```c
#include <STC15F2K60S2.H>
#include "intrins.h"

#define u8 unsigned char
#define u32 unsigned int

sbit S7 = P3^0;
sbit S6 = P3^1;
sbit S5 = P3^2;
sbit S4 = P3^3;

unsigned char show_dat[] = {
	0xc0, 0xf9, 0xa4, 0xb0, 0x99, 0x92, 0x82, 0xf8, 0x80, 0x90, 0xff, 0xc1, 0x1e
};

void set_port(u8 port, u8 dat){
	P2 = (P2 & 0x1f) | port;
	P0 = dat;
	P2 = P2 & 0x1f;
}

void allinit(){
	set_port(0x80, 0xff);
	set_port(0xa0, 0x00);
	set_port(0xc0, 0xff);
	set_port(0xe0, 0xff);
}

void key_proc(){
    
}
void led_proc(){
    
}
void disp_proc(){
    
}
```

### 显示

```c
#include <STC15F2K60S2.H>
#include "show.h"

u8 data_adc[] = {
	0xc0, 0xf9, 0xa4, 0xb0, 0x99, 0x92, 0x82, 0xf8, 0x80, 0x90, 0xff
};

void Delay1ms()		//@12.000MHz
{
	unsigned char i, j;

	i = 12;
	j = 169;
	do
	{
		while (--j);
	} while (--i);
}

void show(u8 yi, u8 er, u8 san, u8 si, u8 wu, u8 liu, u8 qi, u8 ba){
	P2 = 0xc0;
	P0 = 0x01;
	P2 = 0xe0;
	P0 = yi;
	Delay1ms();
	
	P2 = 0xc0;
	P0 = 0x02;
	P2 = 0xe0;
	P0 = er;
	Delay1ms();
	
	P2 = 0xc0;
	P0 = 0x04;
	P2 = 0xe0;
	P0 = san;
	Delay1ms();
	
	P2 = 0xc0;
	P0 = 0x08;
	P2 = 0xe0;
	P0 = si;
	Delay1ms();
	
	P2 = 0xc0;
	P0 = 0x10;
	P2 = 0xe0;
	P0 = wu;
	Delay1ms();
	
	P2 = 0xc0;
	P0 = 0x20;
	P2 = 0xe0;
	P0 = liu;
	Delay1ms();
	
	P2 = 0xc0;
	P0 = 0x40;
	P2 = 0xe0;
	P0 = qi;
	Delay1ms();
	
	P2 = 0xc0;
	P0 = 0x80;
	P2 = 0xe0;
	P0 = ba;
	Delay1ms();
	
	P2 = 0x00;
}
```

### AT24c02

```c
/*
    writecommand 0xa0
    readcommand  0xa1
*/
// 读一个字节
unsigned char read24c02(unsigned char addr){
	unsigned char dat;
	
	IIC_Start();
	IIC_SendByte(0xa0);
	IIC_WaitAck();
	
	IIC_SendByte(addr);
	IIC_WaitAck();
	
	IIC_Start();
	IIC_SendByte(0xa1);
	IIC_WaitAck();
	dat = IIC_RecByte();
	IIC_SendAck(1);
	IIC_Stop();
	
	return dat;
}
// 写一个字节
void write24c02(unsigned char addr, unsigned char dat){
	IIC_Start();
	IIC_SendByte(0xa0);
	IIC_WaitAck();
	
	IIC_SendByte(addr);
	IIC_WaitAck();
	
	IIC_SendByte(dat);
	IIC_WaitAck();
	IIC_Stop();
}
```

### AD/DA

```c
/*
    writecommand 0x90
    readcommand  0x91
    
    set_pcf(0x03);
    dac_dat = read_pcf();
	dac_dat = dac_dat * (5.0 / 255);
*/
// 选择通道, 通道1是光敏，通道3是电阻
void seletPcf(unsigned char channel){
	IIC_Start();
	IIC_SendByte(0x90);
	IIC_WaitAck();
	
	IIC_SendByte(channel);
	IIC_WaitAck();
	
	IIC_Stop();
}

// ADC输入
unsigned char readChannel(){
	unsigned int dat;
	
	IIC_Start();
	
	IIC_SendByte(0x91);
	IIC_WaitAck();
	
	dat = IIC_RecByte();
	IIC_SendAck(1);
	IIC_Stop();
	
	return dat;
}
// DAC输出
void out_channel(unsigned char dat){
	IIC_Start();

	IIC_SendByte(0x90);
	IIC_WaitAck();
	
	IIC_SendByte(0x40);
	IIC_WaitAck();

	IIC_SendByte(dat);
	IIC_WaitAck();
	
	IIC_Stop();
}
```

### DS1302-时间

```c
/*
显示时间的时候要特别注意，BCD码转10！！！
应当为16进制
*/
void set_time(unsigned char hour, unsigned char min, unsigned char sec)
{
	Write_Ds1302_Byte(0x8e, 0x00);
	Write_Ds1302_Byte(0x80, sec);
	Write_Ds1302_Byte(0x82, min);
	Write_Ds1302_Byte(0x84, hour);
	Write_Ds1302_Byte(0x8e, 0x80);
}
void read_time()
{
	time[2] = Read_Ds1302_Byte(0x85);
	time[1] = Read_Ds1302_Byte(0x83);
	time[0] = Read_Ds1302_Byte(0x81);
}
```

### onewire-温度

```c
/*
特别注意！！！驱动里的延时函数！！！
void Delay_OneWire(unsigned int t)  //STC89C52RC
{
	t = t * 10;
	while(t--);
}
*/

float read_tmp(){
	float tmp;
	unsigned char h_dat, l_dat;
	unsigned int temp;
	
	init_ds18b20();
	Write_DS18B20(0xcc);
	Write_DS18B20(0x44); // 开始转化
	
	init_ds18b20();
	Write_DS18B20(0xcc);
	Write_DS18B20(0xbe);
	
	l_dat = Read_DS18B20();
	h_dat = Read_DS18B20();
	
	temp = (h_dat << 8) | l_dat;
	tmp = temp / 16.0;
	
	return tmp;
}
```

### 超声波测距

```c
// 超声波
sbit TX = P1^0;
sbit RX = P1^1;
u32 distant; 

void Delay12us()		//@12.000MHz
{
	unsigned char i;

	_nop_();
	_nop_();
	i = 33;
	while (--i);
}
void send_wave(){
	u8 i = 8;
	while(i--){
		TX = 1;
		Delay12us();
		TX = 0;
		Delay12us();
	}
}
void Timer1Init(void)		//1微秒@12.000MHz
{
	TMOD &= 0x0F;		//设置定时器模式
	TL1 = 0x00;		//设置定时初值
	TH1 = 0x00;		//设置定时初值
	TF1 = 0;		//清除TF1标志
	//TR1 = 1;		//定时器1开始计时
}
void test_dis(){
	send_wave();
	TR1 = 1;		//定时器1开始计时
	while(RX == 1 && TF1 == 0);
	TR1 = 0;
	if(TF1 == 0){
		//distant = ((TH1 << 8) | TL1) * 340 / 2 / 10000;
		distant = ((TH1 << 8) | TL1) * 0.017;
		
		liu = data_pros[distant / 100 % 10];
		qi = data_pros[distant / 10 % 10];
		ba = data_pros[distant % 10];
		show(0xff, 0xff,0xff, 0xff, 0xff, liu, qi, ba);
	}else{
		show(0x00, 0xff,0xff, 0xff, 0xff, 0xff, 0xff, 0xff);
		TF1 = 0;
	}
	TL1 = 0;
	TH1 = 0;
}
```

### NE555

```c
/*
定时器定时1s，计数多少次，即频率
特别注意：
	TL0 = 0xff;				//设置定时初始值
	TH0 = 0xff;				//设置定时初始值
*/
void Timer0_Init(void)		//1毫秒@12.000MHz
{
	AUXR |= 0x80;			//定时器时钟1T模式
	TMOD = 0x04;			//设置定时器模式
	TL0 = 0xff;				//设置定时初始值
	TH0 = 0xff;				//设置定时初始值
	TF0 = 0;				//清除TF0标志
	TR0 = 1;				//定时器0开始计时
	
	ET0 = 1;
	EA = 1;
}
void time0_isr() interrupt 1{
	timer0_cnt++;
}
void Timer1_Init(void)		//1毫秒@12.000MHz
{
	AUXR |= 0x40;			//定时器时钟1T模式
	TMOD = 0x04;			//设置定时器模式
	TL1 = 0x20;				//设置定时初始值
	TH1 = 0xD1;				//设置定时初始值
	TF1 = 0;				//清除TF1标志
	TR1 = 1;				//定时器1开始计时
	
	ET1 = 1;
	EA = 1;
}
void time1_isr() interrupt 3{
	timer1_cnt++;
	
	if(timer1_cnt == 1000){
		f_dat = timer0_cnt;
		timer1_cnt = 0;
		timer0_cnt = 0;
	}
}
```

### timer

```c
void time0_isr() interrupt 1{
	
}
void time1_isr() interrupt 3{
	
}
```

### key

```c
sbit R0 = P3^2;
sbit R1 = P3^3;
sbit _R0 = P3^0;
sbit _R1 = P3^1;

sbit L0 = P3^4;
sbit L1 = P3^5;
sbit _L0 = P4^2;
sbit _L1 = P4^4;

void key_proc(){
	R0 = R1 = _R0 = _R1 = 1;
	L0 = L1 = _L0 = _L1 = 1;
	L0 = 0;
	if(R0 == 0){  // S17， 数据加
		Delay10ms();while(R0 == 0);Delay10ms();
		if(set_adc_flag)	adc_para += 0.5;
		if(adc_para > 5)	adc_para = 0;
		error_flag = 0;
	}
	if(R1 == 0){  // S16  数据减
		Delay10ms();while(R1 == 0);Delay10ms();
		if(set_adc_flag)	adc_para -= 0.5;
		if(adc_para < 0)	adc_para = 5;
		error_flag = 0;
	}
	if(_R0 == 0){Delay10ms();while(_R0 == 0);Delay10ms();cnt++;}
	if(_R1 == 0){Delay10ms();while(_R1 == 0);Delay10ms();cnt++;}
	
	R0 = R1 = _R0 = _R1 = 1;
	L0 = L1 = _L0 = _L1 = 1;
	L1 = 0;
	if(R0 == 0){  // S13，清零
		Delay10ms();while(R0 == 0);Delay10ms();
		if(cnt_flag)	adc_cnt = 0;
		error_flag = 0;
	}
	if(R1 == 0){  // S12，界面
		Delay10ms();while(R1 == 0);Delay10ms();
	}
	if(_R0 == 0){Delay10ms();while(_R0 == 0);Delay10ms();cnt++;}
	if(_R1 == 0){Delay10ms();while(_R1 == 0);Delay10ms();cnt++;}
	
	R0 = R1 = _R0 = _R1 = 1;
	L0 = L1 = _L0 = _L1 = 1;
	_L0 = 0;
	if(R0 == 0){Delay10ms();while(R0 == 0);Delay10ms();cnt++;}
	if(R1 == 0){Delay10ms();while(R1 == 0);Delay10ms();cnt++;}
	if(_R0 == 0){Delay10ms();while(_R0 == 0);Delay10ms();cnt++;}
	if(_R1 == 0){Delay10ms();while(_R1 == 0);Delay10ms();cnt++;}
	
	R0 = R1 = _R0 = _R1 = 1;
	L0 = L1 = _L0 = _L1 = 1;
	_L1 = 0;
	if(R0 == 0){Delay10ms();while(R0 == 0);Delay10ms();cnt++;}
	if(R1 == 0){Delay10ms();while(R1 == 0);Delay10ms();cnt++;}
	if(_R0 == 0){Delay10ms();while(_R0 == 0);Delay10ms();cnt++;}
	if(_R1 == 0){Delay10ms();while(_R1 == 0);Delay10ms();cnt++;}
}
```



