#include <Wire.h>
#include <LiquidCrystal_I2C.h>

const int k1 = 12; // krancowka 1
const int k2 = 11; // krancowka 2
const int buttonPin2 = 9;  // lewo up
const int buttonPin3 = 8;  // lewo down
const int buttonPin4 = 2;  // lewo start
const int buttonPin5 = 3;  // prawo up
const int buttonPin6 = 4;  // prawo down
const int buttonPin7 = 10; // stop

LiquidCrystal_I2C lcd(0x27, 16, 2);

int time1 = 10; // Czas gracza 1 w minutach
int time2 = 10; // Czas gracza 2 w minutach
int seconds1 = 0;
int seconds2 = 0;
bool running = false;
int currentPlayer = 0; // 0 gracz 1, 1 gracz 2
int startPlayer = 0;   // 0  zaczyna lewy, 1  zaczyna prawy
unsigned long lastMillis = 0;

void setup() {
    Serial.begin(9600);
    pinMode(k1, INPUT_PULLUP);
    pinMode(k2, INPUT_PULLUP);
    pinMode(buttonPin2, INPUT_PULLUP);
    pinMode(buttonPin3, INPUT_PULLUP);
    pinMode(buttonPin4, INPUT_PULLUP);
    pinMode(buttonPin5, INPUT_PULLUP);
    pinMode(buttonPin6, INPUT_PULLUP);
    pinMode(buttonPin7, INPUT_PULLUP);
    
    lcd.init();
    lcd.backlight();
    lcd.setCursor(2, 0);
    lcd.print("Autor: Piotr");
    lcd.setCursor(2, 1);
    lcd.print("Ciechanowski");
    delay(3000);
    lcd.clear();
    updateDisplay();
}

void loop() {
    if (!running) {
        // Regulacja czasu dla gracza 1
        if (digitalRead(buttonPin2) == LOW) {
            if (time1 < 60) time1++;
            seconds1 = 0;  // Ustawiamy sekundy na 0
            updateDisplay();
            delay(200);
        }
        if (digitalRead(buttonPin3) == LOW) {
            if (time1 > 0) time1--;
            seconds1 = 0;  // Ustawiamy sekundy na0
            updateDisplay();
            delay(200);
        }
        
        // Regulacja czasu dla gracza 2
        if (digitalRead(buttonPin5) == LOW) {
            if (time2 < 60) time2++;
            seconds2 = 0;  // Ustawiamy sekundyna 0
            updateDisplay();
            delay(200);
        }
        if (digitalRead(buttonPin6) == LOW) {
            if (time2 > 0) time2--;
            seconds2 = 0;  // Ustawiamysekundy na 0
            updateDisplay();
            delay(200);
        }
        
        //Wybór gracza startowego za pomocą k1 i k2
        if (digitalRead(k1) == LOW) {
            startPlayer = 0;
            updateDisplay();
            delay(200);
        }
        if (digitalRead(k2) == LOW) {
            startPlayer = 1;
            updateDisplay();
            delay(200);
        }
    }
    
    if (digitalRead(buttonPin4) == LOW) {
        running = true;
        currentPlayer = startPlayer;
        lastMillis = millis();
        Serial.println("Zegar start");
        delay(200);
    }
    if (digitalRead(buttonPin7) == LOW) {
        running = false;
        Serial.println("Zegar stop");
        delay(200);
    }
    if (digitalRead(k1) == LOW && running) {
        currentPlayer = 1;
        lastMillis = millis();
        Serial.println("Tura gracza 2");
        delay(200);
    }
    if (digitalRead(k2) == LOW && running) {
        currentPlayer = 0;
        lastMillis = millis();
        Serial.println("Tura gracza 1");
        delay(200);
    }
    if (running) {
        updateTime();
    }
}

void updateTime() {
    if (millis() - lastMillis >= 1000) {
        lastMillis = millis();
        if (currentPlayer == 0) {
            if (seconds1 > 0) {
                seconds1--;
            } else if (time1 > 0) {
                time1--;
                seconds1 = 59;
            }
        } else {
            if (seconds2 > 0) {
                seconds2--;
            } else if (time2 > 0) {
                time2--;
                seconds2 = 59;
            }
        }
        updateDisplay();
    }
}

void updateDisplay() {
    lcd.clear();
    lcd.setCursor(0, 0);
    if (!running) {
        lcd.print("Start: ");
        lcd.print(startPlayer == 0 ? "Left" : "Right");
        lcd.print("    ");
    } else {
        lcd.print("Left       Right");
    }
    
    lcd.setCursor(0, 1);
    if (time1 == 0 && seconds1 == 0) {
        lcd.print("END T");
    } else {
        lcd.print(time1);
        lcd.print(":");
        if (seconds1 < 10) lcd.print("0");
        lcd.print(seconds1);
    }
    
    lcd.setCursor(11, 1);
    if (time2 == 0 && seconds2 == 0) {
        lcd.print("END T");
    } else {
        lcd.print(time2);
        lcd.print(":");
        if (seconds2 < 10) lcd.print("0");
        lcd.print(seconds2);
    }
}