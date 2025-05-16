# Final-Project
# ESP32 WhatsApp Message Sender via CallMeBot


## Overview
This project is to use an **ESP32** to send an automatic **WhatsApp message** at startup, using the **CallMeBot API**. It demonstrates how to connect to Wi-Fi, make HTTP requests, and interact with external services using IoT hardware.



## What You Need

- ESP32 
- Micro USB cable
- Arduino IDE installed
- Internet connection
- WhatsApp on your phone
- CallMeBot API key (free)



## How It Works

1. The ESP32 connects to Wi-Fi.
2. Sends an HTTP GET request to the CallMeBot API with your WhatsApp number and message.
3. CallMeBot sends the message to your WhatsApp account.



## Setup Instructions

### 1. Get Your CallMeBot API Key
Send this exact message on WhatsApp to `+34 694 25 79 52`:
"I allow callmebot to send me messages"
CallMeBot will reply with your personal API key.



### 2. Configure Arduino IDE
- Install the **ESP32 board support** via Boards Manager.
- Board: `AI Thinker ESP32 cam `
- Baud rate: `115200`
- COM port: `Com 6 serial port usb`



## Code

```
#include <WiFi.h>
#include <HTTPClient.h>

// Replace with your credentials
const char* ssid = "your_wifi";
const char* password = "your_password";
String apiKey = "your_api_key"; // from CallMeBot
String phoneNumber = "+1xxxxxxxxxx"; // your full WhatsApp number

void setup() {
  Serial.begin(115200);
  delay(1000); // allow time for Serial to start

  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("Connected to WiFi");

  delay(2000); // wait to ensure Wi-Fi is fully ready
  sendWhatsAppMessage("Hello from ESP32!");
}

void loop() {
  // No repeated tasks — single message on startup only
}

String urlEncode(String str) {
  String encoded = "";
  char c;
  char code0, code1;
  for (int i = 0; i < str.length(); i++) {
    c = str.charAt(i);
    if (isalnum(c)) {
      encoded += c;
    } else {
      code1 = (c & 0xf) + '0';
      if ((c & 0xf) > 9) code1 = (c & 0xf) - 10 + 'A';
      c = (c >> 4) & 0xf;
      code0 = c + '0';
      if (c > 9) code0 = c - 10 + 'A';
      encoded += '%';
      encoded += code0;
      encoded += code1;
    }
  }
  return encoded;
}

void sendWhatsAppMessage(String message) {
  HTTPClient http;
  String encoded = urlEncode(message);
  String url = "https://api.callmebot.com/whatsapp.php?phone=" + phoneNumber + "&text=" + encoded + "&apikey=" + apiKey;

  Serial.println("Request URL: " + url);

  http.begin(url);
  int httpCode = http.GET();

  if (httpCode > 0) {
    String response = http.getString();
    Serial.println("API Response: " + response);
  } else {
    Serial.println("Error sending message: " + String(httpCode));
  }

  http.end();
}
```

Code Explanation
loop() {} — Why It's Empty
Arduino requires a loop() function, but this project only sends one message after startup. Since we don't need to repeat anything, loop() is present but left empty.

urlEncode() — Why It's Needed
This function ensures your message is properly formatted for a URL:
1. Special characters like spaces, punctuation, etc. are converted to percent-encoded format (e.g., " " becomes %20, "!" becomes %21).
2. Without encoding, the CallMeBot server returns a "Bad Request" error.


Troubleshooting Log

| Problem                             | Solution                                              
| `Error: unable to open ftdi device` | Used **Upload (→)** instead of **Debug** in Arduino IDE 
| Message not received on WhatsApp    | Added delay after Wi-Fi connected                       
| API responded with "Bad Request"    | Added `urlEncode()` function to format message               
| ESP32 uploads but no output         | Set **Serial Monitor baud rate to 115200**                   


Final Result
1. ESP32 connects to Wi-Fi
2. Sends WhatsApp message automatically
3. Confirmed working via Serial Monitor and phone notification


Files in This Repo
File                   | Description                 

| `esp32_whatsapp.ino` | Main Arduino sketch         |




Notes
1. CallMeBot has message rate limits — avoid sending too frequently
2. Only works with verified phone numbers and API keys










