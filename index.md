---
lang: zh_CN
---

# 电设软件

## 1. 基础硬件控制

### 电机驱动

<div align=left><img src="img\1571387948131.png" alt="1571387948131" style="zoom:10%;" /></div>

- **控制引脚定义：**

| 车轮 | 旋向控制（pin1，pin2） | PWM波输出 |
| :--: | :--------------------: | :-------: |
| 左前 |       （46，47）       |     2     |
| 右前 |       （48，49）       |     3     |
| 左后 |       （50，51）       |     5     |
| 右后 |       （52，53）       |     6     |

```c
static int mot_FL1 = 46;
static int mot_FL2 = 47;
static int mot_FR1 = 48;
static int mot_FR2 = 49;
static int mot_BL1 = 50;
static int mot_BL2 = 51;
static int mot_BR1 = 52;
static int mot_BR2 = 53;
static int pwm_fl = 2;
static int pwm_fr = 3;
static int pwm_bl = 5;
static int pwm_br = 6;
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
    digitalWrite(pin,LOW);
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



## 3. 控制算法

### 循迹



### 寻路&避障



### 状态切换




