I understand now—you're referring to using page templates to facilitate data extraction from web pages accessed by the ESP32, not just for generating web pages. This approach allows the ESP32 to extract data in a structured way, even if the page’s structure changes. The idea is to define templates for substructures in the page and dynamically adapt the extraction process based on these templates, ensuring the parsing logic remains robust and maintainable.

How Template-Based Parsing for Data Extraction Works

In this approach, templates represent specific structures or substructures within the HTML page. These templates define patterns for the sections where dynamic data resides, allowing the parsing logic to extract content without being tightly coupled to the exact HTML structure. If the web page changes slightly, only the template needs to be updated, leaving the core parsing logic unchanged.

Key Components of Template-Based Parsing

1. Defining Substructure Templates Each template corresponds to a specific segment of the web page, like a table, a div block, or a paragraph, where the data is located. These templates define static markers that remain consistent, even if the content between them changes.

Example: Let's say you're extracting weather data from a web page. A template might define the start and end of a <div> section that contains the temperature.


const char weatherTemplate[] PROGMEM = R"rawliteral(
<div class="weather-data">
    Temperature: %TEMPERATURE%°C
</div>)rawliteral";

Here, %TEMPERATURE% is a placeholder for the dynamic data. When you parse the HTML page, you'll look for the static markers (<div class="weather-data">) and extract the value between them.



2. Parsing Logic Using Templates The ESP32 accesses the web page and scans through the HTML, matching against predefined templates. Once a match is found, the parsing logic extracts the dynamic data by recognizing the placeholders in the template.

Example: A parsing function could identify the beginning and end of the static sections and then replace the placeholder %TEMPERATURE% with the actual data from the HTML.


String extractFromTemplate(const String& html, const String& templateStr, const String& placeholder) {
    String startMarker = templateStr.substring(0, templateStr.indexOf(placeholder));
    String endMarker = templateStr.substring(templateStr.indexOf(placeholder) + placeholder.length());

    int startIndex = html.indexOf(startMarker);
    if (startIndex >= 0) {
        startIndex += startMarker.length();
        int endIndex = html.indexOf(endMarker, startIndex);
        if (endIndex > startIndex) {
            return html.substring(startIndex, endIndex);
        }
    }
    return String("");
}

void parseWeatherPage(const String& html) {
    String temperature = extractFromTemplate(html, FPSTR(weatherTemplate), "%TEMPERATURE%");
    Serial.println("Extracted Temperature: " + temperature);
}

In this example:

The extractFromTemplate function uses the start marker (before the placeholder) and the end marker (after the placeholder) to extract the temperature value from the HTML page.

You can use multiple templates to extract other data like humidity, wind speed, etc.



3. Handling Multiple Substructures You can define different templates for various sections of the web page, allowing you to extract specific pieces of data independently. If the structure of a web page changes, only the relevant template needs updating, while the overall parsing logic stays the same.

Example: Extracting both temperature and humidity using different templates:


const char humidityTemplate[] PROGMEM = R"rawliteral(
<div class="humidity-data">
    Humidity: %HUMIDITY%%
</div>)rawliteral";

void parseWeatherPage(const String& html) {
    String temperature = extractFromTemplate(html, FPSTR(weatherTemplate), "%TEMPERATURE%");
    String humidity = extractFromTemplate(html, FPSTR(humidityTemplate), "%HUMIDITY%");
    Serial.println("Extracted Temperature: " + temperature);
    Serial.println("Extracted Humidity: " + humidity);
}

By defining separate templates for temperature and humidity, you ensure that each piece of data is extracted independently, even if the overall page structure evolves over time.



Advantages of Template-Based Page Parsing

1. Maintainability If the page structure changes slightly, you don't need to rewrite complex parsing code. Instead, you only update the relevant templates to match the new structure, allowing the core parsing logic to remain the same.


2. Separation of Concerns Templates isolate the structure of the web page from the extraction logic. The extraction process doesn’t depend on the exact HTML layout but rather on the consistent static markers defined in the template.


3. Reusable Templates If multiple web pages or different parts of the same page share similar structures, you can reuse templates across different sections or pages, reducing code redundancy.


4. Modular Parsing Parsing logic becomes modular since each template corresponds to a specific part of the web page. If one section changes, you only modify the template for that section, not the entire parsing logic.



Example: Complete Template-Based Parsing on ESP32

Here’s a more complete example demonstrating the use of multiple templates to extract weather data from a web page:

#include <WiFi.h>
#include <HTTPClient.h>

const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";

// Define templates for temperature and humidity
const char weatherTemplate[] PROGMEM = R"rawliteral(
<div class="weather-data">
    Temperature: %TEMPERATURE%°C
</div>)rawliteral";

const char humidityTemplate[] PROGMEM = R"rawliteral(
<div class="humidity-data">
    Humidity: %HUMIDITY%%
</div>)rawliteral";

// Extract data using the template
String extractFromTemplate(const String& html, const String& templateStr, const String& placeholder) {
    String startMarker = templateStr.substring(0, templateStr.indexOf(placeholder));
    String endMarker = templateStr.substring(templateStr.indexOf(placeholder) + placeholder.length());
    
    int startIndex = html.indexOf(startMarker);
    if (startIndex >= 0) {
        startIndex += startMarker.length();
        int endIndex = html.indexOf(endMarker, startIndex);
        if (endIndex > startIndex) {
            return html.substring(startIndex, endIndex);
        }
    }
    return String("");
}

// Function to fetch and parse weather data
void fetchWeatherData() {
    HTTPClient http;
    http.begin("http://example.com/weather");
    int httpCode = http.GET();

    if (httpCode > 0) {
        String payload = http.getString();
        String temperature = extractFromTemplate(payload, FPSTR(weatherTemplate), "%TEMPERATURE%");
        String humidity = extractFromTemplate(payload, FPSTR(humidityTemplate), "%HUMIDITY%");
        Serial.println("Temperature: " + temperature);
        Serial.println("Humidity: " + humidity);
    }
    else {
        Serial.println("Failed to fetch data");
    }
    
    http.end();
}

void setup() {
    Serial.begin(115200);
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.println("Connecting to WiFi...");
    }

    Serial.println("Connected to WiFi");
    fetchWeatherData();
}

void loop() {
    // Periodically fetch weather data
    delay(60000);
    fetchWeatherData();
}

Conclusion

Using page templates for data extraction on an ESP32-based system allows you to build flexible and maintainable parsing logic. By defining static templates for various sections of the web page, you ensure that your parsing code can easily adapt to changes in the HTML structure without requiring major code rewrites. This approach is particularly useful in embedded systems with limited resources, where efficient memory and processing are critical.

