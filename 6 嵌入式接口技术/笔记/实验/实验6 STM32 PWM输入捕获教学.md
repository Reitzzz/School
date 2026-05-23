# STM32 PWM输入捕获教学

## 实验目的

本实验的目的是：**用STM32定时器TIM2的输入捕获功能，测量TIM3输出的PWM信号的频率和占空比**。

实验5是"发送"PWM信号（输出），实验6是"接收"并测量PWM信号（输入）。就像实验5是说话，实验6是听别人说话并分析。

---

## 从问题出发：什么是输入捕获？

### PWM输出 vs 输入捕获

**实验5（PWM输出）**：
- TIM3产生一个PWM信号（100Hz，23.5%占空比）
- 信号从PA7引脚输出

**实验6（输入捕获）**：
- TIM2"监听"PA7引脚上的信号
- 测量信号的频率和占空比
- 结果在调试窗口显示

**类比**：PWM输出就像你用节拍器发出固定节奏；输入捕获就像你用耳朵听节拍器，数出每分钟多少拍。

### 输入捕获的原理

当引脚上的电平发生变化（上升沿或下降沿）时，定时器自动记录当前的计数值：

```
        ┌───┐       ┌───┐       ┌───┐
        │   │       │   │       │   │
────────┘   └───────┘   └───────┘   └───────
        ↑       ↑       ↑
      捕获1   捕获2   捕获3
      
捕获1和捕获2之间的时间 = 一个周期的时间
捕获1和捕获1.5之间的时间 = 高电平时间
频率 = 1 / 周期
占空比 = 高电平时间 / 周期时间
```

---

## 实验步骤：你需要写什么代码？

这个实验需要写3个地方的代码：
1. `PWM_driver.c` 中的 `PWMTest_GPIO_TIM2_Config()` — 配置输入捕获
2. `it.c` 中的 `TIM2_IRQHandler` — 处理捕获中断
3. `main.c` — 主程序

**注意**：`PWMGenerator_GPIO_TIM3_Config()` 已经写好了（产生PWM信号的配置），你只需要写测量端的代码。

---

### 实验报告截图1：输入捕获配置（PWM_driver）

**你需要写的代码**：

`PWM_driver.c` — 你需要补全的部分（填充到已有函数中）：

```c
void PWMTest_GPIO_TIM2_Config(void)
//配置通用定时器TIM2的PWM输入模式，测量TIM2_CH1上PWM_x信号频率和占空比
{
    GPIO_InitTypeDef GPIO_InitStructure;
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
    TIM_ICInitTypeDef TIM_ICInitStructure;
    NVIC_InitTypeDef NVIC_InitStructure;

    // ========== 第一步：配置GPIO为浮空输入 ==========
    // PA0 -> TIM2_CH1，作为PWM输入捕获引脚
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);

    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING; // 浮空输入
    GPIO_Init(GPIOA, &GPIO_InitStructure);

    // ========== 第二步：配置TIM2基本计数 ==========
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);

    TIM_TimeBaseStructure.TIM_Period = 0xFFFF;          // ARR = 65535（最大值）
    TIM_TimeBaseStructure.TIM_Prescaler = 719;          // PSC = 719，计数频率100kHz
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure);

    // ========== 第三步：配置PWM输入捕获通道 ==========
    // CH1直接输入，捕获上升沿，用来测周期
    TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;
    TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Rising;
    TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_DirectTI;
    TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;
    TIM_ICInitStructure.TIM_ICFilter = 0x00;
    TIM_IC1Init(TIM2, &TIM_ICInitStructure);

    // CH2间接输入，同样来自TI1，捕获下降沿，用来测高电平时间
    TIM_ICInitStructure.TIM_Channel = TIM_Channel_2;
    TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Falling;
    TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_IndirectTI;
    TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;
    TIM_ICInitStructure.TIM_ICFilter = 0x00;
    TIM_IC2Init(TIM2, &TIM_ICInitStructure);

    // ========== 第四步：配置从模式 ==========
    // 选择TI1FP1作为触发源；每次上升沿到来时自动复位计数器
    // 这样CCR1保存一个完整周期计数，CCR2保存高电平时间计数
    TIM_SelectInputTrigger(TIM2, TIM_TS_TI1FP1);
    TIM_SelectSlaveMode(TIM2, TIM_SlaveMode_Reset);
    TIM_SelectMasterSlaveMode(TIM2, TIM_MasterSlaveMode_Enable);

    // ========== 第五步：使能TIM2捕获中断 ==========
    TIM_ITConfig(TIM2, TIM_IT_CC1, ENABLE);

    // ========== 第六步：配置NVIC ==========
    NVIC_InitStructure.NVIC_IRQChannel = TIM2_IRQn;
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0;
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 0;
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NVIC_Init(&NVIC_InitStructure);

    // ========== 第七步：启动TIM2 ==========
    TIM_Cmd(TIM2, ENABLE);
}
```

**关键配置解释**：

| 配置项 | 值 | 含义 |
|--------|-----|------|
| GPIO_Mode | IN_FLOATING | 浮空输入，外部信号直接进来 |
| CH1 | Rising + DirectTI | 捕获上升沿，CCR1记录周期计数 |
| CH2 | Falling + IndirectTI | 捕获下降沿，CCR2记录高电平计数 |
| SlaveMode | Reset | 每次上升沿复位计数器，方便直接计算周期 |
| TIM_Period | 0xFFFF | 最大计数值（16位定时器） |

---

