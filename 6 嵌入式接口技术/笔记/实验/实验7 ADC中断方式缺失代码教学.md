# 实验7 ADC中断方式缺失代码教学

## 实验目的

本实验的目标是：**使用 STM32 标准外设库，实现 ADC1 采集内部温度传感器数据，并用中断方式获取 AD 转换结果**。

实验报告中的要求可以拆成三句话：

1. 模拟信号源使用 STM32 内部温度传感器。
2. 使用定时器 3 每隔 1 秒触发一次 ADC 转换。
3. 在 ADC 中断服务函数中读取数字量，计算模拟电压和温度，并显示到 LCD。

所以你真正需要补全的代码主要有两个地方：

```text
USER_DEVICE_DRIVER/
  sensor_driver.c      -> 配置 TIM3、ADC1、NVIC

USER_APP/
  it.c                 -> 编写 ADC1_2_IRQHandler 中断服务函数
```

---

## 从问题出发：为什么要这样写？

### 第一步：为什么要配置 RCC 时钟？

STM32 的外设默认是关闭时钟的。  
如果你想使用 ADC1 或 TIM3，必须先给它们开启时钟。

```c
RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
```

**问题**：为什么 ADC1 用 `RCC_APB2PeriphClockCmd`，TIM3 用 `RCC_APB1PeriphClockCmd`？

**答案**：因为 ADC1 挂在 APB2 总线上，TIM3 挂在 APB1 总线上。  
不同外设挂在哪条总线上，要看 STM32 的芯片手册。做实验时可以先记住：

```text
GPIO、ADC1 通常在 APB2
TIM2、TIM3、TIM4 通常在 APB1
```

ADC 时钟还需要单独分频：

```c
RCC_ADCCLKConfig(RCC_PCLK2_Div6);
```

假设系统时钟是 72MHz，PCLK2 是 72MHz，那么 ADC 时钟就是：

```text
72MHz / 6 = 12MHz
```

这个频率符合 STM32F1 ADC 的工作要求。

---

### 第二步：为什么要用 TIM3 触发 ADC？

实验要求是“每隔 1s 触发一次 AD 转换”。  
如果在 `while(1)` 里手动启动 ADC，时间不稳定；用定时器就可以精准控制间隔。

TIM3 的配置代码如下：

```c
TIM_TimeBaseStructure.TIM_Period = 10000 - 1;
TIM_TimeBaseStructure.TIM_Prescaler = 7200 - 1;
TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;
TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
TIM_TimeBaseInit(TIM3, &TIM_TimeBaseStructure);
```

这里的时间计算是：

```text
TIM3 计数频率 = 72MHz / 7200 = 10000Hz
计数 10000 次 = 1 秒
```

所以：

```c
TIM_Prescaler = 7200 - 1;
TIM_Period    = 10000 - 1;
```

表示 TIM3 每 1 秒产生一次更新事件。

然后让 TIM3 的更新事件作为触发输出：

```c
TIM_SelectOutputTrigger(TIM3, TIM_TRGOSource_Update);
```

这句话的意思是：**TIM3 每次更新，就向外发出一个触发信号 TRGO**。

---

### 第三步：为什么 ADC 要选择 `ADC_ExternalTrigConv_T3_TRGO`？

因为实验要求 ADC 不是软件启动，而是由 TIM3 触发启动。

```c
ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_T3_TRGO;
ADC_ExternalTrigConvCmd(ADC1, ENABLE);
```

这两句配合起来表示：

```text
ADC1 等待 TIM3 的 TRGO 信号
TIM3 每 1 秒触发一次
ADC1 每 1 秒转换一次
```

如果忘记写：

```c
ADC_ExternalTrigConvCmd(ADC1, ENABLE);
```

那么 ADC 不会真正响应外部触发。

---

### 第四步：为什么要使用内部温度传感器通道？

实验中没有接外部模拟电压，而是采用 STM32 内部温度传感器作为模拟信号源。

