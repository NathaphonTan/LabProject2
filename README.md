# LabProject2
# แนวทางการทำงาน ESP32 project cook book
## 1. ระบุตัวอย่างที่ใช้ ว่าเอามาจากตัวอย่างไหน
ตัวอย่างนี้มาจาก ESP32 Project Cookbook โดยในตัวอย่างเดิมจะใช้ MPU6050 เพื่ออ่านข้อมูล Acceleration และ Rotation ซึ่งจะใช้ไลบรารี Wire.h และ MPU6050.h ในการติดต่อกับเซ็นเซอร์
## 2. ระบุว่า จะแก้ไขตรงไหนบ้าง เพื่ออะไร
การเพิ่มฟังก์ชันการตรวจสอบอุณหภูมิ: MPU6050 สามารถอ่านอุณหภูมิจากเซ็นเซอร์ภายในได้ ซึ่งจะต้องเพิ่มการอ่านข้อมูลอุณหภูมิจากเซ็นเซอร์ในโค้ด โดยใช้ฟังก์ชัน getTemperature() ของไลบรารี MPU6050.h
การแก้ไข: เพิ่มการอ่านข้อมูลอุณหภูมิจากฟังก์ชัน mpu.getTemperature()
*คำนวณอุณหภูมิจากข้อมูลดิบที่ได้จากเซ็นเซอร์ (MPU6050 จะให้ค่าดิบที่ต้องแปลงเป็นเซลเซียส)*

การเพิ่มการอ่านอุณหภูมิ:
  ```cpp
tempRaw = mpu.getTemperature();
 ```
เพื่ออ่านข้อมูล raw ที่ได้จากเซ็นเซอร์ MPU6050
 
การแปลงข้อมูลอุณหภูมิ:

  ```cpp
float temperature = (tempRaw / 340.00) + 36.53;
 ```
ซึ่งคำนวณค่าจากข้อมูล raw ของ MPU6050 และแปลงให้เป็นอุณหภูมิในเซลเซียส
 
การแสดงผลอุณหภูมิ:


  ```cpp
Serial.print("Temperature: "); Serial.print(temperature);  // แสดงอุณหภูมิ
Serial.println(" C");
  ```
เพื่อแสดงอุณหภูมิที่คำนวณออกมาใน Serial Monitor
## 3. แสดงขั้นตอนการทำ project
**ขั้นตอนที่ 1 การต่อวงจร**
ในการทำโปรเจค ESP32 ร่วมกับ MPU6050 จะใช้การเชื่อมต่อแบบ I2C (Inter-Integrated Circuit) ระหว่าง ESP32 และ MPU6050 ซึ่งจะใช้พอร์ต SCL และ SDA สำหรับการสื่อสารข้อมูลระหว่างทั้งสองอุปกรณ์
**อุปกรณ์ที่ต้องใช้:**

ESP32 ( **ในที่นี้เนื่องจากบอร์ดมีปัญหาจึงใช้เบอร์ดที่มีอยู่คือ ESP32 DevKit V1**)
MPU6050 
สาย Jumper 

การเชื่อมต่อ:


| MPU6050 Pin |ESP32 Pin | 
| -------- | -------- | 
| VCC | 3.3V| 
| GND | GND | 
|SDA	|GPIO 21 (หรือ GPIO อื่นๆ ที่ตั้งค่าเป็น SDA ในโค้ด)|
|SCL|	GPIO 22 (หรือ GPIO อื่นๆ ที่ตั้งค่าเป็น SCL ในโค้ด)|
|INT	|ไม่จำเป็นต้องใช้ในการทำโปรเจคนี้| (สามารถปล่อยว่างไว้)|

