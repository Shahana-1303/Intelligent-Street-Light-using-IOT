# Intelligent-Street-Light-using-IOT
This project aims to enhance urban street lighting by developing an Intelligent Solar-Powered Street Light system integrated with IoT technology.     
By reducing reliance on non-renewable energy and enabling real-time brightness adjustments based on traffic and environmental conditions, the system promotes sustainability. 
IoT integration allows for continuous monitoring and remote fault detection, lowering maintenance costs and downtime. 
Overall, this initiative seeks to modernize street lighting infrastructure, improve safety, and create a more environmentally friendly urban environment.
#include &lt;Wire.h&gt;
#include &lt;RTClib.h&gt;
#include &lt;LiquidCrystal_I2C.h&gt;
#include &lt;SoftwareSerial.h&gt;
// Initialize RTC
RTC_DS3231 rtc;
// Initialize GSM on pins 12 (Rx) and 11 (Tx)
SoftwareSerial gsmSerial(12, 11);
// Initialize LCD (I2C address 0x27, 16 columns, 2 rows)
LiquidCrystal_I2C lcd(0x27, 16, 2);
// Define pins
#define PIR_PIN 10
#define STREET_LIGHT_PIN 3
#define BUTTON_1_PIN 8
#define BUTTON_2_PIN 9
// Define phone numbers
const char *POLICE_NUMBER = &quot;7010453547&quot;;
const char *MAINTENANCE_NUMBER = &quot;8778077054&quot;;
unsigned long pirLastActivated = 0; // To keep track of PIR activation
const unsigned long PIR_DELAY = 30000; // 30 seconds delay
bool rtcWorking = true; // Flag to check if RTC is working
void setup() {
Serial.begin(9600);
gsmSerial.begin(9600);
pinMode(PIR_PIN, INPUT);
pinMode(STREET_LIGHT_PIN, OUTPUT);
pinMode(BUTTON_1_PIN, INPUT_PULLUP);
pinMode(BUTTON_2_PIN, INPUT_PULLUP);
// Initialize LCD
lcd.init();
lcd.backlight();
lcd.clear();
lcd.setCursor(0, 0);
lcd.print(&quot;Street Light Ctrl&quot;);
delay(2000); // Display startup message for 2 seconds
// Initial GSM check
gsmSerial.println(&quot;AT&quot;);
delay(1000);
if (gsmSerial.available()) {
Serial.println(gsmSerial.readString());
}
// Initialize RTC and check if it&#39;s working
if (!rtc.begin()) {
Serial.println(&quot;Couldn&#39;t find RTC&quot;);
rtcWorking = false;
sendSMS(MAINTENANCE_NUMBER, &quot;Error: RTC module not detected.&quot;);
lcd.setCursor(0, 1);
lcd.print(&quot;RTC Error!&quot;);
delay(2000);
lcd.clear();
} else {
// Set RTC time to local system time (PC) with IST offset (UTC+5:30)
DateTime now = DateTime(_DATE, TIME_);
now = now + TimeSpan(5 * 3600 + 30 * 60); // IST offset (+5 hours 30 minutes)
rtc.adjust(now);
// Send initial SMS to maintenance team that system is active
sendSMS(MAINTENANCE_NUMBER, &quot;System activated: Street light control
system.&quot;);
lcd.setCursor(0, 1);
lcd.print(&quot;System Activated&quot;);
delay(2000);
lcd.clear();
}
}
void loop() {
if (rtcWorking) {
DateTime now = rtc.now();
int hour = now.hour();
int minute = now.minute();
// Display time on LCD
lcd.setCursor(0, 0);
lcd.print(&quot;Time: &quot;);
if (hour &lt; 10) lcd.print(&#39;0&#39;);
lcd.print(hour);
lcd.print(&quot;:&quot;);
if (minute &lt; 10) lcd.print(&#39;0&#39;);
lcd.print(minute);
// Check if button 1 is pressed to send SMS to police team
if (digitalRead(BUTTON_1_PIN) == HIGH) {
sendSMS(POLICE_NUMBER, &quot;Alert! Some one is need a help on the street light
near area.&quot;);
lcd.setCursor(0, 1);
lcd.print(&quot;Msg Sent: Police&quot;);
delay(2000); // Message display delay
lcd.clear();
}
// Check if button 2 is pressed to send SMS to maintenance team
if (digitalRead(BUTTON_2_PIN) == HIGH) {
sendSMS(MAINTENANCE_NUMBER, &quot;SSL12 - Issue with street light.&quot;);
lcd.setCursor(0, 1);
lcd.print(&quot;Msg Sent: Maint.&quot;);
delay(2000); // Message display delay
lcd.clear();
}
// Street light control based on time
if ((hour &gt;= 18 || hour &lt; 6)) {
if (hour &gt;= 18 &amp;&amp; hour &lt; 22) { // 6 PM to 10 PM: Full brightness
analogWrite(STREET_LIGHT_PIN, 255);
lcd.setCursor(0, 1);
lcd.print(&quot;Full Brightness &quot;);
} else if (hour &gt;= 22 || hour &lt; 3) { // 10 PM to 3 AM: Half brightness
analogWrite(STREET_LIGHT_PIN, 128);
lcd.setCursor(0, 1);
lcd.print(&quot;Half Brightness &quot;);
// Check PIR sensor during this time
if (digitalRead(PIR_PIN) == HIGH) {
analogWrite(STREET_LIGHT_PIN, 255); // Full brightness on motion
pirLastActivated = millis(); // Reset PIR activation time
lcd.setCursor(0, 1);
lcd.print(&quot;Motion: Full Bright&quot;);
}
// Revert to half brightness after 30 seconds if no motion
if (millis() - pirLastActivated &gt; PIR_DELAY) {
analogWrite(STREET_LIGHT_PIN, 128);
lcd.setCursor(0, 1);
lcd.print(&quot;Half Brightness &quot;);
}
} else if (hour &gt;= 3 &amp;&amp; hour &lt; 6) { // 3 AM to 6 AM: Full brightness
analogWrite(STREET_LIGHT_PIN, 255);
lcd.setCursor(0, 1);
lcd.print(&quot;Full Brightness &quot;);
}
} else {
analogWrite(STREET_LIGHT_PIN, 0); // Turn off the light from 6 AM to 6 PM
lcd.setCursor(0, 1);
lcd.print(&quot;Light Off &quot;);
}
}
delay(1000); // Loop every second
}
void sendSMS(const char *number, const char *message) {
gsmSerial.print(&quot;AT+CMGF=1\r&quot;); // Set SMS to text mode
delay(1000);
gsmSerial.print(&quot;AT+CMGS=\&quot;&quot;);
gsmSerial.print(number);
gsmSerial.println(&quot;\&quot;&quot;);
delay(1000);
gsmSerial.print(message);
delay(1000);
gsmSerial.write(26); // ASCII code of CTRL+Z to send SMS
delay(3000);
}
