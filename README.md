# ESP32-LED-Cookbook

ตัวอย่างที่นำมาใช้คือ การสร้าง ESP32 LED Component นำมาจาก Project ST67-Lab04-ESP32-LED-Component

### ส่วนของการแก้ไข 
มีดังนี้   ต้องการแก้ไขเพื่อ

### ขั้นตอนการทำ project
### 1.สร้าง project ใหม่ ชื่อ LED_ Project 

![image](https://github.com/user-attachments/assets/e8b66e5b-3448-4445-bfa9-73275c22bbf3)

เมื่อเลือกตามภาพเสร็จเรียบร้อยแล้วให้ทำการ กด Choose Tamplate

### 2.เมื่อมาถึงหน้านี้ให้ กด template-app และกด Create Project 

![image](https://github.com/user-attachments/assets/f561eea3-1d3e-4a85-aaf3-0f9c7d34e23c)

เนื้อหาในไฟล์ main.c 

![image](https://github.com/user-attachments/assets/3a308998-b7a8-4fd9-a8f6-666d54df7480)


### 3.สร้าง Component ใหม่ ชื่อ LED

![462574705_1089954725689616_227615653684703297_n](https://github.com/user-attachments/assets/d15ec18a-39dd-4655-b56e-273e8790c602)


![462574721_1189440645453950_6758193214623876322_n](https://github.com/user-attachments/assets/068422b7-d1f1-4f0e-a5df-246755be274a)


![image](https://github.com/user-attachments/assets/e94edf59-3675-448b-a5a0-92a7e1a6762b)


##### 3.1 เปลี่ยนชื่อไฟล์ LED.c เป็น LED.cpp
##### 3.2 เพิ่มการประกาศคลาสลงใน LED.h โดยให้ใส่ #include "driver/gpio.h" ด้วย เพื่อใช้งาน GPIO


![image](https://github.com/user-attachments/assets/9707c950-6825-460e-b3e7-12381397ad23)


``` cpp
LED.h
#include "driver/gpio.h" 

class LED
{
    int PinNumber;
public:
    LED(int PinNumber);
    void ON();
    void OFF();
};
```

##### 3.3 แก้ไขไฟล์ LED.cpp ให้เป็นดังนี้

![image](https://github.com/user-attachments/assets/dd268ca5-f1f7-4e36-85be-9580240a7d5d)


``` cpp
LED.cpp
  #include <stdio.h>
#include "LED.h"

LED::LED(int Pin)
{
    PinNumber = Pin;
    gpio_set_direction((gpio_num_t)PinNumber, GPIO_MODE_OUTPUT);
}

void LED::ON()
{
    gpio_set_level((gpio_num_t)PinNumber,1);
}

void LED::OFF()
{
    gpio_set_level((gpio_num_t)PinNumber,0);
}
```

##### 3.4 แก้ไขไฟล์ CMakeLists.txt ใน component/LED ให้เป็นดังต่อไปนี้

![image](https://github.com/user-attachments/assets/f570443b-4c77-46a7-ab72-b0e39d656220)


``` cpp
CMAkeLists.txt (ใน component/LED)
idf_component_register(SRCS "LED.cpp"
                    INCLUDE_DIRS "include"
                    REQUIRES driver)
```


### 4. แก้ไขไฟล์ต่าง ๆ ใน Main

##### 4.1 เปลี่ยนชื่อไฟล์ main.c เป็น main.cpp 

##### 4.2 แก้ไขไฟล์ CMakeLists.txt ใน Main เพื่อเปลี่ยนชื่อไฟล์ main.c เป็น main.cpp

![image](https://github.com/user-attachments/assets/253b5225-52a7-459a-9f4d-14d5d2de9d16)


``` cpp
CMAkeLists.txt (ใน Main)
idf_component_register(SRCS "main.cpp"
                    INCLUDE_DIRS "."
                    REQUIRES LED)
```

##### 4.3 แก้ไขไฟล์ main.cpp

![image](https://github.com/user-attachments/assets/9cae98ed-309c-4c34-b5b9-d47def112d9d)


``` cpp
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "LED.h"

extern "C" void app_main(void);

void app_main(void)
{
    LED led1(5); 
    LED led2(17); 
    while(1)
    {
        // กระพริบเร็ว 3 ครั้ง
        for (int i = 0; i < 3; i++) {
            led1.ON();
            led2.OFF();
            vTaskDelay(200 / portTICK_PERIOD_MS);
            led1.OFF();
            led2.ON();
            vTaskDelay(200 / portTICK_PERIOD_MS);
        }

        // กระพริบช้า 3 ครั้ง
        for (int i = 0; i < 3; i++) {
            led1.ON();
            led2.OFF();
            vTaskDelay(700 / portTICK_PERIOD_MS);
            led1.OFF();
            led2.ON();
            vTaskDelay(700 / portTICK_PERIOD_MS);
        }
    }
}
```

Build และทดสอบบนบอร์ด ESP32

ผลลัพน์คือ รูปแบบกระพริบเร็ว-ช้า (Quick-Fast Blink)
ทำให้ LED กระพริบเร็ว 3 ครั้ง จากนั้นกระพริบช้า 3 ครั้ง และทำซ้ำไปเรื่อย ๆ

วิดิโอประกอบ



##### 4.4 แก้ไขไฟล์ main.cpp

![image](https://github.com/user-attachments/assets/febb623d-e580-47fe-88ac-17f16b655cd8)



``` cpp
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "LED.h"

extern "C" void app_main(void);

#include "driver/ledc.h"

void app_main(void)
{
    LED led1(5); 
    while(1)
    {
        led1.ON();
        vTaskDelay(100 / portTICK_PERIOD_MS);  // ติดเร็ว
        led1.OFF();
        vTaskDelay(100 / portTICK_PERIOD_MS);  // ดับเร็ว
        led1.ON();
        vTaskDelay(100 / portTICK_PERIOD_MS);  // ติดอีกครั้ง
        led1.OFF();
        vTaskDelay(700 / portTICK_PERIOD_MS);  // ดับนานขึ้น (เหมือนการเต้นของชีพจร)
    }
}
```

Build และทดสอบบนบอร์ด ESP32


ผลลัพน์ คือ รูปแบบ "Heartbeat" (เต้นแบบชีพจร)
ให้ LED กระพริบเหมือนการเต้นของชีพจร โดยให้กระพริบติดเร็วแล้วดับอย่างช้า ๆ โดยใช้ LED ดวงเดียว

วิดิโอประกอบ




##### 4.5  แก้ไขไฟล์ main.cpp

![image](https://github.com/user-attachments/assets/cdcc5c00-63bf-4cf1-9c74-94f58e5e292e)


``` cpp
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "LED.h"

extern "C" void app_main(void);

#include "driver/ledc.h"

void app_main(void)
{
    LED leds[] = { LED(5), LED(17), LED(18), LED(19) }; // เชื่อมต่อ LED หลายตัวที่ GPIO ต่าง ๆ
    int numLeds = sizeof(leds) / sizeof(LED);

    while(1)
    {
        // วิ่งจากซ้ายไปขวา
        for (int i = 0; i < numLeds; i++) {
            leds[i].ON();
            vTaskDelay(100 / portTICK_PERIOD_MS);
            leds[i].OFF();
        }
        // วิ่งจากขวาไปซ้าย
        for (int i = numLeds - 2; i > 0; i--) {
            leds[i].ON();
            vTaskDelay(100 / portTICK_PERIOD_MS);
            leds[i].OFF();
        }
    }
}
```

Build และทดสอบบนบอร์ด ESP32


ผลลัพน์ คือ รูปแบบ "Knight Rider" (วิ่งซ้ายไปขวาและขวาไปซ้าย)
คล้ายกับเอฟเฟกต์ในซีรีส์ "Knight Rider" ซึ่ง LED จะวิ่งจากซ้ายไปขวาและกลับจากขวาไปซ้าย


วิดิโอประกอบ






สรุปผล: 





