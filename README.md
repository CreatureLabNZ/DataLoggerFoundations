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
</Details>

## Session 2


<details>
  <Summary>Software setup and configuration</Summary>  

_Setting up our boards to read sensors, format and save data, and communication and user interface design_

**Step 1:**

<details><summary>Minimum working example to set up over-the-air software uploads and blink and LED from a web interface</summary>
  
```c++
#include <WiFi.h>
#include <ESPmDNS.h>
#include <WiFiUdp.h>
#include <ArduinoOTA.h>
#include <WebServer.h>

const char* ssid = "WIFI NETWORK NAME"; ## Wifi name
const char* password = "PASSWORD"; ## Wifi password

#set the address correctly for your network below
IPAddress local_ip(192,168,1,222);
IPAddress gateway(192,168,1,254);
IPAddress subnet(255,255,255,0);

WebServer server(80);

//variabls for blinking an LED with Millis

const int led = 4; // ESP32 Pin to which onboard LED is connected
// unsigned long previousMillis = 0;  // will store last time LED was updated
// const long interval = 1000;  // interval at which to blink (milliseconds)
bool ledState = LOW;  // ledState used to set the LED


void setup() {

  pinMode(led, OUTPUT);  
  Serial.begin(115200);
  Serial.println("Booting");

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
    while (WiFi.waitForConnectResult() != WL_CONNECTED) {
      Serial.println("Connection Failed! Rebooting...");
      delay(5000);
      ESP.restart();
    }

  ArduinoOTA.setPort(3232); // Port defaults to 3232
  ArduinoOTA.setHostname("creature_log");// Hostname defaults to esp3232-[MAC] ## Need to set a unique hostname for your device!
  ArduinoOTA.setPassword("admin"); // No authentication by default
  
  // MD5(admin) = 21232f297a57a5a743894a0e4a801fc3  // Password can be set with it's md5 value as well
  // ArduinoOTA.setPasswordHash("21232f297a57a5a743894a0e4a801fc3");
  ArduinoOTA
    .onStart([]() {
      String type;
      if (ArduinoOTA.getCommand() == U_FLASH)
        type = "sketch";
      else // U_SPIFFS
        type = "filesystem";
      // NOTE: if updating SPIFFS this would be the place to unmount SPIFFS using SPIFFS.end()
      Serial.println("Start updating " + type);
    })
    .onEnd([]() {
      Serial.println("\nEnd");
    })
    .onProgress([](unsigned int progress, unsigned int total) {
      Serial.printf("Progress: %u%%\r", (progress / (total / 100)));
    })
    .onError([](ota_error_t error) {
      Serial.printf("Error[%u]: ", error);
      if (error == OTA_AUTH_ERROR) Serial.println("Auth Failed");
      else if (error == OTA_BEGIN_ERROR) Serial.println("Begin Failed");
      else if (error == OTA_CONNECT_ERROR) Serial.println("Connect Failed");
      else if (error == OTA_RECEIVE_ERROR) Serial.println("Receive Failed");
      else if (error == OTA_END_ERROR) Serial.println("End Failed");
    });

  ArduinoOTA.begin();

  Serial.println("Ready");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

// The first line of the code snippet below, for example, 
// indicates that when a server receives an HTTP request on the root (/) path, 
// it will call the handle_OnConnect() function. 
// It is important to note that the URL specified is a relative path.
// Similarly, we must specify two more URLs to handle the two states of the led.
// if the client requests a URL that isn’t specified with server.on() . 
// It should give a 404 error (Page Not Found) as a response. To accomplish this, 
// we use the server.onNotFound() method.
server.on("/", handle_OnConnect);
server.on("/ledOn", handle_ledOn);
server.on("/ledOff", handle_ledOff);
server.onNotFound(handle_NotFound);

//Now, to start the server, we call the server object’s begin() method.
server.begin();
Serial.println("HTTP server started");

}

void loop() {

  ArduinoOTA.handle();  

  // Actual incoming HTTP requests are handled in the loop function. 
  // For this, we use the server object’s handleClient() method. 
  // We also change the state of LEDs based on the request.
  // Now we must write the handle_OnConnect() function, which we previously attached to the root (/) URL with server.on. 
  // We begin this function by setting the status of both LEDs to LOW (initial state of LEDs) and printing it on the serial monitor.
  // We use the send method to respond to an HTTP request. 
  server.handleClient();
  if(ledState)
  {digitalWrite(led, HIGH);}
  else
  {digitalWrite(led, LOW);}

}

  // Although the method can be called with a number of different arguments, the simplest form requires the HTTP response code, 
  // the content type, and the content.
  // The first parameter we pass to the send method is the code 200 (one of the HTTP status codes), 
  // which corresponds to the OK response. Then we specify the content type as “text/html,” 
  // and finally we pass the SendHTML() custom function, which generates a dynamic HTML page with the LED status.
  void handle_OnConnect() {
  ledState = LOW;
  Serial.println("GPIO4 Status: OFF");
  server.send(200, "text/html", SendHTML(ledState)); 
  }

  // Similarly, we write three more functions to handle LED ON/OFF requests and the 404 Error page.
  void handle_ledOn() {
  ledState = HIGH;
  Serial.println("GPIO4 Status: ON");
  server.send(200, "text/html", SendHTML(true)); 
  }
  void handle_ledOff() {
  ledState = LOW;
  Serial.println("GPIO4 Status: OFF");
  server.send(200, "text/html", SendHTML(false)); 
  }
  void handle_NotFound(){
  server.send(404, "text/plain", "Not found");
  }

  // ! -- Displaying the HTML Web Page -- !
  // Whenever the ESP32 web server receives a request from a web client, the sendHTML() function generates a web page. 
  // It simply concatenates HTML code into a long string and returns to the server.send()
  // The function uses the status of the LEDs as a parameter to generate HTML content dynamically.
  // The first text you should always send is the <!DOCTYPE> declaration, which indicates that we’re sending HTML code.
  String SendHTML(uint8_t ledStat){

  String ptr = "<!DOCTYPE html> <html>\n";

  ptr +="<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0, user-scalable=no\">\n";
  ptr +="<title>LED Control</title>\n";
  ptr +="<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}\n";
  ptr +="body{margin-top: 50px;} h1 {color: #444444;margin: 50px auto 30px;} h3 {color: #444444;margin-bottom: 50px;}\n";
  ptr +="p {font-size: 14px;color: #888;margin-bottom: 10px;}\n";
  ptr +=".button {display: block;width: 80px;background-color: #3498db;border: none;color: white;padding: 13px 30px;text-decoration: none;font-size: 25px;margin: 0px auto 35px;cursor: pointer;border-radius: 4px;}\n";
  ptr +=".button-on {background-color: #3498db;}\n";
  ptr +=".button-on:active {background-color: #2980b9;}\n";
  ptr +=".button-off {background-color: #34495e;}\n";
  ptr +=".button-off:active {background-color: #2c3e50;}\n";
  ptr +="p {font-size: 14px;color: #888;margin-bottom: 10px;}\n";
  ptr +="</style>\n";
  ptr +="</head>\n";
  ptr +="<body>\n";

  ptr +="<h1>ESP32 Web Server</h1>\n";
    ptr +="<h3>Using Access Point(AP) Mode</h3>\n";
  if(ledStat)
   {ptr +="<p>LED Status: ON</p><a class=\"button button-off\" href=\"/ledOff\">OFF</a>\n";}
  else
   {ptr +="<p>LED Status: OFF</p><a class=\"button button-on\" href=\"/ledOn\">ON</a>\n";}

  ptr +="</body>\n";
  ptr +="</html>\n";
  return ptr;
  }
  ```
