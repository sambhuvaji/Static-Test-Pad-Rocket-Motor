# Static-Test-Pad-Rocket-Motor
Design and CAE of a Static Test Pad for both Vertical and Horizontal Rocket Mount Module.

Vertical STP : https://a360.co/3EoeIcd
Horizontal STP : https://a360.co/3rznGzz

Designed in Solidworks. CAE analysis in Fusion 360.
### Material : Steel AISI 4130 (contains Chromium and Molybdenum)

# Test Simulation Result for Vertical STP
## Displacement
![Vertical STP Al 7075 Displacement](https://user-images.githubusercontent.com/77199304/144700907-090347e1-3c3d-4799-9ec8-1cf070b42fb3.png)
## Stress
![Vertical STP Al 7075 Stress](https://user-images.githubusercontent.com/77199304/144700918-822f632e-7d2f-466d-be1e-e1bf6cc1f3ca.png)
## Reaction Force
![Vertical STP Al 7075 Reaction Force](https://user-images.githubusercontent.com/77199304/144700946-05047e87-18eb-42cc-8555-5eb87daf386e.png)


# Test Simulation Result for Horizontal STP
## Displacement
![Horizontal Test Pad AISI 4140 Displacement](https://user-images.githubusercontent.com/77199304/144700771-ea7ee785-cb32-430d-a31f-607f30a3dbb9.png)
## Stress
![Horizontal Test Pad AISI 4140 Stress](https://user-images.githubusercontent.com/77199304/144700821-c109ac69-3616-4388-82ad-49247005a574.png)
## Reaction Force
![Horizontal Test Pad AISI 4140 Reaction Force](https://user-images.githubusercontent.com/77199304/144700843-7abc2c85-fcad-457e-899f-fbc4818c3489.png)

# Materials Considered : Aluminium 7075 and Steel AISI 4130
![STP Simualtion Report 001](https://user-images.githubusercontent.com/77199304/144701077-4a2380bc-c173-40c7-8e88-7b0b9ee829a8.png)

![STP Simualtion Report Inference 001](https://user-images.githubusercontent.com/77199304/144702392-691e1da2-99e5-4214-ab68-5e4702efa323.png)

#Sketch

#include <Buzzer.h>
#include <HX711.h>
#include <LM35.h>
#include <LiquidCrystal.h> 
#include <SPI.h>
#include <SD.h>
LiquidCrystal lcd(9,8,4,5,6,7); 
// LCD module connections (RS, E, D4, D5, D6, D7)
const float tempPin(A0);
const int buzzerPin(A1); 
const int ledPin(A2); 
const int buttonPin(A3); 
const int igniterPin(A4);  
int buttonState = 0; 
const int myFile(10);
// start sequence button 
//LED indicator and/or buzzer 
//igniter transistor circuit
HX711 scale;
float calibration_factor = 000; // loadcell calibration factor
void setup() {
    pinMode(buttonPin, INPUT);
    pinMode(buzzerPin, OUTPUT);
    pinMode(igniterPin, OUTPUT);
    pinMode(ledPin, OUTPUT);
    Serial.begin(9600);
    Serial.println("HX711 Rocket Motor Testpad");
    Serial.println("Affix motor nozzle up. Place igniter in nozzle. Move away from test stand.");
    Serial.println("Press start button to initialize ignition sequence."); 
    lcd.begin(16, 2);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print(" MODEL ROCKET");
    lcd.setCursor(0,1);
    lcd.print(" Testpad");
    delay(2000);
    scale.set_scale();
    scale.tare(); //Reset the scale to 0
    long zero_factor = scale.read_average(); //Get a baseline reading
    //This can be used to remove the need to tare the scale.
    //Useful in permanent scale projects. 
    Serial.print("Zero factor: ");
    Serial.println(zero_factor); 
    Serial.println(" ");
}
    void loop() {
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print(" Rocket testpad"); 
    lcd.setCursor(0,1);
    lcd.print(" STDBY " ); 
    scale.set_scale(calibration_factor);
    SD.begin(myFile);
    lcd.print(scale.get_units(),1); 
    lcd.print(" g");
    delay(500);
    buttonState = digitalRead(buttonPin); 
    if (buttonState == HIGH) {
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print(" Rocket testpad");
        lcd.setCursor(0,1);
        lcd.print(" STAND CLEAR!"); 
        Serial.println("IGNITION SEQUENCE ACTIVATED!");
        for (int i=0; i <= 50; i++){ 
            digitalWrite(ledPin, HIGH); 
             tone(buzzerPin, 1000);
            delay (100);
            digitalWrite (ledPin,LOW); 
            noTone(buzzerPin);
            delay (100);
        }
         lcd.clear();
        lcd.setCursor(0,0);
        lcd.print(" Rocket testpad"); 
        lcd.setCursor(1,1);
        lcd.print(" AQUIRING DATA"); 
        digitalWrite(igniterPin, HIGH); 
        Serial.print("Start time, ms: "); 
        Serial.print (millis()); 
        Serial.println(" "); 
        Serial.println();
            File myFile = SD.open("test.csv", FILE_WRITE);
        for (int i=0; i <= 800; i++){ 
        //800 samples at 80sa/sec = 10 seconds theoretical 
            scale.set_scale(calibration_factor); 
        //Adjust to the calibration factor 
            Serial.print(tempPin); //gets the temperature in celcius and send to serial  
                  Serial.print(",");
                  float force = scale.get_units(2)*100;
                  myFile.println(force);
                  myFile.print(tempPin);
            Serial.print(scale.get_units(), 1);
            Serial.println(i);
        }
        Serial.println(); 
       float ms=millis();
        Serial.print("Stop Time, ms: "); 
        Serial.print(ms); 
        digitalWrite (ledPin,LOW); 
        digitalWrite (igniterPin,LOW); 
        Serial.println();
    } }