### 实验报告截图2：中断服务函数（it.c）

**你需要写的代码**：

`it.c` — 你需要补齐的中断函数代码：

```c
//通用定时器TIM2中断服务程序 
void TIM2_IRQHandler(void) 
//TIM2中断（PWM输入捕获中断）处理程序
{
    uint16_t PeriodValue;
    uint16_t PulseValue;

    if(TIM_GetITStatus(TIM2, TIM_IT_CC1) != RESET) // 检查是否是通道1捕获中断
    {
        PeriodValue = TIM_GetCapture1(TIM2); // CCR1：一个PWM周期的计数值
        PulseValue = TIM_GetCapture2(TIM2);  // CCR2：高电平时间的计数值

        if(PeriodValue != 0)
        {
            // TIM2计数频率 = 72MHz / (719 + 1) = 100kHz
            PWMFreqResult = 100000.0 / PeriodValue;
            PWMDutyResult = (float)PulseValue / PeriodValue;
        }

        TIM_ClearITPendingBit(TIM2, TIM_IT_CC1); // 清除中断标志位
    }
}
```

**关键点**：
1. `TIM_GetCapture1(TIM2)` 读到的是周期计数值
2. `TIM_GetCapture2(TIM2)` 读到的是高电平计数值
3. 频率 = 计数器频率 / 周期计数值
4. 占空比 = 高电平计数值 / 周期计数值

---

### 实验报告截图3：主程序（main.c）

**你需要写的代码**：

`main.c` — 你需要补齐的主程序代码：

```c
 int main(void)
 {
    // 配置TIM3产生PWM信号（已提供）
    PWMGenerator_GPIO_TIM3_Config();

    // 配置TIM2测量PWM信号（需要你写）
    PWMTest_GPIO_TIM2_Config();

    // 主循环：什么都不用做
    // TIM2自动捕获PWM信号，在调试窗口观察PWMDutyResult和PWMFreqResult
    while(1)
    {
        // 捕获结果通过全局变量PWMDutyResult和PWMFreqResult传递
        // 可以在Keil的Watch窗口中查看这两个变量的值
    }
 }
```

**运行效果**：
- TIM3在PA7输出100Hz、23.5%占空比的PWM信号
- TIM2在PA0捕获这个信号，测量结果显示在调试窗口
- `PWMFreqResult` 应该接近100Hz
- `PWMDutyResult` 应该接近0.235（23.5%）

---

## 标准库函数速查

| 你想做什么 | 用什么函数 | 示例 |
|-----------|-----------|------|
| 使能TIM2时钟 | `RCC_APB1PeriphClockCmd()` | `RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);` |
| 初始化定时器 | `TIM_TimeBaseInit()` | `TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure);` |
| 初始化CH1输入捕获 | `TIM_IC1Init()` | `TIM_IC1Init(TIM2, &TIM_ICInitStructure);` |
| 初始化CH2输入捕获 | `TIM_IC2Init()` | `TIM_IC2Init(TIM2, &TIM_ICInitStructure);` |
| 选择触发源 | `TIM_SelectInputTrigger()` | `TIM_SelectInputTrigger(TIM2, TIM_TS_TI1FP1);` |
| 配置从模式 | `TIM_SelectSlaveMode()` | `TIM_SelectSlaveMode(TIM2, TIM_SlaveMode_Reset);` |
| 使能捕获中断 | `TIM_ITConfig()` | `TIM_ITConfig(TIM2, TIM_IT_CC1, ENABLE);` |
| 获取周期捕获值 | `TIM_GetCapture1()` | `period = TIM_GetCapture1(TIM2);` |
| 获取高电平捕获值 | `TIM_GetCapture2()` | `pulse = TIM_GetCapture2(TIM2);` |
| 启动定时器 | `TIM_Cmd()` | `TIM_Cmd(TIM2, ENABLE);` |
| 检查中断标志 | `TIM_GetITStatus()` | `TIM_GetITStatus(TIM2, TIM_IT_CC1)` |
| 清除中断标志 | `TIM_ClearITPendingBit()` | `TIM_ClearITPendingBit(TIM2, TIM_IT_CC1);` |

---

## 常见错误

1. **GPIO模式用错**
   - 症状：捕获不到信号
   - 原因：输入捕获应该用 `GPIO_Mode_IN_FLOATING`（浮空输入），不是输出模式

2. **捕获极性配错**
   - 症状：频率测量不准
   - 原因：应该用上升沿触发，如果用下降沿需要调整计算逻辑

3. **忘记使能捕获中断**
   - 症状：TIM2在计数但中断不触发
   - 原因：需要 `TIM_ITConfig(TIM2, TIM_IT_CC1, ENABLE)`

4. **频率计算公式错误**
   - 症状：显示的频率值不对
   - 原因：计数器频率是100kHz（PSC=719），所以 `频率 = 100000 / 周期计数值`

5. **没有连接信号线**
   - 症状：什么都测量不到
   - 原因：需要一根杜邦线把PA7（TIM3输出）连到PA0（TIM2输入）

---

## 实验报告要点

实验报告中需要截图的代码包括：
1. `PWM_driver.c` 中 `PWMTest_GPIO_TIM2_Config()` 的完整代码
2. `it.c` 中 `TIM2_IRQHandler()` 的完整代码
3. `main.c` 的完整代码

---

## 硬件连接提示

本实验需要一根杜邦线：
```
PA7 (TIM3_CH2, PWM输出) ---杜邦线--- PA0 (TIM2_CH1, 输入捕获)
```
