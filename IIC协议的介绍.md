# IIC协议的介绍

## IIC的概念：

IIC的简单介绍

I2C（Inter IC Bus）是由Philips公司开发的一种通用数据总线

**两根通信线：SCL（Serial Clock）、SDA（Serial Data），SCL就是时钟线，SDA是数据线**

同步，半双工

带数据应答支持总线挂载多设备（一主多从、多主多从）(这里我们讲解一主多从的模式，因为多主多从的模式并不常用所以这里不过多介绍)

对于通讯协议，我们可以以分层的方式来理解，最基本的是把它分为物理层和协议层。物理层规定通讯系统中具有机械、电子功能部分的特性，确保原始数据在物理媒体的传输。协议层主要规定通讯逻辑，统一收发双方的数据打包、解包标准。简单来说物理层规定我们用嘴巴还是用肢体来交流，协议层则规定我们用中文还是英文来交流。
下面我们分别对 I2C 协议的物理层及协议层进行讲解  



## IIC的物理层介绍：

I2C 通讯设备之间的常用连接方式见图所示：

![image-20231019212046152](C:\Users\20215\AppData\Roaming\Typora\typora-user-images\image-20231019212046152.png)



从图中我们可以看出来IIC的物理层有以下特点：

### 1.一主多从模式：

 它是一个支持设备的总线。“总线”指多个设备共用的信号线。在一个 I2C 通讯总线
中，可连接多个 I2C 通讯设备，支持多个通讯主机及多个通讯从机。

简单来说就是一个老师（mcu）将球（数据）传送好几个学生（从设备）的这样一个过程

（这里我们暂时不涉及到多主多从这种模式）

### 2.总线：

 **一个 I2C 总线只使用两条总线线路，一条双向串行数据线(SDA) ，一条串行时钟线**
**(SCL)。数据线即用来表示数据，时钟线用于数据收发同步。**  

**有的设备可能会将SDA改为DIO，SCL改为CLK**

### 3.设备地址：

 **每个连接到总线的设备都有一个独立的地址**，主机可以利用这个地址进行不同设备之
间的访问。(设备的地址可以从相关厂家的手册查找)

### 4.IIC的电路设计：

 **总线通过上拉电阻接到电源。当 I2C 设备空闲时，会输出高阻态，而当所有设备都空**
**闲，都输出高阻态时，由上拉电阻把总线拉成高电平。**

这里我们来深度解析一下：

#### 1.为什么不用推挽输出：

首先我们知道在单片机中一般采用推挽输出和开漏输出两种方式，**那么为什么IIC协议的电路设计不采用推挽输出呢**？



这里我们来分析一下推挽输出的电路：
![Screenshot_20231214_195135](D:\QMDownload\MobileFile\Screenshot_20231214_195135.jpg)



当设备A给高电平的时候上端的p-mos导通，下方的n-mos截止

如果此时设备B给低电平的时候上端的p-mos截止，下方的n-mos导通

这种现象也叫做**串通现象**（相关的电路知识，这里就不详细列出来了，大家自行补充）

![Screenshot_20231214_200938](D:\QMDownload\MobileFile\Screenshot_20231214_200938.jpg)

**问题显而易见：总线上会出现给VCC提供了一条低阻抗路径，电源短路，设备烧毁。**

#### 2.采用开漏输出的优势：

这时候我们来看看开漏输出电路：

![Screenshot_20231214_201334](D:\QMDownload\MobileFile\Screenshot_20231214_201334.jpg)

控制端无论是给高电平还是给低电平，显然都没有办法输出高电平

那如果我们外接一个上拉电阻呢？

![Screenshot_20231214_201925](D:\QMDownload\MobileFile\Screenshot_20231214_201925.jpg)

显然这里就给出来开漏输出需要接上拉电阻的答案：

**接上拉电阻是因为I2C通信需要输出高电平的能力。一般开漏输出无法输出高电平，如果在漏极接上拉电阻，则可以进行电平转换，**

**电压的大小则取决与电阻的大小**



再者开漏输出还可以实现**“线与”**的功能：

这里解释一下什么是“线与”：

"线与"是一种逻辑门电路中常用的逻辑门类型，也被称为与门（AND gate）。它是由**多个输入端**和**一个输出端**组成的逻辑门。

**线与的功能是在所有输入端同时为逻辑高电平（1）时，输出端才会产生逻辑高电平。换句话说，只有当所有输入信号同时为真时，输出信号才会为真。如果任意一个或多个输入信号为逻辑低电平（0），无论其他输入信号的状态如何，输出信号都将为逻辑低电平。**

从下图我们就可以看出来开漏输出如何实现**“线与”**的功能

![Screenshot_20231214_202724](D:\QMDownload\MobileFile\Screenshot_20231214_202724.jpg)

