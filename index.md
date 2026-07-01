# BlueStamp Fingerprint ID Safe with Keypad
My project is a safe that will require a fingerprint and password match to unlock.

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

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 

# First Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="1053" height="592" src="https://www.youtube.com/embed/g-cnZu8ZC8U" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For my first milestone, I wired up a keypad, fingerprint sensor, and lcd to an Arduino and coded it. The system starts off by asking you to enter the password. If you get the password wrong, you lose an attempt and have two more chances to get it right. If you run out of attempts, you are locked out until the safe turns off. If you get it correct, you are then prompted to scan your finger. If the system recognizes your fingerprint, then the safe unlocks. If it doesn't, you are asked to rescan your finger, and if you still get it wrong, you are locked out of the safe. The hardest part of creating this was probably getting my code to work and fixing any incorrect wirings. For my second milestone, I will likely wire up a servo motor that will physically open the safe and a button to save battery.

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code 

```c++
#include <LiquidCrystal_I2C.h>
#include <Wire.h>
#include "Adafruit_Keypad.h"
#include <Adafruit_Fingerprint.h>
#include <Servo.h>

#if (defined(__AVR__) || defined(ESP8266)) && !defined(__AVR_ATmega2560__)
SoftwareSerial mySerial(2, 3);
#else
#define mySerial Serial1
#endif

#define KEYPAD_PID3845
#define R2    4
#define R3    5
#define C3    6
#define R4    7
#define C1    8
#define R1    9
#define C2    10

#include "keypad_config.h"

Servo myservo;
Adafruit_Keypad customKeypad = Adafruit_Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);
LiquidCrystal_I2C lcd (0x27, 16, 2);
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);

String pwd = "";
bool start_pressed = false;
int attempts_left = 3;
int servo_pos = 0;

void setup() {
  Serial.begin(9600);
  customKeypad.begin();
  lcd.init();
  lcd.backlight();
  finger.begin(57600);
  myservo.attach(11);
  reset();
}

void loop() {
  myservo.write(0);
  customKeypad.tick();
  // read keys
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
        lcd.setCursor(0, 0);
        lcd.print("Enter Password:");
      } 
      else if (start_pressed && cur != '#') {
        pwd += cur;
        lcd.setCursor(pwd.length() - 1, 1);
        lcd.print("*");

        // Check if full password length is reached
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
    Serial.println("Waiting for valid finger...");
    finger.getTemplateCount();
    Serial.print("Sensor contains ");
    Serial.print(finger.templateCount);
    Serial.println(" templates");
    int c = -1;
    while (c == -1) {
      c = getFingerprintID();
      Serial.println(c);
      delay(500);
    }
    if (c >= 75) {
      finger_accepted();
    } else {
      lcd.clear();
      lcd.print("No Match Found");
      lcd.setCursor(0, 1);
      lcd.print("Rescan Finger");
      delay(2000);
      c = -1;
      while (c == -1) {
        c = getFingerprintID();
        Serial.println(c);
        delay(500);
      }
      if (c >= 75) {
        finger_accepted();
      }
      else {
        permanently_locked();
      }
    }
    delay(3000);
    reset();
  } 
  else {
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
  lcd.setCursor(0, 0);
  lcd.print("Press * to Start");
}

int getFingerprintID() {
  uint8_t p = finger.getImage();
  switch (p) {
  case FINGERPRINT_OK:
    Serial.println("Image taken");
    break;
  case FINGERPRINT_NOFINGER:
    Serial.println("No finger detected");
    return -1;
  case FINGERPRINT_PACKETRECIEVEERR:
    Serial.println("Communication error");
    return -1;
  case FINGERPRINT_IMAGEFAIL:
    Serial.println("Imaging error");
    return -1;
  default:
    Serial.println("Unknown error");
    return -1;
  }

  // OK success!

  p = finger.image2Tz();
  switch (p) {
  case FINGERPRINT_OK:
    Serial.println("Image converted");
    break;
  case FINGERPRINT_IMAGEMESS:
    Serial.println("Image too messy");
    return -1;
  case FINGERPRINT_PACKETRECIEVEERR:
    Serial.println("Communication error");
    return -1;
  case FINGERPRINT_FEATUREFAIL:
    Serial.println("Could not find fingerprint features");
    return -1;
  case FINGERPRINT_INVALIDIMAGE:
    Serial.println("Could not find fingerprint features");
    return -1;
  default:
    Serial.println("Unknown error");
    return -1;
  }

  // OK converted!
  p = finger.fingerSearch();
  if (p == FINGERPRINT_OK) {
    Serial.println("Found a print match!");
  } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
    Serial.println("Communication error");
    return -1;
  } else if (p == FINGERPRINT_NOTFOUND) {
    Serial.println("Did not find a match");
    return p;
  } else {
    Serial.println("Unknown error");
    return -1;
  }

  // found a match!
  Serial.print("Found ID #");
  Serial.print(finger.fingerID);
  Serial.print(" with confidence of ");
  Serial.println(finger.confidence);

  return (int)(finger.confidence);
}

void permanently_locked() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("SYSTEM LOCKED");
  while(true); // Locked forever
}

void finger_accepted() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Finger Accepted");
  lcd.setCursor(0, 1);
  lcd.print("Lockbox Opening");
  delay(15);
  for (servo_pos = 0; servo_pos <= 90; servo_pos++) {
    myservo.write(servo_pos);
    delay(15);
  }
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Lockbox Open");
  lcd.setCursor(0, 1);
  lcd.print("Press # to Close");
  while (true) {
    customKeypad.tick();
    if (customKeypad.available()) {
      keypadEvent e = customKeypad.read();
      if ((e.bit.EVENT == KEY_JUST_PRESSED) && ((char)e.bit.KEY == '#')) {
        for (servo_pos = 90; servo_pos >= 0; servo_pos--) {
          myservo.write(servo_pos);
          delay(15);
        }
        lcd.clear();
        lcd.setCursor(0, 0);
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
| Liquid Crystal Display Screen | Used to display messages outputted from the code | $12.99 | <a href="https://www.amazon.com/Hosyond-Display-Module-Arduino-Raspberry/dp/B0BWTFN9WF/?th=1"> Link </a> |

# Starter Project

<iframe width="1053" height="592" src="https://www.youtube.com/embed/x7NOztviujc" title="William X. Starter Project" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For my starter project, I decided to choose the Retro Arcade Console. It features four different game modes that you can play. The hardest part of completing this project was probably the soldering part because some of the holes were really small, making it easy to short-circuit.