**ขั้นตอนที่ 2 การลงไลบรารีใน Arduino IDE**
1. ติดตั้ง Arduino IDE
2. ติดตั้ง Board สำหรับ ESP32:
* เปิด Arduino IDE แล้วไปที่เมนู File → Preferences
* ในช่อง Additional Boards Manager URLs ให้ใส่ URL นี้:
https://dl.espressif.com/dl/package_esp32_index.json
* จากนั้นคลิก OK
* ไปที่ Tools → Board → Boards Manager แล้วพิมพ์ "ESP32" และคลิก Install ใน ESP32 by Espressif Systems
3. ติดตั้งไลบรารี MPU6050:
* ไปที่เมนู Sketch → Include Library → Manage Libraries
* ในช่องค้นหา ให้พิมพ์ "MPU6050" และเลือก MPU6050 by jrowberg แล้วคลิก Install
* ไฟล์ไลบรารีจะถูกติดตั้งและพร้อมใช้งานในโปรเจค
4. เลือกบอร์ดและพอร์ต:
* ไปที่เมนู Tools → Board แล้วเลือกบอร์ด ESP32 Dev Module หรือบอร์ดที่ใช้
* ไปที่เมนู Tools → Port และเลือกพอร์ตที่เชื่อมต่อกับ ESP32

**ขั้นตอนที่ 3 การเขียนโค้ดและอัปโหลด**
1. เปิด Arduino IDE และสร้างไฟล์ใหม่ File → New.
2. เขียนโค้ด:

 ```cpp
#include <Wire.h>
#include <MPU6050.h>

MPU6050 mpu;

void setup() {
  Serial.begin(115200);       
  Wire.begin();              
  mpu.initialize();         
}

void loop() {
  int16_t ax, ay, az;
  int16_t gx, gy, gz;
  int16_t tempRaw;


  mpu.getAcceleration(&ax, &ay, &az);

  mpu.getRotation(&gx, &gy, &gz);
  
 
  tempRaw = mpu.getTemperature();
  float temperature = (tempRaw / 340.00) + 36.53;  

 
  Serial.print("Accel X: "); Serial.print(ax);
  Serial.print(" Y: "); Serial.print(ay);
  Serial.print(" Z: "); Serial.println(az);
  
  Serial.print("Gyro X: "); Serial.print(gx);
  Serial.print(" Y: "); Serial.print(gy);
  Serial.print(" Z: "); Serial.println(gz);

  Serial.print("Temperature: "); Serial.print(temperature);  
  Serial.println(" C");

  delay(1000);  
}
```

3. อัปโหลดโค้ดไปยัง ESP32
* คลิกที่ปุ่ม Upload ใน Arduino IDE เพื่ออัปโหลดโค้ดไปยัง ESP32
* หลังจากที่โค้ดถูกอัปโหลดเสร็จสิ้น ให้เปิด Serial Monitor จากเมนู Tools → Serial Monitor
* ตั้งค่า baud rate ที่ 115200 ใน Serial Monitor
## 4. แสดงผลการทำ project
เมื่อเชื่อมต่อแล้ว จะเห็นข้อมูล Acceleration, Gyroscope และอุณหภูมิที่อ่านได้จาก MPU6050 

![ผล](https://github.com/user-attachments/assets/55cc5f8a-73d0-4afd-aa2f-b711d57766b8)


## 5. สรุปผลการทำ project 
โปรเจคนี้ช่วยให้สามารถใช้ ESP32 เชื่อมต่อกับเซ็นเซอร์ MPU6050 เพื่ออ่านข้อมูลการเคลื่อนที่ (Acceleration และ Rotation) พร้อมกับการตรวจสอบอุณหภูมิจากเซ็นเซอร์ได้ในเวลาเดียวกัน การเพิ่มฟังก์ชันการอ่านอุณหภูมิช่วยให้โปรเจคสามารถใช้งานได้หลากหลายมากขึ้น เช่น การตรวจสอบสภาพอุณหภูมิในห้องที่มีการเคลื่อนที่ หรือการตรวจสอบสภาวะของอุปกรณ์ที่มีการเคลื่อนที่
