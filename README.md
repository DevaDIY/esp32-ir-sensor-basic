# esp32-ir-sensor-basic
โครงงานตัวอย่างการใช้งาน ESP32 ร่วมกับ IR Sensor เพื่อตรวจจับวัตถุ เมื่อมีวัตถุผ่านจะสั่งให้ LED สว่างค้าง 5 วินาที หากไม่มีวัตถุจะให้ LED กระพริบทุก ๆ 500ms
# ESP32 + IR Sensor Project

โครงงานตัวอย่างการใช้งาน **ESP32 ร่วมกับ IR Sensor** เพื่อตรวจจับวัตถุ เมื่อมีวัตถุผ่านจะสั่งให้ **LED สว่างค้าง 5 วินาที** หากไม่มีวัตถุจะให้ **LED กระพริบทุก ๆ 500ms**

📖 อ่านบทความอธิบายละเอียดที่ [ESP32 IR Sensor – devadiy.com](https://devadiy.com/esp32-ir-sensor/)  
🌐 แหล่งรวมโครงงาน ESP32 เพิ่มเติมที่ [Deva DIY](https://devadiy.com/)

---

## Hardware ที่ใช้
- ESP32 DevKit V1
- IR Sensor (โมดูลตรวจจับวัตถุอินฟราเรด)
- LED + ตัวต้านทาน 220Ω
- สาย Jumper

---

## การต่อวงจร
- IR Sensor → GPIO32 (Sensor Input)  
- LED → GPIO25 (Output)

---

## Source Code

```cpp
/* Deva DIY */
 
// กำหนด GPIOs ของ Sensor และ LED
const int led = 25;
const int Sensor = 32;
 
// ตัวแปร Timer
uint32_t now_time = millis(); // บันทึกเวลาปัจจุบันที่ทำงาน
uint32_t lasttime_trigger = 0;   // บันทึกเวลาเมื่อตรวจจับวัตถุ
uint32_t led_time = 0; // เวลากระพริบ LED
 
// Flag เมื่อเกิด interrupt กำหนดเป็น True
bool Sensor_Check = false;
 
// ตัวแปรสถานะ HIGH or LOW LED
int State_led = 0;
 
// เมื่อเซนเซอร์ตรวจพบวัตถุตัดผ่าน เกิด Interrupt
void IRAM_ATTR IRSensordetects() {
  Sensor_Check = true;
  lasttime_trigger = millis() + 5000;
}
 
void setup() {
  Serial.begin(115200);
 
  // กำหนดให้ pin sensor เป็น INPUT
  pinMode(Sensor, INPUT);
 
  // ตั้งค่า Interrupt ในฟังก์ชั่น
  attachInterrupt(digitalPinToInterrupt(Sensor), IRSensordetects, FALLING);
 
  // กำหนดให้ LED เป็น OUTPUT
  pinMode(led, OUTPUT);
  digitalWrite(led, State_led);
}
 
void loop() {
  // กระพริบ LED ทุก 500 ms ถ้าไม่พบวัตถุ
  if ((led_time <= millis()) && (!Sensor_Check)) {
    led_time = millis() + 500;
    State_led = !State_led;
    digitalWrite(led, State_led);
  }
 
  // ถ้าพบวัตถุ → LED สว่างค้าง 5 วินาที
  now_time = millis();
  if (Sensor_Check) {
    State_led = 1;
    digitalWrite(led, State_led);
    Serial.println("SENSOR DETECTED");
    delay(100);
 
    if (now_time >= lasttime_trigger) {
      Sensor_Check = false;
      State_led = 0;
      digitalWrite(led, State_led);
    }
  }
}
