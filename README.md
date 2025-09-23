# STM32 Servo Motor Control with PWM and Button Input

This project demonstrates how to control a **servo motor** using **PWM (Pulse Width Modulation)** on an STM32 microcontroller.  
The servo's position is adjusted based on **button inputs**, and **debouncing** is handled using a timer interrupt.

---

## Project Description

- The servo motor requires a **20 ms PWM period**:  
  - **0° position:** duty cycle = 1.5 ms  
  - **90° right:** duty cycle = 2 ms  
  - **90° left:** duty cycle = 1 ms  
- **Timer 2 (TIM2)** is configured in **PWM mode** to generate the servo signal.  
- **Timer 3 (TIM3)** is used to handle **debouncing** for the buttons with 1 ms intervals.  
- Two buttons (`BTN1` and `BTN2`) are used to **increase or decrease** the PWM compare value.  
- A variable `number` holds the current compare value for the PWM signal:
  - Starts at `18500` (corresponding to position 0).  
  - Increments or decrements by `100` per valid button press.

---

## Key Functions

### 1. Timer 2 Initialization (PWM)
```c
htim2.Init.Prescaler = 8-1;
htim2.Init.Period = 20000-1; // 20 ms period
sConfigOC.OCMode = TIM_OCMODE_PWM2;
sConfigOC.Pulse = 18500;     // Initial position
HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1);
````

* Generates PWM signals for the servo.
* `__HAL_TIM_SetCompare(&htim2, TIM_CHANNEL_1, number)` updates the duty cycle to move the servo.

### 2. Timer 3 Initialization (Debounce)

```c
htim3.Init.Prescaler = 800-1;
htim3.Init.Period = 10-1; // 1 ms interval
HAL_TIM_Base_Start_IT(&htim3);
```

* Runs every 1 ms to sample button states and handle **bouncing**.

### 3. Button Handling in Timer Interrupt

```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if(htim->Instance == TIM3){
        // BTN1 pressed
        if(btn1 == 100 && btn1_state == 0) {
            btn1_state = 1;
            if(number < 19500) number += 100;
            __HAL_TIM_SetCompare(&htim2, TIM_CHANNEL_1, number);
        }
        // BTN2 pressed
        if(btn2 == 100 && btn2_state == 0) {
            btn2_state = 1;
            if(number > 17500) number -= 100;
            __HAL_TIM_SetCompare(&htim2, TIM_CHANNEL_1, number);
        }
    }
}
```

* Detects stable button presses.
* Updates `number` to move the servo incrementally.
* Ensures the PWM duty cycle stays within valid range (17500–19500).

---

## Hardware Setup

* **STM32 Microcontroller** (e.g., STM32F1 series).
* **Servo motor** connected to `TIM2_CH1` (PWM output).
* **Buttons**:

  * `BTN1` → increase servo angle
  * `BTN2` → decrease servo angle
* Make sure **pull-up resistors** are enabled for the buttons.

The PWM period and duty cycle are calculated according to the servo datasheet.

---

## Advantages of This Implementation

* Smooth servo movement with **precise PWM control**.
* **Debounce timer** prevents false triggering from mechanical button noise.
* Incremental adjustments allow **fine control** of the servo position.

![Servomotor Datasheet](servomotor_datasheet.png)
