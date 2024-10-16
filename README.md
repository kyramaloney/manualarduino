# Integrating Weather Data with Arduino: A Step-by-Step Guide to Controlling LED Strips

## Table of Contents
1. **Introduction**
2. **Materials Needed for API Integration**
3. **Step 1: Setting Up Your Development Environment**
4. **Step 2: Understanding the API**
5. **Step 3: Arduino Code Implementation**
6. **Step 4: Testing and Troubleshooting**
7. **Conclusion**

---

## 1. Introduction

This manual will guide you through connecting your Arduino to the OpenWeatherMap API to control an LED strip based on the current weather temperature. The LED strip will change colors according to different temperature ranges. This project is a fun way to learn about APIs, Arduino programming, and how to control hardware based on real-time data.

## 2. Materials Needed for API Integration

Before you start, make sure you have the following materials:

- **Arduino Board** (e.g., Arduino Uno or ESP8266)
- **LED Strip** (compatible with Arduino, like the NeoPixel strip)
- **Power Supply** (3V power supply for your LED strip)
- **Jumper Wires** (for connecting components)
- **Wi-Fi Connection** (to connect to the internet)
- **OpenWeatherMap API Key** (free to sign up on their website)
- **Arduino IDE** (for coding and uploading to your Arduino)

## 3. Step 1: Setting Up Your Development Environment

1. **Install the Arduino IDE**:
   - Download the Arduino IDE from the [official website](https://www.arduino.cc/en/software) and install it on your computer.

2. **Install the Required Libraries**:
   - Open the Arduino IDE and go to `Sketch` > `Include Library` > `Manage Libraries...`.
   - In the Library Manager, search for `Adafruit NeoPixel` and install it.
   - Make sure to also install `ArduinoJson` library for parsing JSON data.

3. **Connect Your Arduino**:
   - Use a USB cable to connect your Arduino board to your computer.
   - Select the correct board and port from `Tools` > `Board` and `Tools` > `Port`.

## 4. Step 2: Understanding the API

1. **What is an API?**  
   An API (Application Programming Interface) allows different software applications to communicate with each other. In this project, we will use the OpenWeatherMap API to get weather information.

2. **Getting Your API Key**:
   - Go to the [OpenWeatherMap website](https://openweathermap.org/).
   - Sign up for a free account.
   - After logging in, navigate to the API keys section in your account dashboard and copy your API key.

3. **Finding the City ID**:
   - Use the City Search feature on the OpenWeatherMap website to find your city. For Amsterdam, the city ID is `2759794`.
   - You can use this URL in your browser to confirm the city ID:
     ```
     http://api.openweathermap.org/data/2.5/weather?q=Amsterdam&appid=your_api_key
     ```
   - Replace `your_api_key` with the actual key you obtained.

## 5. Step 3: Arduino Code Implementation

Here is a simple example code that you can use. Copy and paste this into your Arduino IDE.

```cpp
#include <ESP8266WiFi.h>       // For ESP8266
#include <ArduinoJson.h>       // For parsing JSON data
#include <Adafruit_NeoPixel.h> // For controlling the LED strip

#define PIN D1                // Pin connected to the LED strip
#define NUMPIXELS 15          // Number of LEDs in the strip

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

// Wi-Fi credentials
const char* ssid = "yourwifi"; // Your Wi-Fi name
const char* password = "yourpassword"; // Your Wi-Fi password

// OpenWeatherMap API credentials
const String apiKey = "58e5ba5dccabe96136790bae5ce55a65"; // Your API key
const String cityID = "2759794"; // City ID for Amsterdam

// API endpoint
String weatherUrl = "http://api.openweathermap.org/data/2.5/weather?id=" + cityID + "&appid=" + apiKey + "&units=metric";

// Set up Wi-Fi client
WiFiClient client;

void setup() {
  Serial.begin(115200);
  pixels.begin();  // Initialize the NeoPixel strip
  connectToWiFi(); // Connect to Wi-Fi
  updateWeatherData(); // Get the initial weather data
}

void loop() {
  delay(600000); // Wait 10 minutes before checking again
  updateWeatherData(); // Update the weather data periodically
}

// Connect to Wi-Fi function
void connectToWiFi() {
  Serial.print("Connecting to Wi-Fi");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("\nConnected to Wi-Fi");
}

// Update weather data and change LED color
void updateWeatherData() {
  Serial.println("Getting weather data...");
  if (client.connect("api.openweathermap.org", 80)) {
    client.print(String("GET ") + weatherUrl + " HTTP/1.1\r\n" +
                 "Host: api.openweathermap.org\r\n" +
                 "Connection: close\r\n\r\n");

    while (client.connected() || client.available()) {
      if (client.available()) {
        String line = client.readStringUntil('\n');
        if (line.startsWith("{")) {
          parseWeatherData(line); // Parse the JSON response
          break;
        }
      }
    }
    client.stop();
  } else {
    Serial.println("Connection to OpenWeatherMap failed.");
  }
}

// Function to parse weather data
void parseWeatherData(String jsonData) {
  StaticJsonDocument<1024> doc;
  DeserializationError error = deserializeJson(doc, jsonData);

  if (error) {
    Serial.println("Failed to parse JSON");
    return;
  }

  float temp = doc["main"]["temp"]; // Get temperature
  Serial.print("Current temperature: ");
  Serial.println(temp);

  // Set LED color based on temperature
  if (temp > 30) {
    setLEDColor(255, 0, 0); // Red for above 30 degrees
  } else if (temp >= 25) {
    setLEDColor(0, 255, 0); // Green for 25+ degrees
  } else if (temp >= 20) {
    setLEDColor(0, 0, 255); // Blue for 20-25 degrees
  } else if (temp >= 15) {
    setLEDColor(255, 165, 0); // Orange for 15-20 degrees
  } else if (temp >= 10) {
    setLEDColor(128, 0, 128); // Purple for 10-15 degrees
  } else {
    setLEDColor(255, 255, 255); // White for below 10 degrees
  }
}

// Set LED color function
void setLEDColor(int red, int green, int blue) {
  for (int i = 0; i < NUMPIXELS; i++) {
    pixels.setPixelColor(i, pixels.Color(red, green, blue)); // Set color
  }
  pixels.show(); // Update the strip
}
```

## 6. Step 4: Testing and Troubleshooting

### Upload the Code:
After copying the code, click the upload button in the Arduino IDE to upload the code to your Arduino board.

### Monitor Serial Output:
Open the Serial Monitor (found under `Tools` > `Serial Monitor`) in the Arduino IDE to see real-time outputs. This will help you understand if the code is running correctly.

### Check LED Strip:
Ensure your LED strip is correctly connected to the specified pin (D1 in this case) and powered appropriately. If the LEDs do not light up, double-check your connections.

### Wi-Fi Connection Issues:
If you see "Connection to OpenWeatherMap failed," verify your Wi-Fi credentials. Make sure the SSID and password are correct.

### No Weather Data:
If the temperature does not change the LED colors, ensure the API key is valid and check the city ID.

## 7. Conclusion

You have successfully connected your Arduino to the OpenWeatherMap API to control an LED strip based on temperature. This project is a great way to learn about IoT (Internet of Things) applications and real-time data integration. Experiment with the code and feel free to make improvements or adjustments to fit your preferences!
