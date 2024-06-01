# Color Sorter IoT Project

## Project Overview
This project involves creating an IoT-based color sorter that can detect different colors and sort objects accordingly using a robotic arm. The system is equipped with a color sensor, servo motors for picking and dropping objects, and is integrated with ThingSpeak for data logging and Telegram for real-time notifications.

## Components and Libraries
The project uses the following components:
- ESP8266 WiFi Module
- Color Sensor (TCS3200)
- Servo Motors
- ThingSpeak API for data logging
- Telegram API for notifications
- Web server for displaying current detected color

Required libraries:
- `Servo.h`
- `ESP8266WiFi.h`
- `ThingSpeak.h`
- `ESP8266HTTPClient.h`
- `ESP8266WebServer.h`

To install these libraries, use the Arduino Library Manager or the following commands:
```cpp
#include <Servo.h>
#include <ESP8266WiFi.h>
#include <ThingSpeak.h>
#include <ESP8266HTTPClient.h>
#include <ESP8266WebServer.h>
```

## Setup and Configuration

### Pin Configuration
- `s0`, `s1`, `s2`, `s3`, `out`: Pins connected to the TCS3200 color sensor.
- `D2`: Pin for the pick servo motor.
- `D3`: Pin for the drop servo motor.

### WiFi and ThingSpeak Configuration
```cpp
const char *ssid = "rj_chets"; // Enter your WiFi Name
const char *pass = "rj_chets"; // Enter your WiFi Password
const char *myWriteAPIKey = "9B6ILVOYMUSVOADA";
unsigned long myChannelNumber = 691885;
```

### Telegram Bot Configuration
```cpp
const char *botToken = "YOUR_BOT_TOKEN"; // Replace with your Telegram bot token
const char *chatID = "YOUR_CHAT_ID"; // Replace with your Telegram chat ID
```

## Code Overview

### Setup Function
The `setup` function initializes the serial communication, configures the pins, connects to WiFi, sets up the servos, initializes ThingSpeak, and starts the web server.
```cpp
void setup() {
  Serial.begin(9600);
  pinMode(s0, OUTPUT);
  pinMode(s1, OUTPUT);
  pinMode(s2, OUTPUT);
  pinMode(s3, OUTPUT);
  pinMode(out, INPUT);
  digitalWrite(s0, HIGH);
  digitalWrite(s1, HIGH);
  pickServo.attach(D2);
  dropServo.attach(D3);
  pickServo.write(CLOSE_ANGLE);
  dropServo.write(73);
  ThingSpeak.begin(http);

  Serial.println("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(550);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");

  server.on("/", HTTP_GET, handleRoot);
  server.on("/colors", HTTP_GET, handleColors);

  server.begin();
}
```

### Loop Function
The `loop` function handles the color detection, sends notifications via Telegram, logs data to ThingSpeak, and updates the web server.
```cpp
void loop() {
  server.handleClient(); // Handle web server requests

  digitalWrite(s2, LOW);
  digitalWrite(s3, LOW);
  red = pulseIn(out, digitalRead(out) == HIGH ? LOW : HIGH);
  digitalWrite(s3, HIGH);
  blue = pulseIn(out, digitalRead(out) == HIGH ? LOW : HIGH);
  digitalWrite(s2, HIGH);
  green = pulseIn(out, digitalRead(out) == HIGH ? LOW : HIGH);

  Serial.print("R Intensity:");
  Serial.print(red, DEC);
  Serial.print(" G Intensity: ");
  Serial.print(green, DEC);
  Serial.print(" B Intensity : ");
  Serial.print(blue, DEC);

  unsigned long currentMillis = millis();
  if (currentMillis - lastMessageTime >= interval) {
    lastMessageTime = currentMillis; // Update the last message time

    if (red < 39 && red > 29 && green < 93 && green > 83 && blue < 78 && blue > 69) {
      sendTelegramMessage("Red color detected.");
      dropServo.write(73);
      delay(700);
      redcolor++;
      Serial.print("Red");
      open1();
      delay(200);
      close1();
      ThingSpeak.writeField(myChannelNumber, 1, redcolor, myWriteAPIKey);
    } else if (green < 75 && green > 65 && blue < 68 && blue > 60) {
      sendTelegramMessage("Orange color detected.");
      dropServo.write(107);
      delay(700);
      orangecolor++;
      Serial.print("Orange");
      open1();
      delay(200);
      close1();
      ThingSpeak.writeField(myChannelNumber, 2, orangecolor, myWriteAPIKey);
    } else if (red < 46 && red > 36 && green < 46 and green > 37) {
      sendTelegramMessage("Green color detected.");
    } else if (red < 34 && red > 25 and green < 37 and green > 28 and blue < 53 and blue > 43) {
      sendTelegramMessage("Yellow color detected.");
      dropServo.write(132);
      delay(700);
      greencolor++;
      Serial.print("Green");
      open1();
      delay(200);
      close1();
      ThingSpeak.writeField(myChannelNumber, 3, greencolor, myWriteAPIKey);
    } else {
      sendTelegramMessage("No color detected or color out of range.");
    }
  }

  Serial.println();
}
```

### Helper Functions
- `open1()`: Opens the pick servo.
- `close1()`: Closes the pick servo.
- `sendTelegramMessage(String message)`: Sends a message via Telegram.
- `URLEncode(String msg)`: Encodes a URL string.
- `handleRoot()`: Handles the root web server request.
- `handleColors()`: Handles the colors web server request.

## Web Server
The project includes a basic web server to display the currently detected color. The web server is configured in the `setup` function and the handlers are defined to return HTML content.

