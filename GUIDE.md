# demo3 - STM32F103C8T6 竞赛机器人小车

## 项目概述

基于 STM32F103C8T6（Blue Pill）的多功能竞赛机器人小车，集成循迹、陀螺仪导航、舵机抓取和视觉识别功能。使用 STM32CubeIDE 生成 HAL 库框架，GCC 工具链编译。

## 硬件平台

| 项目 | 参数 |
|------|------|
| MCU | STM32F103C8T6 (Cortex-M3, 72MHz) |
| Flash | 64KB |
| RAM | 20KB |
| 封装 | LQFP48 |
| 调试接口 | SWD (PA13/PA14) |
| 外部晶振 | HSE 8MHz, LSE 32.768kHz |

## 目录结构

```
demo3/
├── Core/
│   ├── Inc/                          # 外设配置头文件
│   │   ├── main.h                    # 主头文件 + 引脚宏定义
│   │   ├── gpio.h / tim.h / i2c.h   # 各外设头文件
│   │   ├── usart.h
│   │   ├── stm32f1xx_hal_conf.h     # HAL 配置
│   │   └── stm32f1xx_it.h           # 中断服务声明
│   ├── Src/                          # 外设配置源文件
│   │   ├── main.c                    # ★ 主程序入口 + 5 个任务逻辑
│   │   ├── gpio.c                    # GPIO 初始化
│   │   ├── tim.c                     # 定时器初始化 (TIM1/2/3/4)
│   │   ├── i2c.c                     # I2C1/I2C2 初始化
│   │   ├── usart.c                   # USART1 初始化 + printf 重定向
│   │   ├── stm32f1xx_hal_msp.c      # HAL MSP 外设初始化
│   │   ├── stm32f1xx_it.c           # 中断服务函数
│   │   ├── system_stm32f1xx.c       # 系统时钟初始化
│   │   ├── syscalls.c / sysmem.c    # 系统调用 / 内存管理
│   └── Startup/
│       └── startup_stm32f103c8tx.s  # 启动汇编文件
├── Drivers/
│   ├── BSP/                          # ★ 板级驱动 (手写代码)
│   │   ├── LED/                      # LED 控制
│   │   ├── BEEP/                     # 蜂鸣器 (微秒级延时)
│   │   ├── KEY/                      # 按键输入 + 拍手检测
│   │   ├── OLED/                     # I2C OLED 128x64 显示
│   │   ├── MOTOR/                    # 直流电机 + 双舵机控制
│   │   ├── ENCODER/                  # 编码器读取 + 距离控制
│   │   ├── MPU6050/                  # 陀螺仪偏航角 + PID 运动控制
│   │   ├── Track/                    # 5 路红外循迹传感器
│   │   ├── CAMERA/                   # OpenMV 串口通信 (乒乓球识别)
│   │   └── ZDT_X42S/                 # ZDT_X42S 闭环步进电机驱动
│   ├── CMSIS/                        # ARM Cortex 内核文件
│   └── STM32F1xx_HAL_Driver/        # STM32 HAL 库
├── Debug/                            # 编译产物 (.elf, .map, .o)
├── .settings/                        # Eclipse / CubeIDE 项目配置
├── demo3.ioc                         # STM32CubeMX 引脚配置
├── STM32F103C8TX_FLASH.ld           # 链接脚本
├── demo3 Debug.launch                # 调试配置
├── .cproject / .project              # Eclipse 项目文件
└── .mxproject                        # CubeMX 项目元数据
```

## 外设配置一览

| 外设 | 引脚 | 功能 | 参数 |
|------|------|------|------|
| I2C1 | PB6(SCL), PB7(SDA) | OLED 显示屏 | 100kHz |
| I2C2 | PB10(SCL), PB11(SDA) | MPU6050 陀螺仪 | 400kHz |
| TIM1_CH4 | PA11 | 上舵机 PWM | 50Hz (20ms), pulse=1500 |
| TIM2_CH1/CH2 | PA0/PA1 | 编码器输入 | Encoder mode TI12 |
| TIM3_CH4 | PB1 | 下舵机 PWM | 50Hz (20ms), pulse=1500 |
| TIM4_CH3 | PB8 | 左电机 PWM | 高频 PWM |
| TIM4_CH4 | PB9 | 右电机 PWM | 高频 PWM |
| USART1 | PA9(TX), PA10(RX) | 摄像头 OpenMV | 115200-8-N-1 |
| USART2 | PA2(TX), PA3(RX) | ZDT_X42S 步进电机1 | 115200-8-N-1 |
| USART3 | PB10(TX), PB11(RX) | ZDT_X42S 步进电机2 | 115200-8-N-1 |
| Systick | - | HAL 时基 | 1ms |

