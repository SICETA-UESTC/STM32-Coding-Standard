# STM32开发 C语言代码组织规范
> UESTC 信通科协<br>
> 修订日期：2018.12.12<br>

## 代码格式
- **大括号与缩进**<br>
缩进使用4个空格字符（可设置IDE将Tab自动转换为4个空格）<br>
任何代码块都要加大括号，包括单个语句<br>
左大括号提行打头或者本行空格打头可以混合使用，清晰即可<br>
横向过长的代码提行缩进对齐<br>

- **保留合适间距**<br>
运算符、函数参数列表、for循环体等应保留适当空格以方便阅读<br>
善用IDE的代码规范化功能自动整理格式<br>
代码段根据其逻辑划分用空行隔开，不要全部挤在一起！<br>

- **注意局部变量作用域**<br>
现在不是C89了，不要把局部变量全部定义在函数开头！<br>
哪里使用哪里定义，限制作用域，保证逻辑局部性<br>

举例<br>
``` C
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
    }
}

float GetBaseFreq(float *fft_output, float sample_freq_hz, size_t length)
{
    arm_cmplx_mag_f32(fft_output, fft_output, length / 2);
    
    float max_val;
    size_t base_peak_index;
    arm_max_f32(fft_output + i, 32, &max_val, &base_peak_index);

    float base_freq = base_peak_index * sample_freq_hz / length;

    /* Too loooooong code! */
    base_freq += (fft_output[base_peak_index + 1] - fft_output[base_peak_index - 1]) * coef1 * amp_factor
               + (fft_output[base_peak_index + 2] - fft_output[base_peak_index - 2]) * coef2 * amp_factor;
    
    return base_freq;
}
```

## 类型

- **整数类型**

使用C99标准整数类型<br>
``` C
/* 无符号整型 */
uint8_t//一般用来表示字节数据或字符
uint16_t
uint32_t
uint64_t

/* 有符号整型 */
int8_t
int16_t
int32_t
int64_t

/* 无符号数, 表示正偏移量 字节数 循环控制变量等 */
size_t
/* 有符号数, 表示两个指针之差 */
ptrdiff_t
```

- **布尔逻辑类型**

使用C99标准的_Bool类型<br>
``` C
_Bool is_logical;
is_logical = 1   //true
is_logical = 0   //false
```
也可以使用类似c++风格的布尔类型别名<br>
``` C
/* 为了使用bool类型别名，需要引用此头文件 */
#include <stdbool.h>
/* ... */
bool is_logical;
is_logical = true;
is_logical = false;
```

禁止使用一般数值或指针类型直接作为逻辑变量<br>

``` C
uint8_t *ptr = NULL;
int32_t val = -1;

// Bad habit
if (ptr) {
    /* can work, but confusing! */
}

while (val) {
    /* val == -1 doesn't mean false! */
}

// Good practice
if (ptr != NULL) {
    /* now I know the pointer is valid */
}

while (val != -1) {
    /* much more clear */
}
```

- **结构体 (Structure)**

结构体的定义应放在 **.h** 头文件中<br>
为与HAL库风格统一，结构体的类型名以如下格式命名<br>
``` C
${组件名}_${功能}TypeDef

SPI_HandleTypeDef
UART_HandleTypeDef
GPIO_InitTypeDef
```
结构体中的成员以大驼峰法命名<br>
即每个单词都以大写开头，且之间没有空格或下划线<br>
``` C
typedef struct {
    /* Data Members */
    type_t MemberName0;
    type_t MemberName1;
    /* ... */
} ${组件名}_${功能}TypeDef;
```
举例<br>
``` C
typedef struct {
    uint8_t Data[UART_RX_BUFFER_SIZE];
    _Bool IsRxComplete;
    uint16_t Count;
} UART_RxBufferTypeDef;
```

- **联合体 (Union)**

联合体用于表示不同类型的变量共享同一段内存，可以很方便地进行二进制数据（如字节流）打包与解析，也常常配合位域使用。<br>

联合体的定义应放在 **.h** 头文件中<br>
联合体中成员使用大驼峰法命名<br>
若包含普通结构体成员命名为 **DataField**<br>
若包含位域结构体成员命名为 **BitField**<br>
若包含一段连续字节数组命名为 **Bytes**<br>

``` C
typedef union {
    /* Data Member */
    type_t MemberName;
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
举例<br>
``` C
typedef union {
    struct {
        uint8_t Header;
        uint8_t Command;
        uint8_t CheckSum;
    } DataField;
    uint8_t Bytes[3];
} GY955_Tx_TypeDef;
```

- **位域 (BitField)**

位域是在结构体基础上的数据结构，可以显式成员在内存中占用的bit数<br>
相比于位运算，可以对数据进行更直观的位操作，常常用于表示寄存器内容<br>

位域与联合体配合使用时，可以不声明其类型名，而直接作为联合体的成员变量，并命名为 **BitField**<br>

位成员的类型根据其有无符号以及位数多少使用标准整型<br>
对于只占1个bit的位成员使用 **_Bool** 类型<br>

STM32默认为小端模式，位成员声明应从低位到高位<br>
位成员的命名遵循大写下划线方式，且应和器件datasheet上的寄存器映射表一一对应，可以用匿名位成员表示无效位或者保留位<br>

所有位成员（无论匿名与否）必须注明位数，注意缩进对齐<br>

``` C
typedef union {
    /* Data Member */
    type_t MemberName;
    struct {
        /* LSB --> MSB */
        _Bool BIT_MEMBER_0      : 1;
        _Bool BIT_MEMBER_1      : 1;
        uint8_t BIT_MEMBER_2    : ${位数};
        int16_t BIT_MEMBER_3    : ${位数};
        _Bool                   : 1;
        uint8_t                 : 4;
        /* ... */
    } BitField;
} ${组件名}_${功能}TypeDef;
```

举例<br>
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
        uint8_t             : 2;
    } BitField;
} AD9834_RegTypeDef;
```

