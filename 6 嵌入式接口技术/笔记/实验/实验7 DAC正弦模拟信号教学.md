# 实验7 DAC正弦模拟信号教学

> 对应期末复习要点：第7章例题7.3，或实验7“中断方式产生正弦模拟信号”的控制程序。考试程序设计题可能给出库函数附件，要求按 GPIO、TIM、DAC 的标准库函数写初始化和中断服务程序。

## 目标

用 STM32 输出一个近似正弦波：

1. 准备一张正弦表 `sin_tab[]`，表中每个数是 DAC 要输出的数字量。
2. 配置 `PA4` 为模拟输入模式，对应 `DAC_OUT1`。
3. 配置 DAC 通道 1。
4. 配置定时器中断，定时进入 `TIMx_IRQHandler`。
5. 每次中断取正弦表的下一个点，写入 DAC。

核心思路一句话：

```text
定时器中断控制“什么时候换点”，DAC负责“把数字量变成模拟电压”。
```

## 常用库函数速查

| 用途 | 函数 | 典型写法 |
| --- | --- | --- |
| 开 GPIOA 时钟 | `RCC_APB2PeriphClockCmd()` | `RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);` |
| 开 DAC 时钟 | `RCC_APB1PeriphClockCmd()` | `RCC_APB1PeriphClockCmd(RCC_APB1Periph_DAC, ENABLE);` |
| 初始化 GPIO | `GPIO_Init()` | `GPIO_Init(GPIOA, &GPIO_InitStructure);` |
| 初始化 DAC | `DAC_Init()` | `DAC_Init(DAC_Channel_1, &DAC_InitStructure);` |
| 使能 DAC | `DAC_Cmd()` | `DAC_Cmd(DAC_Channel_1, ENABLE);` |
| 写 DAC 数据 | `DAC_SetChannel1Data()` | `DAC_SetChannel1Data(DAC_Align_12b_R, data);` |
| 软件触发 DAC | `DAC_SoftwareTriggerCmd()` | `DAC_SoftwareTriggerCmd(DAC_Channel_1, ENABLE);` |
| 初始化定时器 | `TIM_TimeBaseInit()` | `TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure);` |
| 开定时器中断 | `TIM_ITConfig()` | `TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);` |
| 启动定时器 | `TIM_Cmd()` | `TIM_Cmd(TIM2, ENABLE);` |
| 检查/清除中断标志 | `TIM_GetITStatus()` / `TIM_ClearITPendingBit()` | `TIM_ClearITPendingBit(TIM2, TIM_IT_Update);` |

## 初始化步骤

### 1. 配置 DAC 输出引脚

DAC 通道 1 通常从 `PA4` 输出，所以 GPIO 要配置为模拟模式：

```c
GPIO_InitTypeDef GPIO_InitStructure;

RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);

GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
GPIO_Init(GPIOA, &GPIO_InitStructure);
```

这里不要配置成推挽输出，因为 DAC 输出的是模拟电压，不是普通数字高低电平。

### 2. 配置 DAC 通道 1

```c
DAC_InitTypeDef DAC_InitStructure;

RCC_APB1PeriphClockCmd(RCC_APB1Periph_DAC, ENABLE);

DAC_InitStructure.DAC_Trigger = DAC_Trigger_None;
DAC_InitStructure.DAC_WaveGeneration = DAC_WaveGeneration_None;
DAC_InitStructure.DAC_LFSRUnmask_TriangleAmplitude = DAC_LFSRUnmask_Bit0;
DAC_InitStructure.DAC_OutputBuffer = DAC_OutputBuffer_Enable;
DAC_Init(DAC_Channel_1, &DAC_InitStructure);

DAC_Cmd(DAC_Channel_1, ENABLE);
```

如果题目要求软件触发，可以把触发方式写成软件触发，并在写数据后调用软件触发函数：

```c
DAC_InitStructure.DAC_Trigger = DAC_Trigger_Software;
DAC_SetChannel1Data(DAC_Align_12b_R, data);
DAC_SoftwareTriggerCmd(DAC_Channel_1, ENABLE);
```

### 3. 配置定时器更新中断

定时器决定正弦表“换点”的速度。假设用 `TIM2`：

```c
TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
NVIC_InitTypeDef NVIC_InitStructure;

RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);

TIM_TimeBaseStructure.TIM_Prescaler = 7199;
TIM_TimeBaseStructure.TIM_Period = 99;
TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;
TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure);

TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);

NVIC_InitStructure.NVIC_IRQChannel = TIM2_IRQn;
NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0;
NVIC_InitStructure.NVIC_IRQChannelSubPriority = 0;
NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
NVIC_Init(&NVIC_InitStructure);

TIM_Cmd(TIM2, ENABLE);
```

如果系统时钟为 72MHz，上面参数表示：

```text
计数频率 = 72MHz / (7199 + 1) = 10kHz
更新频率 = 10kHz / (99 + 1) = 100Hz
```

也就是每 `10ms` 输出正弦表的下一个点。

## 中断服务程序

```c
const uint16_t sin_tab[100] = {
    /* 这里放 0~4095 范围内的 100 个正弦采样点 */
};

static uint8_t index = 0;

void TIM2_IRQHandler(void)
{
    if (TIM_GetITStatus(TIM2, TIM_IT_Update) != RESET)
    {
        TIM_ClearITPendingBit(TIM2, TIM_IT_Update);

        DAC_SetChannel1Data(DAC_Align_12b_R, sin_tab[index]);

        index++;
        if (index >= 100)
        {
            index = 0;
        }
    }
}
```

这段代码最重要的三件事：

1. 先检查是不是 `TIM2` 的更新中断。
2. 进入中断后必须清除中断标志位，否则会一直进中断。
3. 每次写入正弦表的下一个点，表走完后从头开始。

## 频率计算

如果正弦表有 `N` 个点，每个点间隔为 `T_update`，则输出正弦波周期为：

```text
T_sin = N * T_update
```

输出正弦波频率为：

```text
f_sin = 1 / T_sin
```

例如 100 点表，每 `2ms` 更新一次：

```text
T_sin = 100 * 2ms = 200ms
f_sin = 1 / 0.2s = 5Hz
```

## 考试答题模板

程序设计题可以按这个顺序写：

1. 开时钟：`GPIOA`、`DAC`、`TIMx`。
2. 配 GPIO：`PA4` 配成 `GPIO_Mode_AIN`。
3. 配 DAC：`DAC_Init()`，再 `DAC_Cmd()`。
4. 配 TIM：`TIM_TimeBaseInit()`，`TIM_ITConfig()`，`NVIC_Init()`，`TIM_Cmd()`。
5. 写中断服务函数：清标志，写 DAC 数据，更新数组下标。

## 易错点

| 易错点 | 正确理解 |
| --- | --- |
| 把 PA4 配成推挽输出 | DAC 输出模拟量，PA4 应配 `GPIO_Mode_AIN`。 |
| 忘记开 DAC 时钟 | DAC 在 APB1 上，用 `RCC_APB1PeriphClockCmd(RCC_APB1Periph_DAC, ENABLE);`。 |
| 忘记清 TIM 中断标志 | ISR 中必须调用 `TIM_ClearITPendingBit()`。 |
| 正弦表下标越界 | `index >= N` 后要清零。 |
| 不理解软件触发 | 软件触发模式下，写数据后还要 `DAC_SoftwareTriggerCmd()` 启动转换。 |