**显然我们就可以知道为什么当总线空闲时，这SDA和SCL两条线路都是高电平状态。**

#### 3.怎么样通过电路设计来避免干扰：

首先我们知道IIC的所有设备是接在一根总线上的，那么我们进行通信的时候往往只是几个设备进行通信，那么这时候其余的空闲设备可能会受到总线干扰，或者干扰到总线，怎么办呢？

在 I2C 总线上，当只有部分设备需要进行通信时，其他空闲设备可能会受到总线干扰或者干扰到总线。为了解决这个问题，I2C 总线采用了一种多主设备的通信模式和开漏输出结构。

**多主设备通信模式允许多个设备共享同一条总线，并且通过地址来区分不同的设备。每个设备在总线上都有一个唯一的地址，主设备可以选择性地与特定的从设备进行通信。只有被选中的设备会响应主设备的请求，其他设备则处于监听状态。**

**在开漏输出结构中，设备的输出端口可以通过拉低数据线（SDA）来表示逻辑 0，而释放数据线则表示逻辑 1。因此，即使有多个设备同时输出低电平（逻辑 0），总线上的电平也不会受到冲突。这是因为总线上拉电阻（pull-up resistor）的存在，使得总线在没有设备拉低时保持高电平。**

**当一个设备想要发送数据时，它会首先检测总线上的电平状况，确保总线空闲。然后，它会发送起始条件，开始通信。其他设备在检测到起始条件后会进入监听模式，不会主动发送数据。**

**如果有其他设备正在通信，而某个设备希望发送数据，它会等待总线空闲，并在合适的时机发送起始条件，以获取总线控制权。这种机制确保了多个设备之间的协调和共享总线的能力。**

总结起来，为了解决同时存在的多个设备之间的干扰问题，I2C 总线采用多主设备通信模式和开漏输出结构。这些机制允许设备在需要时获取总线控制权，并通过特定的协议和时序规定实现有效的通信。

### 5.仲裁：

 多个主机同时使用总线时，为了防止数据冲突，会利用仲裁方式决定由哪个设备占用
总线。（这里不会深入研究因为涉及到多主多从的模式）

### 6.传输模式：

具有三种传输模式：标准模式传输速率为 100kbit/s ，快速模式为 400kbit/s ，高速模式
下可达 3.4Mbit/s，但目前大多 I2C 设备尚不支持高速模式。

### 7.挂载设备数量限制：

连接到相同总线的 IC 数量受到总线的最大电容 400pF 限制   



## IIC的协议层介绍：

对应IIC的在协议层方面我们一般通过时序图来编写代码，在后面面对陌生的IIC模块，我们一般也是从时序图去入手编写代码，本质上不同IIC模块的协议基本不会差太多，都是大同小异。

下图是IIC的时序图:

![image-20231214220734579](C:\Users\20215\AppData\Roaming\Typora\typora-user-images\image-20231214220734579.png)

起始-地址（最后一位是读写位）-响应-数据（8位等于一字节）-响应-停止

完整传输流程如下：

**① SDA和SCL开始都为高， 然后主机将SDA拉低， 表示开始信号。**

**② 在接下来的8个时间周期里，主机控制SDA的高低， 发送从机地址。 其中第8位如果为0， 表示接下来是写操作，即主机传输数据给从机； 如果为1，表示接下来是读操作，即从机传输数据给主机； 另外， 数据传输是从最高位到最低位，因此传输方式为MSB（ Most Significant Bit）。**

**③ 总线中对应从机地址的设备，发出应答信号。**

**④ 在接下来的8个时间周期里，如果是写操作，则主机控制SDA的高低；如果是读操作，则从机控制SDA的高低。**

**⑤ 每次传输完成， 接收数据的设备， 都发出应答信号。**

**⑥ 最后， 在SCL为高时， 主机由低拉高SDA， 表示停止信号，整个传输结束。**

**数据有效性：传输数据的时候SCL为高电平的时候SDA的电平不能突变，只有当SCL为低电平的时候，SDA才能进行电平的切换。**

关于延迟时间的问题：请查阅iic的相关手册，里面涉及到很多复杂的东西，本人能力有限暂时无法回答为什么要延迟这么久的问题。

下面我们来对这个时序图逐一分解：

### 1.起始信号：

![image-20231214220936277](C:\Users\20215\AppData\Roaming\Typora\typora-user-images\image-20231214220936277.png)

```c
//产生IIC起始信号
void IIC_Start(void)
{
	SDA_OUT();     //sda线输出  
	IIC_SDA=1;	  	  
	IIC_SCL=1;
	delay_us(4);
 	IIC_SDA=0;//START:when CLK is high,DATA change form high to low 
	delay_us(4);
	IIC_SCL=0;//钳住I2C总线，准备发送或接收数据 
}	  
```

