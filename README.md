#Data-logger foundation project

# Data-logger foundation project

## Session 1


<details>
  <Summary>Electronic component selection and assembly</Summary>
  
_What bits, why, and what are the strengths and limitations_

>Parts List:

|Part | Cost | Description | Datasheet |
|-----|------|-------------|-----------|
| Micro controller | 2.96NZD | [ESP32-C3 SuperMini WIFI/Bluetooth Development Board](https://www.aliexpress.com/item/1005005877531694.html) | Datasheet |
| Temp/Humidty/Pressure Sensor | 3.27NZD | [BME280 3.3V](https://www.aliexpress.com/item/1005006098059302.html) | Datasheet |
| Real Time Clock module | 1.64NZD | [DS3231 I2C](https://www.aliexpress.com/item/32822420722.html) | Datasheet |
| TF Card module | 0.69NZD | [TF Card Memory Shield Module SPI](https://www.aliexpress.com/item/32346771288.html) | Datasheet |
| TF memory card | 3.00NZD | [Off-brand 16GB TF/micro SD Card](https://www.aliexpress.com/item/1005001617961938.html?algo_exp_id=3e8b6944-24e4-4192-9359-7344f5438bc5-4&utparam-url=scene%3Asearch%7Cquery_from%3A) | Datasheet |
| Prototyping PCB | 2.10NZ | [DKS-SOLDERBREAD-02](https://www.digikey.co.nz/en/products/detail/digikey/DKS-SOLDERBREAD-02/15970925) | Datasheet |
| Battery holder | 0.37NZD | [Single 18650](https://www.aliexpress.com/item/32580480645.html) | Datasheet |
| Battery 18650 | 12.49NZD | [Molicel M35A 18650 3500mAh 10A](https://cell-supply.co.nz/product/molicel-m35a-18650-3500mah-10a-battery/) | Datasheet |
| Charging Board | 0.51NZD | [TP4056](https://www.aliexpress.com/item/32467578996.html) | [Datasheet] |
| LDO Regulator | 0.76NZD | [MCP1700-3302E](https://www.digikey.co.nz/en/products/detail/microchip-technology/MCP1700-3302E-TO/652680?s=N4IgTCBcDaILIGEAKBGA7ABgwWgMy4zAFEQBdAXyA) | [Datasheet]((https://ww1.microchip.com/downloads/en/DeviceDoc/MCP1700-Data-Sheet-20001826F.pdf)) |
| 100uF Electrolytic Capacitor | 0.13NZD | [CAP ALUM 100UF 20% 10V RADIAL TH](https://www.digikey.com/en/products/detail/chinsan-elite/PF1A101MP20511F3U/16497042?s=N4IgTCBcDaICwEYCcCC0AFAYgggggDAgLLpj4CsCCmAzAKoDCAKqgHIAiIAugL5A) | Datasheet |
| 100nF Ceramic Capacitor | 0.09NZD | [CAP CER 0.1UF 50V X7R RADIAL](https://www.digikey.com/en/products/detail/kemet/C320C104M5R5TA/3726028?s=N4IgTCBcDaIMwE4EFoEHY0DZkDkAiIAugL5A) | Datasheet |
| 27K Ohm resistor | 0.04NZD | [RES 27K OHM 5% 1/4W AXIAL](https://www.digikey.com/en/products/detail/stackpole-electronics-inc/CFM14JT27K0/1742158?s=N4IgTCBcDaIMpgOwGkCKBhAKgWgHIBEQBdAXyA) | Datasheet |
| 100K Ohm resistor | 0.03NZD | [RES 100K OHM 5% 1/8W AXIAL](https://www.digikey.com/en/products/detail/stackpole-electronics-inc/CF18JT100K/1741564?s=N4IgTCBcDaIMIDECMAOAUgFSQBmwaTgwFoA5AERAF0BfIA) | Datasheet |


![Breadboard overview](https://github.com/user-attachments/assets/5d93920e-aa86-407c-98df-cfda81e5d644)

## Session 2


<details>
  <Summary>oftware setup and configuration</Summary>
  
_Setting up our boards to read sensors, format and save data, and communication and user interface design._