### GPIO 引脚分配

| 引脚 | 标签 | 方向 | 用途 |
|------|------|------|------|
| PA4 | R2 | 输入上拉 | 循迹传感器 最右 |
| PA5 | R1 | 输入上拉 | 循迹传感器 右 |
| PA6 | M | 输入上拉 | 循迹传感器 中间 |
| PA7 | L1 | 输入上拉 | 循迹传感器 左 |
| PA8 | L2 | 输入上拉 | 循迹传感器 最左 |
| PA12 | Right2 | 输出 | 右电机方向 2 |
| PA15 | KEY1 | 输入上拉 | 任务选择按键 |
| PB0 | BEEP | 输出 | 无源蜂鸣器 |
| PB3 | Right1 | 输出 | 右电机方向 1 |
| PB4 | Left2 | 输出 | 左电机方向 2 |
| PB5 | Left1 | 输出 | 左电机方向 1 |
| PB13 | LED1 | 输出 | LED 指示灯 |
| PB14 | LED0 | 输出 | LED 指示灯 |
| PB15 | Clap | 输入上拉 | 拍手/声音传感器 |

## 系统架构

```
main() 启动流程:
  1. HAL_Init()                      # HAL 库初始化 + Systick 配置
  2. SystemClock_Config()            # 系统时钟 72MHz (HSE + PLL x9)
  3. MX_GPIO/I2C/TIM/USART_Init()   # 外设初始化
  4. OLED_Init() + OLED_Test()       # OLED 自检
  5. 启动编码器 / PWM 输出
  6. MPU6050 初始化 + 自检
  7. Stop_Motor()                    # 确保电机停止

  8. while(1) 主循环:
       - 读取按键选择任务 (1~5)
       - OLED 显示当前任务号
       - 等待拍手信号启动任务
       - 执行对应任务
```

## BSP 驱动 API

### LED (`Drivers/BSP/LED/`)
```c
void LED0_ON(void);        // 开灯
void LED0_OFF(void);       // 关灯
void LED0_TOGGLE(void);    // 翻转
```

### 蜂鸣器 (`Drivers/BSP/BEEP/`)
```c
void BEEP_ON(uint16_t times, uint16_t us);  // 鸣叫 times 次, 间隔 us 微秒
```

### 按键 (`Drivers/BSP/KEY/`)
```c
uint8_t Key_GetNum(uint8_t keynum);  // 按键循环切换任务号 (1~6)
uint8_t Key_Start(void);             // 拍手触发启动, 返回 1=启动
```

### OLED (`Drivers/BSP/OLED/`)
```c
void OLED_Init(void);
void OLED_Clear(void);
void OLED_ShowString(uint8_t Line, uint8_t Column, char *String);
void OLED_ShowNum(uint8_t Line, uint8_t Column, uint32_t Number, uint8_t Length);
void OLED_ShowSignedNum(uint8_t Line, uint8_t Column, int32_t Number, uint8_t Length);
void OLED_ShowFloat(uint8_t Line, uint8_t Column, float num, uint8_t int_len, uint8_t frac_len);
void OLED_Test(void);     // 全屏点亮测试
```

### 电机 (`Drivers/BSP/MOTOR/`)
```c
// 基础电机控制
void Set_leftMotor_front(uint16_t speed);   // 左轮前进
void Set_leftMotor_back(uint16_t speed);    // 左轮后退
void Set_rightMotor_front(uint16_t speed);  // 右轮前进
void Set_rightMotor_back(uint16_t speed);   // 右轮后退
void Stop_Motor(void);                      // 停止所有电机

// 舵机控制
void Servoup_Rotate(uint16_t angle);             // 上舵机直接控制 (500-2500)
void Servodown_Rotate(uint16_t angle);           // 下舵机直接控制 (500-2500)
void Servoup_Slow_Rotate(uint16_t target, uint16_t step_delay);   // 上舵机平滑转动
void Servodown_Slow_Rotate(uint16_t target, uint16_t step_delay); // 下舵机平滑转动

// 慢速转向 (视觉对准用)
void Turn_Left_Slow(void);
void Turn_Right_Slow(void);
```