</details>

**Step two:**

<details><summary>Use this code to find the address of your temperature sensor and Real Time Clock Module</summary>
```c++
/*********
  Rui Santos & Sara Santos - Random Nerd Tutorials
  Complete project details at https://RandomNerdTutorials.com/esp32-datalogger-download-data-file/

  Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files.
  The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
*********/

#include <Arduino.h>
#include <WiFi.h>
#include <AsyncTCP.h>
#include <ESPAsyncWebServer.h>
#include "FS.h"
#include "SD.h"
#include "SPI.h"
#include <Adafruit_BME280.h>
#include <Adafruit_Sensor.h>
#include "time.h"
#include <WiFiUdp.h>

// Replace with your network credentials
const char* ssid = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";

// NTP server to request epoch time
const char* ntpServer = "pool.ntp.org";

// Variable to save current epoch time
unsigned long epochTime; 

// Variables to hold sensor readings
float temp;
float hum;
float pres;
String dataMessage;

// File name where readings will be saved
const char* dataPath = "/data.txt";

// Timer variables
unsigned long lastTime = 0;
unsigned long timerDelay = 30000;

// Create AsyncWebServer object on port 80
AsyncWebServer server(80);

// BME280 connect to ESP32 I2C (GPIO 21 = SDA, GPIO 22 = SCL)
Adafruit_BME280 bme;

// Init BME280
void initBME(){
  if (!bme.begin(0x76)) {
    Serial.println("Could not find a valid BME280 sensor, check wiring!");
    while (1);
  }
}

// Init microSD card
void initSDCard(){
  if(!SD.begin()){
    Serial.println("Card Mount Failed");
    return;
  }
  uint8_t cardType = SD.cardType();

  if(cardType == CARD_NONE){
    Serial.println("No SD card attached");
    return;
  }

  Serial.print("SD Card Type: ");
  if(cardType == CARD_MMC){
    Serial.println("MMC");
  } else if(cardType == CARD_SD){
    Serial.println("SDSC");
  } else if(cardType == CARD_SDHC){
    Serial.println("SDHC");
  } else {
    Serial.println("UNKNOWN");
  }
  uint64_t cardSize = SD.cardSize() / (1024 * 1024);
  Serial.printf("SD Card Size: %lluMB\n", cardSize);
}

// Write to the SD card
void writeFile(fs::FS &fs, const char * path, const char * message) {
  Serial.printf("Writing file: %s\n", path);

  File file = fs.open(path, FILE_WRITE);
  if(!file) {
    Serial.println("Failed to open file for writing");
    return;
  }
  if(file.print(message)) {
    Serial.println("File written");
  } else {
    Serial.println("Write failed");
  }
  file.close();
}

// Append data to the SD card
void appendFile(fs::FS &fs, const char * path, const char * message) {
  Serial.printf("Appending to file: %s\n", path);

  File file = fs.open(path, FILE_APPEND);
  if(!file) {
    Serial.println("Failed to open file for appending");
    return;
  }
  if(file.print(message)) {
    Serial.println("Message appended");
  } else {
    Serial.println("Append failed");
  }
  file.close();
}

// Delete file
void deleteFile(fs::FS &fs, const char * path){
  Serial.printf("Deleting file: %s\r\n", path);
  if(fs.remove(path)){
    Serial.println("- file deleted");
  } else {
    Serial.println("- delete failed");
  }
}

// Function that gets current epoch time
unsigned long getTime() {
  time_t now;
  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) {
   //Serial.println("Failed to obtain time");
    return(0);
  }
  time(&now);
  return now;
}

// Function that initializes wi-fi
void initWiFi() {
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi ..");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print('.');
    delay(1000);
  }
  Serial.println(WiFi.localIP());
}

void setup() {
  Serial.begin(115200);
  initWiFi();
  initBME();
  initSDCard();
  configTime(0, 0, ntpServer);

  // If the data.txt file doesn't exist
  // Create a file on the SD card and write the data labels
  File file = SD.open("/data.txt");
  if(!file) {
    Serial.println("File doesn't exist");
    Serial.println("Creating file...");
    writeFile(SD, "/data.txt", "Epoch Time, Temperature, Humidity, Pressure \r\n");
  }
  else {
    Serial.println("File already exists");  
  }
  file.close();

  // Handle the root URL
  server.on("/", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send(SD, "/index.html", "text/html");
  });

  // Handle the download button
  server.on("/download", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send(SD, "/data.txt", String(), true);
  });

  // Handle the View Data button
  server.on("/view-data", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send(SD, "/data.txt", "text/plain", false);
  });

  // Handle the delete button
  server.on("/delete", HTTP_GET, [](AsyncWebServerRequest *request){
    deleteFile(SD, dataPath);
    request->send(200, "text/plain", "data.txt was deleted.");
  });

  // Uncomment the following line if you need to serve more static files like CSS and javascript or favicon
  //server.serveStatic("/", SD, "/");

  server.begin();
}

void loop() {
  if ((millis() - lastTime) > timerDelay) {
    //Get epoch time
    epochTime = getTime();
    
    //Get sensor readings
    temp = bme.readTemperature();
    //temp = 1.8*bme.readTemperature() + 32;
    hum = bme.readHumidity();
    pres = bme.readPressure()/100.0F;

    //Concatenate all info separated by commas
    dataMessage = String(epochTime) + "," + String(temp) + "," + String(hum) + "," + String(pres)+ "\r\n";
    Serial.print("Saving data: ");
    Serial.println(dataMessage);

    //Append the data to file
    appendFile(SD, "/data.txt", dataMessage.c_str());

    lastTime = millis();
  }
}
```
</details>