### 2.停止信号：

![image-20231214220936277](C:\Users\20215\AppData\Roaming\Typora\typora-user-images\image-20231214220936277.png)

```c
//产生IIC停止信号
void IIC_Stop(void)
{
	SDA_OUT();//sda线输出
	IIC_SCL=0;
	IIC_SDA=0;//STOP:when CLK is high DATA change form low to high
 	delay_us(4);
	IIC_SCL=1; 
	IIC_SDA=1;//发送I2C总线结束信号
	delay_us(4);							   	
}
```

### 3.应答信号：

![image-20231214221443391](C:\Users\20215\AppData\Roaming\Typora\typora-user-images\image-20231214221443391.png)

```c

/*写应答*/
//等待应答信号到来
//返回值：1，接收应答失败
//        0，接收应答成功
u8 IIC_Wait_Ack(void)
{S²
	u8 ucErrTime=0;
	SDA_IN();      //SDA设置为输入  
	IIC_SDA=1;delay_us(2);	   
	IIC_SCL=1;delay_us(2);	 
	while(READ_SDA) // #define READ_SDA   PBin(7)  //输入SDA 
	{
		ucErrTime++;
		if(ucErrTime>250)
		{
			IIC_Stop();
			return 1;
		}
	}
	IIC_SCL=0;//时钟输出0 	   
	return 0;  
} 

/*读应答*/

//产生ACK应答
void IIC_Ack(void)
{
	IIC_SCL=0;
	SDA_OUT();
	IIC_SDA=0;
	delay_us(2);
	IIC_SCL=1;
	delay_us(2);
	IIC_SCL=0;
}
//不产生ACK应答		    
void IIC_NAck(void)
{
	IIC_SCL=0;
	SDA_OUT();
	IIC_SDA=1;
	delay_us(2);
	IIC_SCL=1;
	delay_us(2);
	IIC_SCL=0;
}					 		
```

### 4.写数据：

```c
void IIC_Send_Byte(u8 txd)
{                        
    u8 t;   
	SDA_OUT(); 	    
    IIC_SCL=0;//拉低时钟开始数据传输
    for(t=0;t<8;t++)
    {              
        IIC_SDA=(txd&0x80)>>7; //0x80对应二进制10000000  
        txd<<=1; 	  
		delay_us(2);   //对TEA5767这三个延时都是必须的
		IIC_SCL=1;
		delay_us(2); 
		IIC_SCL=0;	
		delay_us(2);
    }	 
} 	    
```

### 5.读数据：

```c
/读1个字节，ack=1时，发送ACK，ack=0，发送nACK   
u8 IIC_Read_Byte(unsigned char ack)
{
	unsigned char i,receive=0;
	SDA_IN();//SDA设置为输入
    for(i=0;i<8;i++ )
	{
        IIC_SCL=0; 
        delay_us(2);
		IIC_SCL=1;
        receive<<=1;
        if(READ_SDA)receive++;   
		delay_us(1); 
    }					 
    if (!ack)
        IIC_NAck();//发送nACK
    else
        IIC_Ack(); //发送ACK   
    return receive;
}
```

### 6.完整时序图代码：