STM32F1 内部温度传感器接在 ADC 通道 16：

```c
ADC_TempSensorVrefintCmd(ENABLE);
ADC_RegularChannelConfig(ADC1, ADC_Channel_TempSensor, 1, ADC_SampleTime_239Cycles5);
```

第一句打开内部温度传感器和内部参考电压通道。  
第二句告诉 ADC1：这次转换采集的是内部温度传感器。

采样时间选 `ADC_SampleTime_239Cycles5` 是比较稳妥的写法。  
内部温度传感器阻抗较大，采样时间太短容易导致结果不准。

---

### 第五步：为什么要配置 ADC 中断？

实验要求“采用中断方式获取 ADC 转换结果”。  
所以不能在主循环里一直查询 ADC，而应该在 ADC 转换完成后自动进入中断函数。

开启 ADC 转换完成中断：

```c
ADC_ITConfig(ADC1, ADC_IT_EOC, ENABLE);
```

`EOC` 是 End Of Conversion，意思是“转换结束”。

同时还要配置 NVIC：

```c
NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);

NVIC_InitStructure.NVIC_IRQChannel = ADC1_2_IRQn;
NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
NVIC_Init(&NVIC_InitStructure);
```

ADC1 和 ADC2 共用一个中断入口：

```c
void ADC1_2_IRQHandler(void)
```

所以中断通道要写 `ADC1_2_IRQn`。

---

## 实验步骤：你需要补全什么代码？

### 截图 1：补全 `sensor_driver.c`

这个文件负责配置外设。  
你可以理解成：先把硬件准备好，再让主程序运行。

完整代码如下：

```c
#include "stm32f10x_conf.h"

void SNESOR_TIM_ADC_NVIC_CONFIG(void)
{
    ADC_InitTypeDef ADC_InitStructure;
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
    NVIC_InitTypeDef NVIC_InitStructure;

    RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
    RCC_ADCCLKConfig(RCC_PCLK2_Div6);

    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    NVIC_InitStructure.NVIC_IRQChannel = ADC1_2_IRQn;
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NVIC_Init(&NVIC_InitStructure);

    TIM_TimeBaseStructure.TIM_Period = 10000 - 1;
    TIM_TimeBaseStructure.TIM_Prescaler = 7200 - 1;
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseInit(TIM3, &TIM_TimeBaseStructure);
    TIM_SelectOutputTrigger(TIM3, TIM_TRGOSource_Update);

    ADC_DeInit(ADC1);
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
    ADC_InitStructure.ADC_ScanConvMode = DISABLE;
    ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_T3_TRGO;
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
    ADC_InitStructure.ADC_NbrOfChannel = 1;
    ADC_Init(ADC1, &ADC_InitStructure);

    ADC_TempSensorVrefintCmd(ENABLE);
    ADC_RegularChannelConfig(ADC1, ADC_Channel_TempSensor, 1, ADC_SampleTime_239Cycles5);
    ADC_ITConfig(ADC1, ADC_IT_EOC, ENABLE);
    ADC_ExternalTrigConvCmd(ADC1, ENABLE);
    ADC_Cmd(ADC1, ENABLE);

    ADC_ResetCalibration(ADC1);
    while(ADC_GetResetCalibrationStatus(ADC1));
    ADC_StartCalibration(ADC1);
    while(ADC_GetCalibrationStatus(ADC1));

    TIM_Cmd(TIM3, ENABLE);
}
```

**代码理解顺序**：

```text
1. 开 ADC1 和 TIM3 时钟
2. 配置 ADC 中断通道
3. 配置 TIM3 每 1 秒更新一次
4. 配置 TIM3 更新事件作为 TRGO
5. 配置 ADC1 由 TIM3_TRGO 触发
6. 选择内部温度传感器通道
7. 开启 ADC 转换完成中断
8. 校准 ADC
9. 启动 TIM3
```

---

### 截图 2：补全 `it.c`

这个文件负责中断服务函数。  
当 ADC 转换完成时，程序会自动跳到：