**Step 3:**

<details><summary>Minimum working example to write to the SD-Card</summary>
  
```c++

/*
 * pin 1 - not used          |  Micro SD card     |
 * pin 2 - CS (SS)           |                   /
 * pin 3 - DI (MOSI)         |                  |__
 * pin 4 - VDD (3.3V)        |                    |
 * pin 5 - SCK (SCLK)        | 8 7 6 5 4 3 2 1   /
 * pin 6 - VSS (GND)         | ▄ ▄ ▄ ▄ ▄ ▄ ▄ ▄  /
 * pin 7 - DO (MISO)         | ▀ ▀ █ ▀ █ ▀ ▀ ▀ |
 * pin 8 - not used          |_________________|
 *                             ║ ║ ║ ║ ║ ║ ║ ║
 *                     ╔═══════╝ ║ ║ ║ ║ ║ ║ ╚═════════╗
 *                     ║         ║ ║ ║ ║ ║ ╚══════╗    ║
 *                     ║   ╔═════╝ ║ ║ ║ ╚═════╗  ║    ║
 * Connections for     ║   ║   ╔═══╩═║═║═══╗   ║  ║    ║
 * full-sized          ║   ║   ║   ╔═╝ ║   ║   ║  ║    ║
 * SD card             ║   ║   ║   ║   ║   ║   ║  ║    ║
 * Pin name         |  -  DO  VSS SCK VDD VSS DI CS    -  |
 * SD pin number    |  8   7   6   5   4   3   2   1   9 /
 *                  |                                  █/
 *                  |__▍___▊___█___█___█___█___█___█___/
 *
 * Note:  The SPI pins can be manually configured by using `SPI.begin(sck, miso, mosi, cs).`
 *        Alternatively, you can change the CS pin and use the other default settings by using `SD.begin(cs)`.
 *
 * +--------------+---------+-------+----------+----------+----------+----------+----------+
 * | SPI Pin Name | ESP8266 | ESP32 | ESP32‑S2 | ESP32‑S3 | ESP32‑C3 | ESP32‑C6 | ESP32‑H2 |
 * +==============+=========+=======+==========+==========+==========+==========+==========+
 * | CS (SS)      | GPIO15  | GPIO5 | GPIO34   | GPIO10   | GPIO7    | GPIO18   | GPIO0    |
 * +--------------+---------+-------+----------+----------+----------+----------+----------+
 * | DI (MOSI)    | GPIO13  | GPIO23| GPIO35   | GPIO11   | GPIO6    | GPIO19   | GPIO25   |
 * +--------------+---------+-------+----------+----------+----------+----------+----------+
 * | DO (MISO)    | GPIO12  | GPIO19| GPIO37   | GPIO13   | GPIO5    | GPIO20   | GPIO11   |
 * +--------------+---------+-------+----------+----------+----------+----------+----------+
 * | SCK (SCLK)   | GPIO14  | GPIO18| GPIO36   | GPIO12   | GPIO4    | GPIO21   | GPIO10   |
 * +--------------+---------+-------+----------+----------+----------+----------+----------+
 *
 * For more info see file README.md in this library or on URL:
 * https://github.com/espressif/arduino-esp32/tree/master/libraries/SD
 */

#include "FS.h"
#include "SD.h"
#include "SPI.h"

/*
Uncomment and set up if you want to use custom pins for the SPI communication
#define REASSIGN_PINS
int sck = -1;
int miso = -1;
int mosi = -1;
int cs = -1;
*/

void listDir(fs::FS &fs, const char *dirname, uint8_t levels) {
  Serial.printf("Listing directory: %s\n", dirname);

  File root = fs.open(dirname);
  if (!root) {
    Serial.println("Failed to open directory");
    return;
  }
  if (!root.isDirectory()) {
    Serial.println("Not a directory");
    return;
  }

  File file = root.openNextFile();
  while (file) {
    if (file.isDirectory()) {
      Serial.print("  DIR : ");
      Serial.println(file.name());
      if (levels) {
        listDir(fs, file.path(), levels - 1);
      }
    } else {
      Serial.print("  FILE: ");
      Serial.print(file.name());
      Serial.print("  SIZE: ");
      Serial.println(file.size());
    }
    file = root.openNextFile();
  }
}

void createDir(fs::FS &fs, const char *path) {
  Serial.printf("Creating Dir: %s\n", path);
  if (fs.mkdir(path)) {
    Serial.println("Dir created");
  } else {
    Serial.println("mkdir failed");
  }
}

void removeDir(fs::FS &fs, const char *path) {
  Serial.printf("Removing Dir: %s\n", path);
  if (fs.rmdir(path)) {
    Serial.println("Dir removed");
  } else {
    Serial.println("rmdir failed");
  }
}

void readFile(fs::FS &fs, const char *path) {
  Serial.printf("Reading file: %s\n", path);

  File file = fs.open(path);
  if (!file) {
    Serial.println("Failed to open file for reading");
    return;
  }

  Serial.print("Read from file: ");
  while (file.available()) {
    Serial.write(file.read());
  }
  file.close();
}

void writeFile(fs::FS &fs, const char *path, const char *message) {
  Serial.printf("Writing file: %s\n", path);

  File file = fs.open(path, FILE_WRITE);
  if (!file) {
    Serial.println("Failed to open file for writing");
    return;
  }
  if (file.print(message)) {
    Serial.println("File written");
  } else {
    Serial.println("Write failed");
  }
  file.close();
}

void appendFile(fs::FS &fs, const char *path, const char *message) {
  Serial.printf("Appending to file: %s\n", path);

  File file = fs.open(path, FILE_APPEND);
  if (!file) {
    Serial.println("Failed to open file for appending");
    return;
  }
  if (file.print(message)) {
    Serial.println("Message appended");
  } else {
    Serial.println("Append failed");
  }
  file.close();
}

void renameFile(fs::FS &fs, const char *path1, const char *path2) {
  Serial.printf("Renaming file %s to %s\n", path1, path2);
  if (fs.rename(path1, path2)) {
    Serial.println("File renamed");
  } else {
    Serial.println("Rename failed");
  }
}

void deleteFile(fs::FS &fs, const char *path) {
  Serial.printf("Deleting file: %s\n", path);
  if (fs.remove(path)) {
    Serial.println("File deleted");
  } else {
    Serial.println("Delete failed");
  }
}

void testFileIO(fs::FS &fs, const char *path) {
  File file = fs.open(path);
  static uint8_t buf[512];
  size_t len = 0;
  uint32_t start = millis();
  uint32_t end = start;
  if (file) {
    len = file.size();
    size_t flen = len;
    start = millis();
    while (len) {
      size_t toRead = len;
      if (toRead > 512) {
        toRead = 512;
      }
      file.read(buf, toRead);
      len -= toRead;
    }
    end = millis() - start;
    Serial.printf("%u bytes read for %lu ms\n", flen, end);
    file.close();
  } else {
    Serial.println("Failed to open file for reading");
  }

  file = fs.open(path, FILE_WRITE);
  if (!file) {
    Serial.println("Failed to open file for writing");
    return;
  }

  size_t i;
  start = millis();
  for (i = 0; i < 2048; i++) {
    file.write(buf, 512);
  }
  end = millis() - start;
  Serial.printf("%u bytes written for %lu ms\n", 2048 * 512, end);
  file.close();
}

void setup() {
  Serial.begin(115200);
  while (!Serial) {
    delay(10);
  }

#ifdef REASSIGN_PINS
  SPI.begin(sck, miso, mosi, cs);
  if (!SD.begin(cs)) {
#else
  if (!SD.begin()) {
#endif
    Serial.println("Card Mount Failed");
    return;
  }
  uint8_t cardType = SD.cardType();

  if (cardType == CARD_NONE) {
    Serial.println("No SD card attached");
    return;
  }

  Serial.print("SD Card Type: ");
  if (cardType == CARD_MMC) {
    Serial.println("MMC");
  } else if (cardType == CARD_SD) {
    Serial.println("SDSC");
  } else if (cardType == CARD_SDHC) {
    Serial.println("SDHC");
  } else {
    Serial.println("UNKNOWN");
  }

  uint64_t cardSize = SD.cardSize() / (1024 * 1024);
  Serial.printf("SD Card Size: %lluMB\n", cardSize);

  listDir(SD, "/", 0);
  createDir(SD, "/mydir");
  listDir(SD, "/", 0);
  removeDir(SD, "/mydir");
  listDir(SD, "/", 2);
  writeFile(SD, "/hello.txt", "Hello ");
  appendFile(SD, "/hello.txt", "World!\n");
  readFile(SD, "/hello.txt");
  deleteFile(SD, "/foo.txt");
  renameFile(SD, "/hello.txt", "/foo.txt");
  readFile(SD, "/foo.txt");
  testFileIO(SD, "/test.txt");
  Serial.printf("Total space: %lluMB\n", SD.totalBytes() / (1024 * 1024));
  Serial.printf("Used space: %lluMB\n", SD.usedBytes() / (1024 * 1024));
}

void loop() {}

```
</details>