### 编码器 (`Drivers/BSP/ENCODER/`)
```c
int16_t Get_EncoderNum(TIM_HandleTypeDef *htim);    // 读取编码器计数值
void Reset_Encoder(TIM_HandleTypeDef *htim);         // 重置编码器
void Go_Straight_Distance(float speed, float target_yaw, int16_t target_pulses);  // 走固定距离
```

### MPU6050 陀螺仪 (`Drivers/BSP/MPU6050/`)
```c
uint8_t MPU6050_Init(void);                    // 初始化 (唤醒传感器)
void MPU6050_Calibrate(uint16_t samples);      // 静止校准零偏
void MPU6050_ResetYaw(void);                   // 重置偏航角为 0
void MPU6050_Update(void);                     // 更新偏航角 (需周期性调用)
float MPU6050_GetYaw(void);                    // 获取当前偏航角

// 闭环运动控制
void Go_Straight_Yaw(float speed, float target_yaw, uint32_t duration_ms);   // 基于时间的直线行走
void Go_Straight_Yaw_Until_Line(float speed, float target_yaw);              // 直线走到黑线
void Rotate_To_Angle(float target_angle, int16_t speed);                     // 旋转到指定角度
void Rotate_To_Angle1(float target_angle, int16_t speed);                    // 旋转变体 (考虑惯性)
```

### 循迹 (`Drivers/BSP/Track/`)
```c
uint8_t GetTrackStatus(void);                       // 读取 5 路传感器状态
void Track_Go_Straight(uint32_t duration_ms);       // 循迹前进 (基于时间)
void Track_Back_Straight(uint32_t duration_ms);     // 循迹后退
void Rotate_to_line(uint8_t speed);                 // 旋转直到找到黑线
```

### 摄像头 (`Drivers/BSP/CAMERA/`)
```c
typedef struct { int16_t x; int16_t y; } Pingpong_Coord_t;

uint8_t Receive_Pingpong_Coord(Pingpong_Coord_t *coord, uint32_t timeout_ms);  // 接收坐标
void Align_To_Pingpong(void);    // 自动对准乒乓球并抓取
```

## 5 个竞赛任务

按键 **KEY1** 循环选择任务 (keynum 1~5), 拍手传感器 **Clap** 触发启动。

| 任务 | 功能描述 | 核心逻辑 |
|------|---------|---------|
| 1 | **纯循迹行驶** | 循迹前进 4100ms → 旋转找线 → 循迹 3200ms |
| 2 | **方形路线** | 直走到线 → 转-90° → 循迹 → 转-90° → 直走到线 → 转-92° → 循迹 |
| 3 | **多段导航** | 循迹 → 转90°×5 次 → 直走到线 → 旋转找线 → 循迹 |
| 4 | **物块抓取** | 舵机下降 → 前进 → 舵机夹取 → 舵机上升 → 前进 → 放置 |
| 5 | **物块搬运** | 抓取 → 近距离前进 → 转-90° → 前进 → 夹取 → 上升 → 转-90° → 放置 |

### 步进电机 ZDT_X42S (`Drivers/BSP/ZDT_X42S/`)

ZDT_X42S 闭环步进电机驱动，通过串口协议控制，1.8°/16细分 = 3200脉冲/圈。

```c
// 电机句柄 (每个电机一个实例)
typedef struct {
    UART_HandleTypeDef *huart;  // 对应 USART2 或 USART3
    uint8_t addr;               // 电机地址 (1-247, 默认1)
} ZDT_HandleTypeDef;

// 初始化
void ZDT_Init(ZDT_HandleTypeDef *zdt, UART_HandleTypeDef *huart, uint8_t addr);

// 快速位置模式（推荐）— 设定参数 + 发送脉冲
void ZDT_FastPosSetParams(ZDT_HandleTypeDef *zdt, uint16_t speed_rpm,
                          uint8_t acc, uint8_t mode);
void ZDT_FastPosMove(ZDT_HandleTypeDef *zdt, int32_t pulses);
void ZDT_FastPosMoveAngle(ZDT_HandleTypeDef *zdt, float angle_deg,
                          uint16_t speed_rpm, uint8_t acc, uint8_t mode);

// 标准位置模式 — 一条指令控制全部参数
void ZDT_PosCtrl(ZDT_HandleTypeDef *zdt, uint8_t dir, uint16_t speed_rpm,
                 uint8_t acc, int32_t pulses, uint8_t mode);

// 速度模式 — 连续旋转
void ZDT_SpeedCtrl(ZDT_HandleTypeDef *zdt, uint8_t dir, uint16_t speed_rpm,
                   uint8_t acc);

// 回零操作
void ZDT_SetHomeZero(ZDT_HandleTypeDef *zdt, uint8_t store);
void ZDT_TriggerHome(ZDT_HandleTypeDef *zdt, uint8_t mode);
void ZDT_AbortHome(ZDT_HandleTypeDef *zdt);
uint8_t ZDT_ReadHomeStatus(ZDT_HandleTypeDef *zdt);

// 单位换算
int32_t ZDT_AngleToPulses(float angle_deg);
float ZDT_PulsesToAngle(int32_t pulses);
```

