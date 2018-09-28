# STM32开发 C语言代码组织规范
> UESTC 信通科协
> 修订日期：2018.9.25

## 代码格式
严格缩进，并保留合适的间隙
逻辑相对较独立的代码块直接换行隔开
不要把所有变量都声明在函数体起始位置，在需要的地方再声明局部变量

``` C
/* bad example */
void Func2(type_t arg1,type_t arg2,type_t*arg3)
{
    size_t i;
    uint8_t *hongkong_reporter;
    if(arg3==NULL)return;
    for(i=0;i<length;i++)
    {
        if(arg1==0&&arg2==0)break;
        if(Func2(arg1,arg2)!=VALUE||arg3->Value==0)
        {
            hongkong_reporter="你们这样子是不行的, I AM ANGRY!";
            SendMessage(hongkong_reporter);
        }
        /* ... */
    }
}
/* good example */
void Func1(type_t arg1, type_t arg2, type_t *arg3)
{
    if (arg3 == NULL) {
        return;
    }

    for (size_t i = 0; i < length; i++)
    {
        if (arg1 == 0 && arg2 == 0) {
            break;
        }

        if (Func2(arg1, arg2) != VALUE || arg3->Value == 0) {
            uint8_t *engineering_drawing = "你们给我搞的这个代码啊，EXCITED!";
            SendMessage(engineering_drawing);
        }
        /* ... */
    }
}
```

## 类型

- **使用stdint整数类型**

``` C
/* 无符号整型 */
uint8_t
uint16_t
uint32_t
/* 表示偏移量 字节数等 */
size_t

/* 有符号整型 */
int8_t
int16_t
int32_t
```

- **使用C99标准布尔逻辑类型**

``` C
_Bool
1   //true
0   //false
```

禁止使用一般数值或指针类型直接作为逻辑变量

``` C
uint8_t *ptr = NULL;

if (ptr) {
    /* can work, but confusing! */
}

if (ptr != NULL) {
    /* this is the preferred usage */
}
```

- **结构体**

结构体的定义应放在 **.h** 头文件中
为与HAL库风格统一，结构体名称都以如下类似的格式命名
``` C
${组件名}_${功能}TypeDef

SPI_HandleTypeDef
UART_HandleTypeDef
GPIO_InitTypeDef
```
结构体中成员以大驼峰法命名
即每个单词都以大写开头，且之间没有空格
``` C
typedef struct {
    /* Data Members */
    type_t MemberName0;
    type_t MemberName1;
    /* ... */
} ${组件名}_${功能}TypeDef;
```
举例
``` C
typedef struct {
    uint8_t Data[UART_RX_BUFFER_SIZE];
    _Bool IsRxComplete;
    uint16_t Count;
} UART_RxBufferTypeDef;
```

- **联合体与位域**

联合体的定义应放在 **.h** 头文件中
联合体中成员使用大驼峰法命名
其中包含是位域结构体成员统一命名为 **BitField**

位域常常用于表示寄存器，因此位成员一般以大写下划线命名
且应和器件datasheet上的寄存器映射表一一对应
位成员可以匿名以表示无效位或者保留位，但必须在后面指明位数

``` C
typedef union {
    /* Data Member */
    type_t MemberName;
    /* Inner Struct */
    struct {
        /* LSB --> MSB */
        _Bool BIT_MEMBER_0      : 1;
        _Bool BIT_MEMBER_1      : 1;
        uint16_t BIT_MEMBER_2   : ${位数};
        _Bool                   : 1;
        /* ... */
    } BitField;
} ${组件名}_${功能}TypeDef;
```
举例
``` C
typedef union {
    uint16_t Data;
    struct {
        _Bool               : 1;
        _Bool MODE          : 1;
        _Bool               : 1;
        _Bool DIV2          : 1;
        _Bool SIGN_PIB      : 1;
        _Bool OPBITEN       : 1;
        _Bool SLEEP12       : 1;
        _Bool SLEEP1        : 1;
        _Bool RESET         : 1;
        _Bool PIN_SW        : 1;
        _Bool PSEL          : 1;
        _Bool FSEL          : 1;
        _Bool HLB           : 1;
        _Bool B28           : 1;
        uint16_t            : 2;
    } BitField;
} AD9834_RegTypeDef;
```

-  **枚举**

枚举的定义应放在 **.h** 头文件中
枚举类型的成员名使用大写下划线命名法
如果进行赋值，注意对其

