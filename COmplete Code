#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <TimeLib.h>
#include <DS1307RTC.h>

// Define I2C address and LCD dimensions
#define I2C_ADDR    0x27
#define LCD_COLS    16
#define LCD_ROWS    2

// Define button pins
#define BUTTON_PIN          2
#define BUTTON_SET_HOUR     3
#define BUTTON_SET_MINUTE   4
#define ALARM_OUTPUT_PIN    13
#define MOTOR_PIN1          10 // Motor driver pin 1
#define MOTOR_PIN2          11 // Motor driver pin 2

// Initialize LCD object
LiquidCrystal_I2C lcd(I2C_ADDR, LCD_COLS, LCD_ROWS);

// Global variables
bool alarmMode = false;
int alarmHour = 0;
int alarmMinute = 0;
bool hourSetMode = false;
bool minuteSetMode = false;
bool alarmTriggered = false;

void setup() {
  // Initialize LCD
  lcd.init();
  lcd.backlight();

  // Initialize button pins
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  pinMode(BUTTON_SET_HOUR, INPUT_PULLUP);
  pinMode(BUTTON_SET_MINUTE, INPUT_PULLUP);
  pinMode(ALARM_OUTPUT_PIN, OUTPUT);
  pinMode(MOTOR_PIN1, OUTPUT);
  pinMode(MOTOR_PIN2, OUTPUT);

  // Initialize serial communication
  Serial.begin(9600);
  
  // Declare tmElements_t variable to hold time elements
  tmElements_t tm;
  
  // Set the current time
  if (RTC.read(tm)) {
    setTime(tm.Hour, tm.Minute, tm.Second, tm.Day, tm.Month, tmYearToCalendar(tm.Year));
  } else {
    setTime(12, 0, 0, 1, 1, 2022); // Set the time to 12:00:00 on 1 Jan 2022 as default
  }
}

void loop() {
  // Check if the button is pressed to toggle alarm mode
  if (digitalRead(BUTTON_PIN) == LOW) {
    static unsigned long lastPressTime = 0;
    unsigned long currentMillis = millis();
    if (currentMillis - lastPressTime < 500) {
      // Double press detected, set alarm to current time
      alarmHour = hour();
      alarmMinute = minute();
    }
    lastPressTime = currentMillis;
    
    // Toggle alarm mode
    alarmMode = !alarmMode;
    
    // When entering alarm set mode, set the alarm to current time from RTC module
    if (alarmMode) {
      tmElements_t tm;
      if (RTC.read(tm)) {
        alarmHour = tm.Hour;
        alarmMinute = tm.Minute;
      }
    }
  }

  // Set alarm hour using button
  if (alarmMode && digitalRead(BUTTON_SET_HOUR) == LOW) {
    hourSetMode = true;
    minuteSetMode = false;
    delay(200); // Button debounce
  }

  // Set alarm minute using button
  if (alarmMode && digitalRead(BUTTON_SET_MINUTE) == LOW) {
    minuteSetMode = true;
    hourSetMode = false;
    delay(200); // Button debounce
  }

  // Increment alarm hour
  if (hourSetMode && digitalRead(BUTTON_SET_HOUR) == LOW) {
    alarmHour = (alarmHour + 1) % 24;
    delay(200); // Button debounce
  }

  // Increment alarm minute
  if (minuteSetMode && digitalRead(BUTTON_SET_MINUTE) == LOW) {
    alarmMinute = (alarmMinute + 1) % 60;
    delay(200); // Button debounce
  }
  
  // Get current time
  tmElements_t tm;
  if (RTC.read(tm)) {
    // Format time and date strings
    char timeStr[9];
    sprintf(timeStr, "%02d:%02d:%02d", tm.Hour, tm.Minute, tm.Second);
    
    char dateStr[12];
    sprintf(dateStr, "%02d-%02d-%04d", tm.Day, tm.Month, tmYearToCalendar(tm.Year));
    
    // Display time and date on LCD
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Time: ");
    lcd.print(timeStr);
    
    lcd.setCursor(0, 1);
    lcd.print("Date: ");
    lcd.print(dateStr);
    
    // Print time and date to serial monitor
    Serial.print("Time: ");
    Serial.print(timeStr);
    Serial.print("  Date: ");
    Serial.println(dateStr);
    
    // Switch to alarm set mode if alarmMode is true
    if (alarmMode) {
      lcd.setCursor(0, 2);
      lcd.print("Alarm Set Mode");
      lcd.setCursor(0, 3);
      lcd.print("Set Alarm: ");
      lcd.print(alarmHour < 10 ? "0" : "");
      lcd.print(alarmHour);
      lcd.print(":");
      lcd.print(alarmMinute < 10 ? "0" : "");
      lcd.print(alarmMinute);
    }
  } else {
    // Print error message if RTC communication fails
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("RTC Communication");
    lcd.setCursor(0, 1);
    lcd.print("Error");
  }
  
  // Check if alarm set time matches current time, if so, trigger alarm output pin and motor
  if (!alarmMode && alarmHour == hour() && alarmMinute == minute()) {
    digitalWrite(ALARM_OUTPUT_PIN, HIGH);
    Serial.println("Alarm triggered!");
    alarmTriggered = true; // Set flag to indicate alarm is triggered
  } else {
    digitalWrite(ALARM_OUTPUT_PIN, LOW);
    alarmTriggered = false; // Reset flag when alarm is not triggered
    // Commenting out to avoid redundant messages
    // Serial.println("Alarm not triggered.");
  }

  // Control motor driver based on current time or if alarm is triggered
  if (alarmTriggered) {
    digitalWrite(MOTOR_PIN1, HIGH); // Rotate motor in one direction
    digitalWrite(MOTOR_PIN2, LOW);
    Serial.println("Motor running due to alarm.");
  } else if (hour() == 10 && minute() == 0 && !alarmMode) {
    digitalWrite(MOTOR_PIN1, HIGH); // Rotate motor in one direction
    digitalWrite(MOTOR_PIN2, LOW);
    Serial.println("Motor rotating clockwise.");
  } else if (hour() == 15 && minute() == 0 && !alarmMode) {
    digitalWrite(MOTOR_PIN1, LOW); // Rotate motor in the opposite direction
    digitalWrite(MOTOR_PIN2, HIGH);
    Serial.println("Motor rotating counterclockwise.");
  } else {
    digitalWrite(MOTOR_PIN1, LOW); // Stop motor
    digitalWrite(MOTOR_PIN2, LOW);
    Serial.println("Motor stopped.");
  }
  
  // Delay for 1 second
  delay(1000);
}
