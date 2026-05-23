# STM32 PWM输出教学

## 实验目的

本实验的目的是：**用STM32定时器TIM3的PWM功能，在PB5引脚输出100Hz、25%占空比的PWM信号，控制LED0亮度**。

实验4用定时器中断让LED闪烁（亮/灭交替），实验5用PWM让LED亮度可调——通过快速切换亮灭，利用人眼的"视觉暂留"效应，看起来就是亮度变化。

---

## 从问题出发：什么是PWM？

### PWM的本质

PWM（Pulse Width Modulation，脉冲宽度调制）就是**快速地开关LED**：

- 占空比25%：亮1/4时间，灭3/4时间 → 看起来比较暗
- 占空比50%：亮一半时间，灭一半时间 → 看起来中等亮度
- 占空比100%：一直亮 → 最亮

**类比**：PWM就像你快速开关手电筒的开关。如果1秒内亮0.25秒灭0.75秒，重复很多次，看起来就是手电筒变暗了。

### 关键参数

```
        ┌───┐       ┌───┐       ┌───┐
        │   │       │   │       │   │
────────┘   └───────┘   └───────┘   └───────
        ←高电平→
        ←──── 一个周期 ────→

频率 = 1 / 周期时间 = 100Hz（每秒100个脉冲）
占空比 = 高电平时间 / 周期时间 = 25%（高电平占1/4）
```

---

## 实验步骤：你需要写什么代码？

这个实验需要写2个地方的代码：
1. `led_driver.c` 中的 `LED0_GPIO_TIM3_Config()` — 配置PWM输出
2. `main.c` — 主程序

---

### 实验报告截图1：PWM配置（led_driver）

**你需要写的代码**：

`led_driver.c` — 你需要补全的部分（填充到已有函数中）：

```c
void LED0_GPIO_TIM3_Config(void)
//配置GPIOB的TIM3接口，PB5输出100HZ,25%的PWM信号驱动LED0。
{
    GPIO_InitTypeDef GPIO_InitStructure;
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;
    TIM_OCInitTypeDef TIM_OCInitStructure;

    // ========== 第一步：计算定时器参数 ==========
    // 目标：100Hz的PWM，占空比25%
    // 公式：定时时间 = (PSC+1) × (ARR+1) / CK_INT

    u32 CK_INT_FREQ = 72000000;      // 内部时钟源频率 72MHz
    u32 CK_CNT_FREQ = 1000 * 1000;   // 计数器时钟频率 1MHz
    u16 Update_FREQ = 100;            // PWM频率 100Hz
    float Duty = 0.25;               // 占空比 25%

    // 预分频值 PSC：72MHz / 1MHz - 1 = 71
    u16 PrescalerValue = (u16)(CK_INT_FREQ / CK_CNT_FREQ) - 1;
    // ARR值：1MHz / 100Hz - 1 = 9999
    u16 ARRValue = (u16)(CK_CNT_FREQ / Update_FREQ) - 1;
    // CCR值（决定占空比）：0.25 × (9999+1) = 2500
    u16 CCRValue = (u16)(Duty * (ARRValue + 1));

    // ========== 第二步：配置GPIO为复用推挽输出 ==========
    // 注意：利用PB5作为TIM3的通道2，需要先开启AFIO时钟，然后执行部分重映射
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB | RCC_APB2Periph_AFIO, ENABLE);
    GPIO_PinRemapConfig(GPIO_PartialRemap_TIM3, ENABLE);

    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5;           // PB5
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;     // 复用推挽输出
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOB, &GPIO_InitStructure);

    // ========== 第三步：配置TIM3 ==========
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);

    // 3.1 基本定时参数
    TIM_TimeBaseStructure.TIM_Period = ARRValue;             // ARR = 9999
    TIM_TimeBaseStructure.TIM_Prescaler = PrescalerValue;    // PSC = 71
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;
    TIM_TimeBaseInit(TIM3, &TIM_TimeBaseStructure);

    // 3.2 配置PWM输出通道
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;             // PWM模式1
    TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable; // 使能输出
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;     // 极性配置
    TIM_OCInitStructure.TIM_Pulse = CCRValue;                     // CCR = 2500
    TIM_OC2Init(TIM3, &TIM_OCInitStructure);                      // 配置通道2（重映射到PB5）

    // ========== 第四步：启动定时器 ==========
    TIM_Cmd(TIM3, ENABLE);
}
```

