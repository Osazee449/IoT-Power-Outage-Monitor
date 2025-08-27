# IoT-Power-Outage-Monitor
An ESP32-based IoT device that detects power failures and sends real-time email alerts via IFTTT
This system is designed to provide immediate awareness of a power grid failure. It uses a standard USB power bank to create a simple Uninterruptible Power Supply (UPS), ensuring the monitor itself remains operational during an outage. When the main 5V power from the wall adapter is lost, the ESP32 detects the voltage drop, connects to a local Wi-Fi network, and sends a webhook to the IFTTT cloud service, which in turn triggers an email notification.

## Components Used

* **Microcontroller:** ESP32 Development Board
* **Power Supply:** 5V USB Wall Adapter (Main) & 10,000mAh USB Power Bank (Backup)
* **Software:** Arduino IDE (C++), IFTTT Webhooks

## Circuit & Wiring
The ESP32 is powered continuously by the USB power bank. A "sense" wire is connected from the 5V line of the main wall adapter to GPIO pin 13 on the ESP32. A common ground is established between the main power line and the ESP32.

![A photo of my completed project breadboard](./IMG_4416.PNG)

## Source Code

Below is the complete, commented code for the project.

  /*
  IoT Power Outage & Grid Reliability Monitor
  Author: Osazee Imasuen
  Date: August 20, 2025


  Description:
  This code runs on an ESP32 to monitor the status of the main power supply.
  If the main power (sensed via a 5V line) is lost, the device connects to Wi-Fi
  and sends a webhook to IFTTT to trigger an email notification. The device
  is powered by a USB power bank to ensure it remains operational during an outage.
*/


// Include necessary libraries for WiFi and making HTTP requests
#include <WiFi.h>
#include <HTTPClient.h>


// --- IMPORTANT: CONFIGURE THESE SETTINGS ---
const char* ssid = "SpectrumSetup-02";         // Your Wi-Fi network name
const char* password = "purplenest895"; // Your Wi-Fi password


// Your IFTTT Webhook details
const char* ifttt_event_name = "power_outage"; // The event name you created
const char* ifttt_key = "dutsEs_PE3qeYrsLIA7Y__ ";      // Your unique key from the Webhooks documentation page


// Define the GPIO pin used to sense the main power
const int sensePin = 13; // Using GPIO 13 (D13)


// Variable to track the alert status to prevent spamming
bool alertSent = false;


// Function to send the notification to IFTTT
void sendNotification() {
  // Construct the IFTTT webhook URL
  String url = "http://maker.ifttt.com/trigger/";
  url += ifttt_event_name;
  url += "/with/key/";
  url += ifttt_key;


  // Create an HTTP client object
  HTTPClient http;


  // Start the HTTP request
  http.begin(url);
 
  // Send the GET request and get the HTTP response code
  int httpResponseCode = http.GET();


  // Check the response and print status to Serial Monitor for debugging
  if (httpResponseCode > 0) {
    Serial.print("Notification sent! HTTP Response code: ");
    Serial.println(httpResponseCode);
  } else {
    Serial.print("Error sending notification. HTTP Error code: ");
    Serial.println(httpResponseCode);
  }


  // Free resources
  http.end();
}


void setup() {
  // Start the Serial Monitor for debugging purposes
  Serial.begin(115200);
  Serial.println("Power Outage Monitor Initializing...");


  // Set the sense pin as an input
  pinMode(sensePin, INPUT);


  // Allow time for system to stabilize
  delay(1000);
}


void loop() {
  // Read the current state of the main power
  int powerState = digitalRead(sensePin);


  // Check if the main power is OFF
  if (powerState == LOW) {
    // Check if an alert has already been sent
    if (!alertSent) {
      Serial.println("POWER OUTAGE DETECTED!");
     
      // Connect to Wi-Fi
      Serial.print("Connecting to WiFi...");
      WiFi.begin(ssid, password);
     
      int wifi_retry_count = 0;
      while (WiFi.status() != WL_CONNECTED && wifi_retry_count < 20) {
        delay(500);
        Serial.print(".");
        wifi_retry_count++;
      }
      Serial.println();


      // If Wi-Fi connection is successful, send the notification
      if (WiFi.status() == WL_CONNECTED) {
        Serial.println("WiFi connected!");
        sendNotification();
        alertSent = true; // Mark the alert as sent
      } else {
        Serial.println("WiFi connection failed. Could not send alert.");
      }
    }
  }
  // If the main power is ON
  else {
    Serial.println("Power is ON. All systems normal.");
    // Reset the alert status so it can be triggered again if power fails later
    if (alertSent) {
      Serial.println("Resetting alert status.");
      alertSent = false;
    }
  }


  // Wait for 5 seconds before checking again
  delay(5000);
}