```c
void ADC1_2_IRQHandler(void)
```

在中断函数中要做四件事：

```text
1. 判断是不是 ADC1 转换完成中断
2. 读取 ADC 数字量
3. 计算电压和温度
4. 显示到 LCD，并清除中断标志
```

完整代码如下：

```c
#include "lcd_driver.h"
#include "stm32f10x_conf.h"

static void LCD_ShowFixed3(u16 x, u16 y, u32 value)
{
    LCD_ShowNum(x, y, value / 1000, 1, 16);
    LCD_ShowChar(x + 8, y, '.', 16, 0);
    LCD_ShowxNum(x + 16, y, value % 1000, 3, 16, 0X80);
}

static void LCD_ShowTemperature(u16 x, u16 y, int temp_mc)
{
    u32 abs_temp;

    if(temp_mc < 0)
    {
        LCD_ShowChar(x, y, '-', 16, 0);
        abs_temp = (u32)(-temp_mc);
    }
    else
    {
        LCD_ShowChar(x, y, ' ', 16, 0);
        abs_temp = (u32)temp_mc;
    }

    LCD_ShowNum(x + 8, y, abs_temp / 1000, 2, 16);
    LCD_ShowChar(x + 24, y, '.', 16, 0);
    LCD_ShowxNum(x + 32, y, abs_temp % 1000, 3, 16, 0X80);
}

void ADC1_2_IRQHandler(void)
{
    u16 adc_value;
    u32 adc_mv;
    int temp_mc;

    if(ADC_GetITStatus(ADC1, ADC_IT_EOC) != RESET)
    {
        adc_value = ADC_GetConversionValue(ADC1);
        adc_mv = (u32)adc_value * 3300 / 4096;
        temp_mc = 25000 + ((1430 - (int)adc_mv) * 10000) / 43;

        POINT_COLOR = BLUE;
        LCD_ShowNum(180, 130, adc_value, 4, 16);
        LCD_ShowFixed3(180, 150, adc_mv);
        LCD_ShowTemperature(156, 170, temp_mc);

        ADC_ClearITPendingBit(ADC1, ADC_IT_EOC);
    }
}
```

---

## ADC 数据怎么算成电压？

ADC 是 12 位的，所以数字量范围是：

```text
0 ~ 4095
```

如果参考电压按 3.3V 计算，那么电压公式是：

```text
电压 = ADC数字量 / 4096 * 3.3V
```

为了避免浮点数运算，代码里用毫伏 mV 表示：

```c
adc_mv = (u32)adc_value * 3300 / 4096;
```

例如：

```text
adc_value = 2048
adc_mv = 2048 * 3300 / 4096 = 1650mV
显示为 1.650V
```

---

## 温度怎么算？

STM32F1 内部温度传感器常用公式是：

```text
T = 25 + (V25 - Vsense) / Avg_Slope
```

其中：

```text
V25       = 1.43V
AvgSlope  = 4.3mV/摄氏度
Vsense   = ADC 采集得到的传感器电压
```

换成毫伏计算：

```text
温度 = 25 + (1430 - adc_mv) / 4.3
```

代码中为了保留三位小数，把温度单位放大为“毫摄氏度”：

```c
temp_mc = 25000 + ((1430 - (int)adc_mv) * 10000) / 43;
```

这里：

```text
25000 表示 25.000 摄氏度
10000 / 43 等价于除以 4.3，同时保留三位小数
```

---

## 主程序 `main.c` 为什么不用改？

主程序已经完成了 LCD 初始化和外设初始化调用：

```c
LCD_Init();
SNESOR_TIM_ADC_NVIC_CONFIG();
```

然后它只负责显示固定标题：

```c
LCD_ShowString(60,130,200,16,16,"SENSOR_ADC_VAL:");
LCD_ShowString(60,150,200,16,16,"SENSOR_ADC_VOL:0.000V");
LCD_ShowString(60,170,200,16,16,"TEMPRETURE:00.000 C");
```