``` C
typedef enum {
    /* Enum Tags */
    TAG0        = ${VALUE},
    TAG1        = ${VALUE},
    TAG2        = ${VALUE},
    TAG3        = ${VALUE},
    /* ... */
} ${组件名}_${状态标识}TypeDef;
```
举例
``` C
typedef enum {
    AD9959_CHANNEL_0    = 0x10,
    AD9959_CHANNEL_1    = 0x20,
    AD9959_CHANNEL_2    = 0x40,
    AD9959_CHANNEL_3    = 0x80,
} AD9559_ChannelTypeDef;
```

## 变量命名
普通变量名遵循小写下划线准则
即用下划线将单词隔开，且全部小写

- **全局变量**

加入前缀 **g_** 表示 global

``` C
type_t g_variable_name;
```
举例
``` C
// in usart.c
UART_RxBufferTypeDef g_uart1_rx_buffer；
// in main.c
extern UART_RxBufferTypeDef g_uart1_rx_buffer;
```

- **静态局部变量**

加入前缀 **s_** 表示 static

``` C
static type_t s_variable_name;
```
举例
``` C
static int32_t *s_buffer;
static uint16_t s_sample_count;
```

- **函数形参、局部变量与常量**

不加任何前缀，依旧遵守小写下划线原则

``` C
type_t Func(type_t arg_name) {
    type_t local_variable_name;
    /* ... */
}

const constant_variable_name;
```
举例
``` C
void SRAM_WriteBytes(uint32_t offset, uint8_t* data_buffer, uint32_t count)
{
    __IO uint16_t *addr = (__IO uint16_t*)(BASE_ADDR + offset);
    uint16_t *data_buffer_16b = (uint16_t *)data_buffer;
    /* ... */
}

const uint8_t ascii_font_16x8[95][16] = { /* ... */ };
```

- **布尔逻辑类型**

加入前缀 **is_ has_** 表示逻辑条件
可以与全局、静态局部变量的命名规则结合，注意他们的前缀放在最前面

``` C
_Bool is_condition;
_Bool has_condition_value;
```

举例

``` C
_Bool is_number;
_Bool is_screen_vertical;
_Bool g_has_extra_gain;
static _Bool s_is_tx_ready;
```

## 函数

- **函数名**

与HAL库风格一致，采用组件名下划线大驼峰法结合
部分函数可不带组件名前缀

``` C
void ${组件名}_FunctionName(/* args */);
```

注意函数与参数的命名清楚、简洁、尽量让使用者能直接明白其功能
某些常用词可以使用缩写
部分常用缩写:

>**Init**   : Initialize 初始化
>**Str**    : String 字符串
>**Info**   : Information 信息
>**Err**    : Error 错误
>**Msg**    : Message 消息
>**Addr**   : Address 地址
>**Reg**    : Register 寄存器
>**Clk**    : Clock 时钟
>**Freq**   : Frequency 频率
>**Amp**    : Amplitude 幅度
>**Mag**    : Magnitude 大小
>**Lib**    : Library 库
>**Img**    : Image 图像
>**Tx**     : Transmit 发送
>**Rx**     : Receive 接收

举例
``` C
void FATFS_Init(void);
void PE4302_SetAttenuation(uint8_t db_2x);
uint16_t LCD_ReadPixel(uint16_t x, uint16_t y);

void DelayMs(uint16_t ms);
```

- **函数声明**

全局函数、静态局部函数与内联函数、外部函数的声明分别集中在一起
函数声明统一在 **.h** 头文件中, 并保持完整的参数名

举例
``` C
// in ad9959.h
/* Global Functions */
void AD9959_Init(void);
void AD9959_Reset(void);
void AD9959_SetFreq(uint8_t channel, uint32_t freq);
void AD9959_SetPhase(uint8_t channel, float phase);
void AD9959_SetAmp(uint8_t channel, uint16_t amp);
void AD9959_SweepFreq(
	uint8_t channel, uint16_t amp,
	uint32_t start_freq, uint32_t end_freq, uint32_t step_freq,
	uint16_t step_time);
/* Static Local Functions */
static void AD9959_WriteReg(uint8_t reg, uint32_t data);
static inline void AD9959_Update(void);
/* External Functions */
extern void DelayMs(uint16_t ms);
extern void DelayUs(uint32_t us);
extern void ErrorHandler(uint8_t *file, uint32_t line);
```

- **函数定义**

函数定义（实现）统一在 **.c** 源文件中，并与头文件中的声明一一对应

## 注释
如果使用中文注释 
为防止乱码，注意代码文件编码统一设置为 **UTF-8 无BOM**