**运动模式宏定义:**
- `ZDT_MODE_REL_PREV` (0) — 相对于上一目标位置
- `ZDT_MODE_ABS_ZERO` (1) — 相对坐标零点绝对位置
- `ZDT_MODE_REL_CURRENT` (2) — 相对当前实时位置 (推荐)

**使用示例:**
```c
ZDT_HandleTypeDef motor1, motor2;

// 初始化两个电机
ZDT_Init(&motor1, &huart2, 1);  // 地址1, USART2
ZDT_Init(&motor2, &huart3, 1);  // 地址1, USART3

// 电机1 顺时针转90度, 速度800RPM, 加速度100, 相对运动
ZDT_FastPosMoveAngle(&motor1, 90.0f, 800, 100, ZDT_MODE_REL_CURRENT);

// 电机2 逆时针转45度
ZDT_FastPosMoveAngle(&motor2, -45.0f, 800, 100, ZDT_MODE_REL_CURRENT);
```

## 通信协议

### OpenMV 摄像头串口协议

UART 通信, 115200-8-N-1, 自定义协议栈:
```
包头: 0xFF
数据: X坐标(2字节, 低字节在前) + Y坐标(2字节, 低字节在前)
包尾: 0xFE

帧格式: [0xFF] [xL] [xH] [yL] [yH] [0xFE]
```

控制指令:
- `0x01` — 开始识别乒乓球
- `0x02` — 停止识别

## 编译与烧录

### 编译环境
- **IDE:** STM32CubeIDE (版本 6.14.1)
- **工具链:** GCC ARM Embedded
- **HAL 库:** STM32Cube FW_F1 V1.8.7
- **CubeMX 版本:** 6.14.1

### 编译
在 STM32CubeIDE 中: `Project → Build All` (或 Ctrl+B)
产物: `Debug/demo3.elf`, `demo3.hex`, `demo3.bin`

### 烧录
使用 ST-Link / J-Link / DAP-Link 通过 SWD 接口 (PA13-SWDIO, PA14-SWCLK):
- 在 STM32CubeIDE 中点击 `Run` 或 `Debug`
- 或使用 STM32CubeProgrammer

### 配置参数
- Flash 起始地址: `0x08000000` (64KB)
- RAM 起始地址: `0x20000000` (20KB)
- 堆大小: 0x200 (512B)
- 栈大小: 0x400 (1024B)
- NVIC 优先级分组: Group 4
- SysTick 优先级: 1
- USART1 中断优先级: 3

## 代码规范说明

- **STM32CubeIDE 用户代码区域:** `USER CODE BEGIN` / `USER CODE END` 注释之间的代码在 CubeMX 重新生成时会被保留
- **手写 BSP 驱动:** `Drivers/BSP/` 下的代码完全手动编写, 不受 CubeMX 管理
- **`extern` 声明:** BSP 驱动通过 `extern` 引用 `main.c`/_`it.c` 中定义的 HAL 句柄 (如 `htim2`, `huart1`, `hi2c2`)
- **`printf` 重定向:** 在 `usart.c` 中通过 `__io_putchar()` 实现, 输出到 USART1

## 相关资料

- STM32F103C8T6 数据手册: STM32F103x8/xB Datasheet
- STM32F1 HAL 驱动描述: UM1850
- MPU6050 数据手册: InvenSense MPU-6050
- 开发板参考: STM32F103C8T6 最小系统板 (Blue Pill)
