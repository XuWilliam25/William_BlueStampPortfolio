# BlueStamp Fingerprint ID Safe with Keypad
For my intensive project, I decided to build a lockbox to store my valuables. It features a password keypad, fingerprint sensor, and mobile app. There are two ways to unlock the safe: either correctly enter the password and scan your fingerprint on the lockbox console, or use the face ID feature on your phone to directly unlock the safe.

You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:
```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| William X | Irvington High School | Electrical Engineering | Incoming Sophomore

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**



For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**



For my second milestone, I wired up a servo motor to my Arduino and organized the wires a bit. However, as soon as I transferred everything over to the Arduino R4, my fingerprint sensor stopped working, which I spent two weeks trying to fix. After that, I synced the Arduino R4 with Blynk so that I could open the lockbox from my phone (in case something goes wrong with the fingerprint sensor). How the Blynk system works is, there's a button in the app that's connected to a datastream, essentially a variable, on the website. When you click the button once, it changes the value of the datastream from 0 to 1 and sends a message to the Arduino, allowing the servo to rotate. When you click it again, the value changes back to 0, prompting it to rotate back to its initial position. For my final milestone, I plan to fix any bugs that still exist, add a switch to turn the box on and off without being connected to my computer, and drill everything into the actual box.

# First Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/g-cnZu8ZC8U" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For my first milestone, I wired up a keypad, fingerprint sensor, and lcd to an Arduino and coded it. The system starts off by asking you to enter the password. If you get the password wrong, you lose an attempt and have two more chances to get it right. If you run out of attempts, you are locked out until the safe turns off. If you get it correct, you are then prompted to scan your finger. If the system recognizes your fingerprint, then the safe unlocks. If it doesn't, you are asked to rescan your finger, and if you still get it wrong, you are locked out of the safe. The hardest part of creating this was probably getting my code to work and fixing any incorrect wirings. For my second milestone, I will likely wire up a servo motor that will physically open the safe and a button to save battery.

# Schematics 
![Keypad Wiring](keypad_wirings.jpg)
<br>
**Figure 1 - Keypad Wiring**
<br>
<br>
<br>
<br>
![Button Wiring](button_wiring.jpg)
<br>
**Figure 2 - Button Wiring**
<br>
<br>
<br>
<br>
![LCD Wiring](lcd_wiring.jpg)
<br>
**Figure 3 - LCD Wiring**
<br>
<br>
<br>
<br>
![Servo Motor Wiring](servo_wiring.jpg)
<br>
**Figure 4 - Servo Motor Wiring**

# Code 

```c++
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include "Adafruit_Keypad.h"
#include <Adafruit_Fingerprint.h>
#include <Servo.h>
#define KEYPAD_PID3845
#define R2 4
#define R3 5
#define C3 6
#define R4 7
#define C1 8
#define R1 9
#define C2 10
#include "keypad_config.h"
Servo myservo;
Adafruit_Keypad customKeypad = Adafruit_Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);
LiquidCrystal_I2C lcd(0x27, 16, 2);
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&Serial1);
String pwd = "";
bool start_pressed = false;
int attempts_left = 3;
int servo_pos = 0;
void setup() {
  Serial.begin(9600);
  delay(500);
  Serial.println("---SYSTEM STARTING---");
  Wire.begin();
  Wire.setClock(100000);
  delay(100);
  lcd.init();
  lcd.backlight();
  lcd.clear();
  lcd.print("Starting Up...");
  Serial.println("LCD initialized");
  customKeypad.begin();
  Serial.println("Keypad initialized");
  Serial1.begin(57600);
  while (!Serial1);
  finger.begin(57600);
  delay(3000); 
  
  while (Serial1.available() > 0) {
    Serial1.read();
  }
  
  

  delay(200);
  lcd.clear();
  if (finger.verifyPassword()) {
    Serial.println("Fingerprint sensor initialized");
  } else {
    Serial1.end();
    delay(500);
    Serial1.begin(57600);
    delay(500);
    while (Serial1.available() > 0) {
      Serial1.read();
    }
    if (finger.verifyPassword()) {
      Serial.println("Fingerprint sensor initialized");
    } else {
      Serial.println("ERROR: Fingerprint sensor not found");
      delay(2000);
    }
  }
  delay(1000);
  reset();
}
void loop() {
  customKeypad.tick();
  while (customKeypad.available()) {
    keypadEvent e = customKeypad.read();
    if (e.bit.EVENT == KEY_JUST_PRESSED) {
      char cur = (char)e.bit.KEY;
      Serial.print("Key pressed: ");
      Serial.println(cur);
      if (cur == '*') {
        start_pressed = true;
        pwd = "";
        lcd.clear();
        lcd.print("Enter Password:");
      } else if (start_pressed && cur != '#') {
        pwd += cur;
        lcd.setCursor(pwd.length() - 1, 1);
        lcd.print("*");
        if (pwd.length() == 4) {
          checkPassword();
        }
      }
    }
  }
  delay(10);
}
void checkPassword() {
  lcd.clear();
  lcd.setCursor(0, 0);
  if (pwd == "1234") {
    attempts_left = 3;
    lcd.print("Accepted");
    lcd.setCursor(0, 1);
    lcd.print("Scan Fingerprint");
    Serial.println("Password accepted, scan fingerprint now");
    int template_status = finger.getTemplateCount();
    if (template_status == FINGERPRINT_OK) {
      Serial.print("Sensor connection verified. Templates found: ");
      Serial.println(finger.templateCount);
    } else {
      Serial.println("ERROR: Sensor failed to return template count.");
    }
    finger.LEDcontrol(FINGERPRINT_LED_FLASHING, 25, FINGERPRINT_LED_PURPLE, 0);
    delay(100);
    int c = -1;
    unsigned long scanStartTime = millis();
    while (c == -1) {
      c = getFingerprintID();
      Serial.print("Scan status: ");
      Serial.println(c);
      delay(500);
      if (millis() - scanStartTime > 20000) {
        Serial.println("Fingerprint scan timed out.");
        finger.LEDcontrol(FINGERPRINT_LED_OFF, 0, 0, 0);
        lcd.clear();
        lcd.print("Timed Out");
        delay(2000);
        break;
      }
    }
    if (c >= 75) {
      finger_accepted();
    } else if (c != -1) {
      lcd.clear();
      lcd.print("No Match Found");
      lcd.setCursor(0, 1);
      lcd.print("Rescan Finger");
      delay(2000);
      c = -1;
      while (c == -1) {
        c = getFingerprintID();
        Serial.print("Scan status: ");
        Serial.println(c);
        delay(500);
      }
      if (c >= 75) {
        finger_accepted();
      } else {
        permanently_locked();
      }
    }
    delay(2000);
    reset();
  } else {
    attempts_left--;
    lcd.print("Access Denied");
    lcd.setCursor(0, 1);
    lcd.print("Attempts Left: ");
    lcd.print(attempts_left);
    delay(2000);
    if (attempts_left <= 0) {
      permanently_locked();
    } else {
      reset();
    }
  }
}
void reset() {
  pwd = "";
  start_pressed = false;
  lcd.clear();
  lcd.print("Press * to Start");
}
int getFingerprintID() {
  delay(10);
  while(Serial1.available() > 0) { 
    Serial1.read();
  }
  uint8_t p = finger.getImage();
  if (p == FINGERPRINT_NOFINGER) {
    return -1;
  }
  if (p == FINGERPRINT_PACKETRECIEVEERR) {
    return -1;
  }
  if (p != FINGERPRINT_OK) {
    return -1;
  }
  Serial.println("Image successfully taken!");
  delay(20);
  p = finger.image2Tz();
  if (p != FINGERPRINT_OK) {
    Serial.println("Failed to convert image features.");
    return -1;
  }
  delay(20);
  p = finger.fingerSearch();
  if (p == FINGERPRINT_OK) {
    Serial.print("Match Found! ID #");
    Serial.print(finger.fingerID);
    Serial.print(" with confidence score of ");
    Serial.println(finger.confidence);
    return (int)(finger.confidence);
  } else if (p == FINGERPRINT_NOTFOUND) {
    Serial.println("Fingerprint does not match stored database.");
    return 0;
  }
  return -1;
}
void permanently_locked() {
  lcd.clear();
  lcd.print("SYSTEM LOCKED");
  while(true);
}
void finger_accepted() {
  finger.LEDcontrol(FINGERPRINT_LED_OFF, 0, 0, 0);
  lcd.clear();
  lcd.print("Finger Accepted");
  lcd.setCursor(0, 1);
  lcd.print("Lockbox Opening");
  delay(15);
  myservo.attach(11);
  myservo.write(0);
  Serial.println("Servo initialized");
  delay(15);
  for (servo_pos = 0; servo_pos <= 90; servo_pos++) {
    myservo.write(servo_pos);
    delay(15);
  }
  lcd.clear();
  lcd.print("Lockbox Open");
  lcd.setCursor(0, 1);
  lcd.print("Press # to Close");
  while (true) {
    customKeypad.tick();
    if (customKeypad.available()) {
      keypadEvent e = customKeypad.read();
      if ((e.bit.EVENT == KEY_JUST_PRESSED) && ((char)e.bit.KEY == '#')) {
        Serial.println("Closing Servo");
        for (servo_pos = 90; servo_pos >= 0; servo_pos--) {
          myservo.write(servo_pos);
          delay(15);
        }
        myservo.detach(); 
        lcd.clear();
        lcd.print("Lockbox Closed");
        delay(750);
        break;
      }
    }
  }
}
```

# Bill of Materials 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Arduino Uno | Used to relay code to the entire system | $20.70 | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Breadboard | Used to wire devices to the Arduino | $8.99 | <a href="https://www.amazon.com/EL-CP-003-Breadboard-Solderless-Distribution-Connecting/dp/B01EV6LJ7G/"> Link </a> |
| Keypad | Used to input the password | $6.50 | <a href="https://www.adafruit.com/product/3845?srsltid=AfmBOop0mFDCKUcpbJfNziJJQCqdIGndAFKcya28DPK7Vt_bTC7mqKFMPDg"> Link </a> |
| Fingerprint Sensor | Used to scan user's fingerprint | €36,19 | <a href="https://www.amazon.com.be/-/en/Fingerprint-Identification-Capacitive-Recognition-Assistance/dp/B08GKY4RK1?language=en_GB"> Link </a> |
| Liquid Crystal Display Screen | Used to display messages outputted from the code | $12.99 | <a href="https://www.amazon.com/Hosyond-Display-Module-Arduino-Raspberry/dp/B0BWTFN9WF/"> Link </a> |
| Servo Motor | Used to open the lockbox | $13.98 | <a href="https://www.amazon.com/Deegoo-FPV-Servo-MG995-Metal-Gear/dp/B07NQJ1VZ2/"> Link </a> |
| Switch | Used to turn the lockbox on and off | $6.39 | <a href="https://www.amazon.com/DaierTek-Listed-Switches-Automotive-KCD1-5Pack/dp/B07S1MV462/"> Link </a> |
| Battery Holder | Used to hold AA batteries | $3.95 | <a href="https://www.amazon.com/Battery-Spring-Holder-Plastic-Storage/dp/B072FBL5HG/"> Link </a> |
| AA Batteries | Used to power the lockbox | $6.49 | <a href="https://www.amazon.com/Amazon-Basics-Batteries-Leak-Free-Household/dp/B00O869KJE/"> Link </a> |

# Starter Project

<iframe width="560" height="315" src="https://www.youtube.com/embed/x7NOztviujc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For my starter project, I decided to choose the Retro Arcade Console. It features four different game modes that you can play: Tetris, Snake, Racing, and Slot. The hardest part of completing this project for me was soldering because some of the holes were really small, making it easy to create an accidental short circuit.