-  **枚举 (Enumeration)**

枚举的定义应放在 **.h** 头文件中<br>
枚举类型名按照以下结构命名<br>
``` C
${组件名}_${功能标识};
```
枚举类型的枚举成员使用大写下划线命名法<br>
如果进行赋值，注意缩进对其<br>

``` C
typedef enum {
    /* Enum Tags */
    TAG0        = ${VALUE},
    TAG1        = ${VALUE},
    TAG2        = ${VALUE},
    TAG3        = ${VALUE},
    /* ... */
} ${组件名}_${功能标识};
```
举例<br>
``` C
typedef enum {
    AD9959_CHANNEL_0    = 0x10,
    AD9959_CHANNEL_1    = 0x20,
    AD9959_CHANNEL_2    = 0x40,
    AD9959_CHANNEL_3    = 0x80,
} AD9559_Channels;

typedef enum {
    GY955_QUERY_ACC     = 0x15,
    GY955_QUERY_GYRO    = 0x25,
    GY955_QUERY_MAG     = 0x35,
    GY955_QUERY_EULER   = 0x45,
    GY955_QUERY_Q4      = 0x55,
} GY955_QueryCmd;
```

## 变量
普通变量名遵循小写下划线准则<br>
即用下划线将单词隔开，且全部小写<br>

- **全局变量**

全局变量可在工程的所有文件中访问，应慎用<br>
命名时加入前缀 **g_** 表示 global<br>

``` C
type_t g_variable_name;
```
举例<br>
``` C
// in usart.c
UART_RxBufferTypeDef g_uart1_rx_buffer；
// in main.c
extern UART_RxBufferTypeDef g_uart1_rx_buffer;
```

- **静态全局变量**

静态局部变量可在当前文件的所有函数中访问，但对其他文件不可见<br>
命名时加入前缀 **s_** 表示 static<br>

``` C
static type_t s_variable_name;
```
举例<br>
``` C
static int32_t *s_buffer;
static uint16_t s_sample_count;
```

- **函数形参与局部变量**

它们仅在当前代码块（一个函数或一个循环体内）可以访问<br>
命名时不加任何前缀，依旧遵守小写下划线原则<br>

``` C
type_t Func(type_t arg_name) {
    type_t local_variable_name;
    /* ... */
}
```
举例<br>
``` C
void SRAM_WriteBytes(uint32_t offset, uint8_t* data_buffer, uint32_t count)
{
    __IO uint16_t *addr = (__IO uint16_t*)(BASE_ADDR + offset);
    uint16_t *data_buffer_16b = (uint16_t *)data_buffer;
    /* ... */
}
```

- **常量**

一般作为全局资源使用，类似全局变量但不可修改<br>
命名时不加任何前缀，依旧遵守小写下划线原则<br>

举例<br>
``` C
const uint8_t ascii_font_16x8[95][16] = { /* ... */ };
```

- **布尔逻辑类型**

加入前缀 **is_ has_** 表示逻辑条件<br>
可以与全局、静态变量的命名规则结合，注意他们的前缀放在最前面<br>

``` C
_Bool is_condition;
_Bool has_condition_value;
```

举例<br>
``` C
_Bool is_number;
_Bool is_screen_vertical;
_Bool g_has_extra_gain;
static _Bool s_is_tx_ready;
```

## 函数

- **函数命名**

与HAL库风格一致，采用组件名下划线大驼峰法结合<br>
部分函数可不带组件名前缀<br>

``` C
void ${组件名}_FunctionName(/* args */);
```

注意函数与参数的命名清楚、简洁、尽量让使用者能直接明白其功能<br>
某些长度较长的常用词可以使用缩写（在保证大家都认识的前提下）<br>
<br>
**部分常用缩写表:**
>   
    Init    : Initialize    初始化
    Str     : String        字符串
    Info    : Information   信息
    Err     : Error         错误
    Msg     : Message       消息
    Addr    : Address       地址
    Src     : Source        源地址
    Dst     : Destination   目的地址
    Reg     : Register      寄存器
    Clk     : Clock         时钟
    Freq    : Frequency     频率
    Amp     : Amplitude     幅度
    Mag     : Magnitude     大小
    Lib     : Library       库
    Img     : Image         图像
    Tx      : Transmit      发送
    Rx      : Receive       接收
这些缩写同样可以用在类型与变量命名中，注意对应的大小写规则即可<br>

举例<br>
``` C
void FATFS_Init(void);
void PE4302_SetAttenuation(uint8_t gain_2x_db);
uint16_t LCD_ReadPixel(uint16_t x, uint16_t y);

void DelayMs(uint16_t ms);
```

- **函数声明**

全局函数、静态局部函数与内联函数、外部函数的声明分别集中在一起<br>
函数声明统一在 **.h** 头文件中, 并保持完整的参数名<br>

举例<br>
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
extern void _ErrorHandler(uint8_t *file, uint32_t line);
```

- **函数定义**

函数定义（实现）统一在 **.c** 源文件中，并与头文件中的声明一一对应<br>

## 注释
如果使用中文等非ASCII字符进行注释<br>
为防止乱码，注意代码文件编码统一设置为 **UTF-8 无BOM**<br>