```c


//初始化IIC
void IIC_Init(void)
{					     
	GPIO_InitTypeDef GPIO_InitStructure;
	RCC_APB2PeriphClockCmd(	RCC_APB2Periph_GPIOB, ENABLE );	//使能GPIOB时钟
	   
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6|GPIO_Pin_7;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP ;   //推挽输出
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
	GPIO_SetBits(GPIOB,GPIO_Pin_6|GPIO_Pin_7); 	//PB6,PB7 输出高
}
//产生IIC起始信号
void IIC_Start(void)
{
	SDA_OUT();     //sda线输出  
	IIC_SDA=1;	  	  
	IIC_SCL=1;
	delay_us(4);
 	IIC_SDA=0;//START:when CLK is high,DATA change form high to low 
	delay_us(4);
	IIC_SCL=0;//钳住I2C总线，准备发送或接收数据 
}	  
//产生IIC停止信号
void IIC_Stop(void)
{
	SDA_OUT();//sda线输出
	IIC_SCL=0;
	IIC_SDA=0;//STOP:when CLK is high DATA change form low to high
 	delay_us(4);
	IIC_SCL=1; 
	IIC_SDA=1;//发送I2C总线结束信号
	delay_us(4);							   	
}
//等待应答信号到来
//返回值：1，接收应答失败
//        0，接收应答成功
u8 IIC_Wait_Ack(void)
{
	u8 ucErrTime=0;
	SDA_IN();      //SDA设置为输入  
	IIC_SDA=1;delay_us(2);	   
	IIC_SCL=1;delay_us(2);	 
	while(READ_SDA)
	{
		ucErrTime++;
		if(ucErrTime>250)
		{
			IIC_Stop();
			return 1;
		}
	}
	IIC_SCL=0;//时钟输出0 	   
	return 0;  
} 
//产生ACK应答
void IIC_Ack(void)
{
	IIC_SCL=0;
	SDA_OUT();
	IIC_SDA=0;
	delay_us(2);
	IIC_SCL=1;
	delay_us(2);
	IIC_SCL=0;
}
//不产生ACK应答		    
void IIC_NAck(void)
{
	IIC_SCL=0;
	SDA_OUT();
	IIC_SDA=1;
	delay_us(2);
	IIC_SCL=1;
	delay_us(2);
	IIC_SCL=0;
}					 				     
//IIC发送一个字节
//返回从机有无应答
//1，有应答
//0，无应答			  
void IIC_Send_Byte(u8 txd)
{                        
    u8 t;   
	SDA_OUT(); 	    
    IIC_SCL=0;//拉低时钟开始数据传输
    for(t=0;t<8;t++)
    {              
        IIC_SDA=(txd&0x80)>>7; //10000000  00000111
        txd<<=1; 	  
		delay_us(2);   //对TEA5767这三个延时都是必须的
		IIC_SCL=1;
		delay_us(2); 
		IIC_SCL=0;	
		delay_us(2);
    }	 
} 	    
//读1个字节，ack=1时，发送ACK，ack=0，发送nACK   
u8 IIC_Read_Byte(unsigned char ack)
{
	unsigned char i,receive=0;
	SDA_IN();//SDA设置为输入
    for(i=0;i<8;i++ )
	{
        IIC_SCL=0; 
        delay_us(2);
		IIC_SCL=1;
        receive<<=1;
        if(READ_SDA)receive++;   
		delay_us(1); 
    }					 
    if (!ack)
        IIC_NAck();//发送nACK
    else
        IIC_Ack(); //发送ACK   
    return receive;
}




























```

## 案例：

### 1.TM1650



#### TM1650基本概念：

TM1650 是一种带键盘扫描接口的 LED（发光二极管显示器）驱动控制专用电路。内部集成有 MCU

输入输出控制数字接口、数据锁存器、LED 驱动、键盘扫描、辉度调节等电路。TM1650 性能稳定、质

量可靠、抗干扰能力强，可适用于 24 小时长期连续工作的应用场合。



**内部结构框图**：

![image-20231221092220374](C:\Users\20215\AppData\Roaming\Typora\typora-user-images\image-20231221092220374.png)



#### TM1650的时序图：

（这里只介绍数码管）

**时序图：**

![image-20231221092949494](C:\Users\20215\AppData\Roaming\Typora\typora-user-images\image-20231221092949494.png)

command1：发送数据命令，0x48

command2：开显示，亮度大小多大

address：数码管的位选控制，即点亮哪一位数码管

data：数码管段选，即点亮数码管的那一段



**控制命令说明：**

![image-20231221093630388](C:\Users\20215\AppData\Roaming\Typora\typora-user-images\image-20231221093630388.png)

这里使用模式命令即可

**亮度调节命令说明**：

![image-20231221093719111](C:\Users\20215\AppData\Roaming\Typora\typora-user-images\image-20231221093719111.png)



**数码管位选地址：**

![image-20231221093932503](C:\Users\20215\AppData\Roaming\Typora\typora-user-images\image-20231221093932503.png)

​		即：0x68，0x6a，0x6c，0x6e



#### 完整代码：

```c
#include "myiic.h"
#include "delay.h"


/*数码管初始化*/
void Show_Init(uint8_t Data_Command,uint8_t Display_Command)
{
	IIC_Start();
	IIC_Send_Byte(Data_Command);//命令控制
	IIC_Wait_Ack();
	IIC_Send_Byte(Display_Command);//亮度控制
	IIC_Wait_Ack();
	IIC_Stop();

}
/*数码管位选、段选*/
void Data_Display(uint8_t Location_Section,uint8_t Number_Section)
{
	IIC_Start();
	IIC_Send_Byte(Location_Section);//位选
	IIC_Wait_Ack();
	IIC_Send_Byte(Number_Section);//段选
	IIC_Wait_Ack();
	IIC_Stop();
	
}
/*关闭数码管*/
void Display_Off()
{
	IIC_Start();
	IIC_Send_Byte(0x48);
	IIC_Wait_Ack();
	IIC_Send_Byte(0x00);
	IIC_Wait_Ack();
	IIC_Stop();

}
```

### 2.EEPROM

### 3.MPU6050
