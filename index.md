---
lang: zh_CN
---

# 电设软件

## 1. 基础硬件控制

### Arduino 引脚

PWM： 2-13；44-46；4&13 (980 Hz) others(490 Hz).

The PWM outputs generated on **pins 5 and 6** will have higher-than-expected duty cycles. This is because of interactions with the `millis()` and `delay()` functions, which share the same internal timer used to generate those PWM outputs. This will be noticed mostly on low duty-cycle settings (e.g. 0 - 10) and may result in a value of 0 not fully turning off the output on **pins 5** and **6**. 

外部中断：  2 (int 0), 3 (int 1). 18 (int 5), 19 (int 4), 20 (int 3), and 21 (int 2).

 SPI: 50 (MISO), 51 (MOSI), 52 (SCK), 53 (SS).  

 LED: 13.  

### 电机驱动

<img src="img\ourvehicle.jpg" alt="ourvehicle" style="zoom:40%;" />

- **控制引脚定义：**

| 车轮 | 旋向控制（pin1，pin2） |
| :--: | :--------------------: |
| 左前 |        （7，8）        |
| 右前 |       （9，10）        |
| 左后 |       （11，12）       |
| 右后 |       （44，45）       |

```c
static int pwm_fl1 = 7;
static int pwm_fl2 = 8;
static int pwm_fr1 = 9;
static int pwm_fr2 = 10;
static int pwm_bl1 = 11;
static int pwm_bl2 = 12;
static int pwm_br1 = 44;
static int pwm_br2 = 45;
```

注：PWM 频率为490 Hz

定义 （pin1，pin2）=（HIGH，LOW） → 向前       （LOW，HIGH）→ 向后

必须通过试验严格对应引脚与驱动电路输入，使之符合上述规律



- **基础运动函数：**

| 运动状态 | 函数名      | 定义                                                         |
| -------- | ----------- | ------------------------------------------------------------ |
| 前进     | FWD()       | mot fl,fr,bl,br $\longleftarrow$ (HIGH, LOW)                 |
| 后退     | BWD()       | mot fl,fr,bl,br $\longleftarrow$ (LOW, HIGH)                 |
| 右行     | RIGHT()     | mot fl,br$\longleftarrow$(HIGH, LOW) mot fr,bl$\longleftarrow$(LOW,HIGH) |
| 左行     | LEFT()      | mot fl,br$\longleftarrow$ (LOW, HIGH) mot fr,bl$\longleftarrow$(HIGH,LOW) |
| 中心转动 | SPIN()      | cw: mot fl,bl$\longleftarrow$(HIGH,LOW) mot fr,br$\longleftarrow$(LOW,HIGH)<br />ccw: mot fl,bl$\longleftarrow$(LOW,HIGH) mot fr,br$\longleftarrow$(HIGH,LOW) |
| 角点转动 | CORN()      | cw: mot fl,bl$\longleftarrow$(HIGH,LOW)  mot fr,br$\longleftarrow$(LOW,LOW)<br />ccw: mot fr,br$\longleftarrow$(HIGH,LOW) mot fl,bl$\longleftarrow$(LOW,LOW) |
| 对角移动 | DIAG(angle) | 45: mot fl,br$\longleftarrow$(HIGH,LOW) mot fr,bl$\longleftarrow$(LOW,LOW)<br />135: mot fr,bl$\longleftarrow$(HIGH,LOW) mot fl,br$\longleftarrow$(LOW,LOW)<br />-135: mot fl,br$\longleftarrow$(LOW,HIGH) mot fr,bl$\longleftarrow$(LOW,LOW)<br />-45: mot fr,bl$\longleftarrow$(LOW,HIGH) mot fr,bl$\longleftarrow$(LOW,LOW) |

- **代码实现**

```c
int pwm_value = 60;

void run(int pin,int pwm,int value)
{
    digitalWrite(pin,HIGH);
    digitalWrite(pin+1,LOW);
    analogWrite(pwm,value);
}
void run_r(int pin,int pwm,int value)
{
    digitalWrite(pin,LOW);
    digitalWrite(pin+1,HIGH);
    analogWrite(pwm,value);
}
void stop(int pin,int pwm)
{
    digitalWrite(pin,LOW);
    digitalWrite(pin+1,LOW);
    analogWrite(pwm,0);
}

void ALLSTOP()
{
    stop(mot_FL1,pwm_fl);
    stop(mot_FR1,pwm_fr);
    stop(mot_BL1,pwm_bl);
    stop(mot_BR1,pwm_br);
}

void FWD()
{
    run(mot_FL1,pwm_fl,pwm_value);
    run(mot_FR1,pwm_fr,pwm_value);
    run(mot_BL1,pwm_bl,pwm_value);
    run(mot_BR1,pwm_br,pwm_value);
}

void BWD()
{
    
}

void RIGHT()
{
    
}

void LEFT()
{
    
}

void SPIN()
{
    
}

void CORN()
{
    
}

void DIAG(int angle)
{
    
}
```



## 2.传感器

### 通讯模块



### 光电模块

引脚定义如下：

```c
//光电信号引脚定义
//1，2，3代表左中右。0代表旁路传感器
//并排三路传感器端为“方向舵”, 单路传感器为交点前向探测器
static int le_f1 = 46
static int le_f2 = 47
static int le_f3 = 48
static int le_b0 = 49
static int le_l1 = 50
static int le_l2 = 51
static int le_l3 = 52
static int le_r0 = 53
```



## 3. 控制算法

### 循迹



### 寻路

1. **交叉点计数的实现**

   迷宫区域内无位置信息，需要借助检测黑线交点并计数实现位置信息的更新。

   两大**关键**：全向移动小车如何识别交叉点；如何更好地避免重复计数

   

   **交叉点识别**

   正常寻迹时，只有中间的红外管检测到黑线，当全部红外管检测到黑线时，可认为**车头**到达了交叉点，但我们必须找到车中心到达交叉点的时刻，（因为我们的车辆是全向移动，前进方向的变换在交点处进行）车中心到达交叉点可用两侧传感器进行。

   为了使车辆能够更好地转换状态，需要在交叉点处停车。具体实现是，车头检测到交点后先减速，直至中心到达，车辆平稳停车。

   

   **避免重复计数**

   如果每个循环周期都读取一次传感器数据，则在经过黑线交点时应有多个循环对应检测到交点的状态，为避免重复计数，同时也是为了避免路面或传感器等不理想因素导致的错误信号，下面设计两种方法。

   1. 基于加速度计

      若检测到交点，检查加速度计在此时前进方向的位移积分，若此位移大致为一格的长度，可认为是交点，并且随即将积分值置0

   2. 基于两次检测

      当前端检测到交点时，forward_detected=1，交点（坐标）信息不更新。当两侧传感器检测到交点时，若forward_detected == 1，更新交点（坐标）信息。再置forward_detected=0；
   
      

2. **寻路算法**

3. **导航**

   导航，即在已知路径的情况下，控制小车运动的方向，到达目标点。并在前进遇到的障碍时进行报错。

   航向

4. 











### 状态切换

#### 人员救援