真正变化的数据由 ADC 中断函数负责刷新。  
所以 `while(1)` 里可以什么都不写：

```c
while(1)
{
}
```

这正是中断方式的特点：主程序不用一直等 ADC，ADC 转换完成后会自动进入中断。

---

## 标准库函数速查

| 你想做什么 | 使用函数 | 示例 |
|---|---|---|
| 开启 ADC1 时钟 | `RCC_APB2PeriphClockCmd()` | `RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);` |
| 开启 TIM3 时钟 | `RCC_APB1PeriphClockCmd()` | `RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);` |
| 配置 ADC 时钟分频 | `RCC_ADCCLKConfig()` | `RCC_ADCCLKConfig(RCC_PCLK2_Div6);` |
| 配置 TIM 基本计数 | `TIM_TimeBaseInit()` | `TIM_TimeBaseInit(TIM3, &TIM_TimeBaseStructure);` |
| 设置 TIM 触发输出 | `TIM_SelectOutputTrigger()` | `TIM_SelectOutputTrigger(TIM3, TIM_TRGOSource_Update);` |
| 配置 ADC | `ADC_Init()` | `ADC_Init(ADC1, &ADC_InitStructure);` |
| 选择 ADC 通道 | `ADC_RegularChannelConfig()` | `ADC_RegularChannelConfig(ADC1, ADC_Channel_TempSensor, 1, ADC_SampleTime_239Cycles5);` |
| 开启 ADC 中断 | `ADC_ITConfig()` | `ADC_ITConfig(ADC1, ADC_IT_EOC, ENABLE);` |
| 读取 ADC 值 | `ADC_GetConversionValue()` | `adc_value = ADC_GetConversionValue(ADC1);` |
| 清除中断标志 | `ADC_ClearITPendingBit()` | `ADC_ClearITPendingBit(ADC1, ADC_IT_EOC);` |

---

## 常见错误

1. **忘记开启 ADC 外部触发**

   症状：TIM3 已经启动，但 ADC 中断不进。

   原因：少写了这句：

   ```c
   ADC_ExternalTrigConvCmd(ADC1, ENABLE);
   ```

2. **TIM3 时间算错**

   症状：LCD 数据刷新太快或太慢。

   原因：`Prescaler` 和 `Period` 没有配成 1 秒。

   推荐写法：

   ```c
   TIM_Prescaler = 7200 - 1;
   TIM_Period = 10000 - 1;
   ```

3. **忘记开启内部温度传感器**

   症状：ADC 值不正常。

   原因：少写了：

   ```c
   ADC_TempSensorVrefintCmd(ENABLE);
   ```

4. **忘记清除 ADC 中断标志**

   症状：程序反复进入中断，运行异常。

   原因：中断服务函数最后没有清除标志位。

   正确写法：

   ```c
   ADC_ClearITPendingBit(ADC1, ADC_IT_EOC);
   ```

5. **LCD 小数部分不补零**

   症状：`1.005V` 显示成 `1.  5V` 或类似效果。

   原因：用了 `LCD_ShowNum()` 显示小数部分。  
   小数部分建议用 `LCD_ShowxNum(..., 0X80)`，这样高位 0 会显示出来。

---

## 实验报告截图要点

实验报告中建议截图以下代码：

1. `sensor_driver.c` 中的 `SNESOR_TIM_ADC_NVIC_CONFIG()` 完整函数。
2. `it.c` 中的 `ADC1_2_IRQHandler()` 完整函数。
3. LCD 上每 1 秒变化的 ADC 数字量、电压和温度结果。

截图前确认：

```text
1. Keil 工程中已经加入 RCC、ADC、TIM、NVIC 相关标准库文件
2. stm32f10x_conf.h 已经包含 stm32f10x_adc.h、stm32f10x_tim.h、stm32f10x_rcc.h、misc.h
3. 编译没有语法错误
4. ST-LINK 能正常识别开发板
```