**Step 4:**

<details><summary>Download a Data File via Web Server</summary>

Use Notepad to save a file called 'Index.html' to the SD-Card, and paste the below text in it

```c++
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ESP32 Datalogger</title>
    <style>
        html {
            font-family: Arial, Helvetica, sans-serif;
        }

        body {
            background-color: #f4f4f4;
            margin: 0;
            padding: 0;
        }

        .container {
            max-width: 800px;
            margin: 50px auto;
            text-align: center;
        }

        h1 {
            color: #333;
        }

        .button {
            display: inline-block;
            padding: 10px 20px;
            margin: 10px;
            font-size: 16px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition-duration: 0.4s;    }

        .button-data {
            background-color: #858585;
            color: #fff;
        }

        .button-delete {
            background-color: #780320;
            color: #fff;
        }

        .button:hover {
            background-color: #0056b3;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>ESP32 Datalogger - Manage Data</h1>
        <a href="view-data"><button class="button button-data">View Data</button></a>
        <a href="download"><button class="button button-data">Download Data</button></a>
        <a href="delete"><button class="button button-delete">Delete Data</button></a>
    </div>
</body>
</html>
```
**Challenge**: Modify this code to suit our "pins" and upload over wifi

```c++
/*********
  Rui Santos & Sara Santos - Random Nerd Tutorials
  Complete project details at https://RandomNerdTutorials.com/esp32-datalogger-download-data-file/

  Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files.
  The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
*********/

#include <Arduino.h>
#include <WiFi.h>
#include <AsyncTCP.h>
#include <ESPAsyncWebServer.h>
#include "FS.h"
#include "SD.h"
#include "SPI.h"
#include <Adafruit_BME280.h>
#include <Adafruit_Sensor.h>
#include "time.h"
#include <WiFiUdp.h>

// Replace with your network credentials
const char* ssid = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";

// NTP server to request epoch time
const char* ntpServer = "pool.ntp.org";

// Variable to save current epoch time
unsigned long epochTime; 

// Variables to hold sensor readings
float temp;
float hum;
float pres;
String dataMessage;

// File name where readings will be saved
const char* dataPath = "/data.txt";

// Timer variables
unsigned long lastTime = 0;
unsigned long timerDelay = 30000;

// Create AsyncWebServer object on port 80
AsyncWebServer server(80);

// BME280 connect to ESP32 I2C (GPIO 21 = SDA, GPIO 22 = SCL)
Adafruit_BME280 bme;

// Init BME280
void initBME(){
  if (!bme.begin(0x76)) {
    Serial.println("Could not find a valid BME280 sensor, check wiring!");
    while (1);
  }
}

// Init microSD card
void initSDCard(){
  if(!SD.begin()){
    Serial.println("Card Mount Failed");
    return;
  }
  uint8_t cardType = SD.cardType();

  if(cardType == CARD_NONE){
    Serial.println("No SD card attached");
    return;
  }

  Serial.print("SD Card Type: ");
  if(cardType == CARD_MMC){
    Serial.println("MMC");
  } else if(cardType == CARD_SD){
    Serial.println("SDSC");
  } else if(cardType == CARD_SDHC){
    Serial.println("SDHC");
  } else {
    Serial.println("UNKNOWN");
  }
  uint64_t cardSize = SD.cardSize() / (1024 * 1024);
  Serial.printf("SD Card Size: %lluMB\n", cardSize);
}

// Write to the SD card
void writeFile(fs::FS &fs, const char * path, const char * message) {
  Serial.printf("Writing file: %s\n", path);

  File file = fs.open(path, FILE_WRITE);
  if(!file) {
    Serial.println("Failed to open file for writing");
    return;
  }
  if(file.print(message)) {
    Serial.println("File written");
  } else {
    Serial.println("Write failed");
  }
  file.close();
}

// Append data to the SD card
void appendFile(fs::FS &fs, const char * path, const char * message) {
  Serial.printf("Appending to file: %s\n", path);

  File file = fs.open(path, FILE_APPEND);
  if(!file) {
    Serial.println("Failed to open file for appending");
    return;
  }
  if(file.print(message)) {
    Serial.println("Message appended");
  } else {
    Serial.println("Append failed");
  }
  file.close();
}

// Delete file
void deleteFile(fs::FS &fs, const char * path){
  Serial.printf("Deleting file: %s\r\n", path);
  if(fs.remove(path)){
    Serial.println("- file deleted");
  } else {
    Serial.println("- delete failed");
  }
}

// Function that gets current epoch time
unsigned long getTime() {
  time_t now;
  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) {
   //Serial.println("Failed to obtain time");
    return(0);
  }
  time(&now);
  return now;
}

// Function that initializes wi-fi
void initWiFi() {
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi ..");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print('.');
    delay(1000);
  }
  Serial.println(WiFi.localIP());
}

void setup() {
  Serial.begin(115200);
  initWiFi();
  initBME();
  initSDCard();
  configTime(0, 0, ntpServer);

  // If the data.txt file doesn't exist
  // Create a file on the SD card and write the data labels
  File file = SD.open("/data.txt");
  if(!file) {
    Serial.println("File doesn't exist");
    Serial.println("Creating file...");
    writeFile(SD, "/data.txt", "Epoch Time, Temperature, Humidity, Pressure \r\n");
  }
  else {
    Serial.println("File already exists");  
  }
  file.close();

  // Handle the root URL
  server.on("/", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send(SD, "/index.html", "text/html");
  });

  // Handle the download button
  server.on("/download", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send(SD, "/data.txt", String(), true);
  });

  // Handle the View Data button
  server.on("/view-data", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send(SD, "/data.txt", "text/plain", false);
  });

  // Handle the delete button
  server.on("/delete", HTTP_GET, [](AsyncWebServerRequest *request){
    deleteFile(SD, dataPath);
    request->send(200, "text/plain", "data.txt was deleted.");
  });

  // Uncomment the following line if you need to serve more static files like CSS and javascript or favicon
  //server.serveStatic("/", SD, "/");

  server.begin();
}

void loop() {
  if ((millis() - lastTime) > timerDelay) {
    //Get epoch time
    epochTime = getTime();
    
    //Get sensor readings
    temp = bme.readTemperature();
    //temp = 1.8*bme.readTemperature() + 32;
    hum = bme.readHumidity();
    pres = bme.readPressure()/100.0F;

    //Concatenate all info separated by commas
    dataMessage = String(epochTime) + "," + String(temp) + "," + String(hum) + "," + String(pres)+ "\r\n";
    Serial.print("Saving data: ");
    Serial.println(dataMessage);

    //Append the data to file
    appendFile(SD, "/data.txt", dataMessage.c_str());

    lastTime = millis();
  }
}

```
</details>

<details><summary> **Final Challenge:** </summary>
Follow along the walkthrough in the below link to set up the data logger which utilises deep sleep mode. Replace the use of the DS18B20 temperature sensor with our BME280 Sensor. Instead of using the "NTP" time programme, use our DS3231 with the I2C protocol. Both the BME280 and DS3231 are able to utilise I2C.
  (https://randomnerdtutorials.com/esp32-data-logging-temperature-to-microsd-card/)
  
</details>