**PWM参数计算详解**：

```
PSC = 72MHz / 1MHz - 1 = 71
ARR = 1MHz / 100Hz - 1 = 9999
CCR = 0.25 × (9999 + 1) = 2500

PWM周期 = (PSC+1) × (ARR+1) / 72MHz = 72 × 10000 / 72MHz = 0.01s = 100Hz ✓
高电平时间 = CCR / CK_CNT = 2500 / 1MHz = 0.0025s
占空比 = CCR / (ARR+1) = 2500 / 10000 = 25% ✓
```

**思考**：为什么GPIO模式是 `GPIO_Mode_AF_PP`（复用推挽）而不是 `GPIO_Mode_Out_PP`（普通推挽）？

**答案**：普通推挽输出由软件控制高低电平，而PWM需要定时器硬件自动控制引脚电平。"复用"的意思是把这个引脚的控制权交给外设（TIM3），而不是CPU。如果用普通推挽，CPU会和TIM3抢控制权，引脚状态会乱。

**思考**：为什么用 `TIM_OC2Init()` 而不是 `TIM_OC1Init()`？

**答案**：本实验要求使用TIM3_CH2，并把TIM3做部分重映射后输出到PB5，所以要用 `TIM_OC2Init()`。如果不用重映射，TIM3_CH2默认在PA7；开启 `GPIO_PinRemapConfig(GPIO_PartialRemap_TIM3, ENABLE)` 后，TIM3_CH2才映射到PB5。

---

### 实验报告截图2：主程序（main.c）

**你需要写的代码**：

`main.c` — 你需要补齐的主程序代码：

```c
 int main(void)
 {
    // 初始化LED0的TIM3 PWM输出
    LED0_GPIO_TIM3_Config();

    // 主循环：什么都不用做
    // TIM3硬件自动输出PWM信号，LED0以25%亮度常亮
    while(1)
    {
        // PWM信号由硬件自动产生，不需要CPU干预
    }
 }
```

**运行效果**：LED0以25%的亮度常亮（比完全点亮暗一些）。

---

## 标准库函数速查

| 你想做什么 | 用什么函数 | 示例 |
|-----------|-----------|------|
| 使能TIM3时钟 | `RCC_APB1PeriphClockCmd()` | `RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);` |
| 初始化定时器 | `TIM_TimeBaseInit()` | `TIM_TimeBaseInit(TIM3, &TIM_TimeBaseStructure);` |
| 初始化PWM通道 | `TIM_OC2Init()` | `TIM_OC2Init(TIM3, &TIM_OCInitStructure);` |
| 启动定时器 | `TIM_Cmd()` | `TIM_Cmd(TIM3, ENABLE);` |
| 修改占空比 | `TIM_SetCompare2()` | `TIM_SetCompare2(TIM3, 5000);` // 改为50% |

---

## 常见错误

1. **GPIO模式用错**
   - 症状：LED一直亮或一直灭，没有PWM效果
   - 原因：应该用 `GPIO_Mode_AF_PP`（复用推挽），不是 `GPIO_Mode_Out_PP`

2. **通道和引脚不匹配**
   - 症状：目标引脚没信号，其他引脚有信号
   - 原因：没有开启TIM3部分重映射时，TIM3_CH2默认在PA7；本实验要先开启部分重映射，才能让TIM3_CH2从PB5输出

3. **PSC/ARR/CCR计算错误**
   - 症状：频率或占空比不对
   - 原因：公式用错，注意CCR决定占空比，ARR决定频率

4. **忘记启动定时器**
   - 症状：配置都对但没有输出
   - 原因：必须调用 `TIM_Cmd(TIM3, ENABLE)`

---

## 实验报告要点

实验报告中需要截图的代码包括：
1. `led_driver.c` 中 `LED0_GPIO_TIM3_Config()` 的完整代码
2. `main.c` 的完整代码
