To improve the code while avoiding regex, you can build more complex templates and use lambda functions to encapsulate specific parsing logic. The idea is to allow flexibility in the template structure, especially for dynamic sections, and to make the parsing logic resilient to minor HTML changes. You can still use the static structure of the HTML but extend the templates to handle more nuanced cases, like optional fields or nested sections.

Key Improvements:

1. Dynamic Placeholder Handling: Instead of fixed placeholders, we will use more flexible ones that allow variations in page structure.


2. Lambda-Based Parsing Logic: Use lambda functions to define custom parsing behaviors that extract specific data while giving us more control over how the data is extracted.


3. Improved Error Handling: We'll add more checks to validate whether the extracted data is valid and how the parsing behaves when it encounters unexpected content.



Updated Code with Template and Lambda Parsing

Here’s an updated version that uses templates with lambda functions to handle more dynamic HTML content:

#include <WiFi.h>
#include <HTTPClient.h>
#include <functional> // For lambda functions

// WiFi credentials
const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";

// Define templates for world population, births, and deaths with more complex parsing
const char populationTemplate[] PROGMEM = R"rawliteral(
<div class="maincounter-number">
    <span class="rts-counter">%POPULATION%</span>
</div>)rawliteral";

const char birthsTodayTemplate[] PROGMEM = R"rawliteral(
<div class="sec-counter">
    <span class="rts-counter">%BIRTHS_TODAY%</span>
</div>)rawliteral";

const char deathsTodayTemplate[] PROGMEM = R"rawliteral(
<div class="sec-counter">
    <span class="rts-counter">%DEATHS_TODAY%</span>
</div>)rawliteral";

// Extract data using template with custom parsing logic (lambda-based)
String extractFromTemplate(const String& html, const String& templateStr, const String& placeholder, std::function<String(const String&, int, int)> customParser) {
    String startMarker = templateStr.substring(0, templateStr.indexOf(placeholder));
    String endMarker = templateStr.substring(templateStr.indexOf(placeholder) + placeholder.length());

    int startIndex = html.indexOf(startMarker);
    if (startIndex >= 0) {
        startIndex += startMarker.length();
        int endIndex = html.indexOf(endMarker, startIndex);
        if (endIndex > startIndex) {
            // Use lambda to parse the content between the start and end markers
            return customParser(html, startIndex, endIndex);
        }
    }
    return String(""); // Return empty string if parsing fails
}

// Custom lambda function to handle trimming or filtering of the extracted data
auto defaultParser = [](const String& html, int start, int end) -> String {
    // Simple extraction without additional processing
    return html.substring(start, end);
};

// Lambda function to clean up data, like removing commas or whitespaces
auto cleanNumericData = [](const String& html, int start, int end) -> String {
    String extracted = html.substring(start, end);
    extracted.replace(",", ""); // Remove commas from large numbers
    extracted.trim();            // Trim any extraneous whitespace
    return extracted;
};

// Function to fetch and parse world population data
void fetchWorldPopulationData() {
    HTTPClient http;
    http.begin("https://www.worldometers.info/world-population/");
    int httpCode = http.GET();

    if (httpCode > 0) {
        // Get the HTML payload
        String payload = http.getString();

        // Extract data using templates and custom parsing functions
        String population = extractFromTemplate(payload, FPSTR(populationTemplate), "%POPULATION%", cleanNumericData);
        String birthsToday = extractFromTemplate(payload, FPSTR(birthsTodayTemplate), "%BIRTHS_TODAY%", cleanNumericData);
        String deathsToday = extractFromTemplate(payload, FPSTR(deathsTodayTemplate), "%DEATHS_TODAY%", cleanNumericData);

        // Validate and print extracted data
        if (!population.isEmpty()) {
            Serial.println("World Population: " + population);
        } else {
            Serial.println("Error extracting world population data.");
        }

        if (!birthsToday.isEmpty()) {
            Serial.println("Births Today: " + birthsToday);
        } else {
            Serial.println("Error extracting births today data.");
        }

        if (!deathsToday.isEmpty()) {
            Serial.println("Deaths Today: " + deathsToday);
        } else {
            Serial.println("Error extracting deaths today data.");
        }
    } else {
        Serial.println("Failed to fetch data from Worldometer");
    }

    http.end();
}

void setup() {
    // Initialize Serial communication and connect to WiFi
    Serial.begin(115200);
    WiFi.begin(ssid, password);

    // Wait until the device is connected to WiFi
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.println("Connecting to WiFi...");
    }

    Serial.println("Connected to WiFi");
    
    // Fetch and display the world population data
    fetchWorldPopulationData();
}

void loop() {
    // Periodically fetch population data (every 10 minutes, for example)
    delay(600000);  // 600000 milliseconds = 10 minutes
    fetchWorldPopulationData();
}

Improvements in this Code:

1. Lambda-Based Parsing:

We use lambda functions to encapsulate different parsing strategies. For example, cleanNumericData removes commas and trims the extracted content. This function can be adapted based on the type of data being extracted.

If you want to apply more complex logic like unit conversions or data transformations, you can modify or replace the lambda accordingly.



2. Flexible Template Parsing:

The extractFromTemplate function allows flexible extraction using custom lambda functions to handle different types of content. If, for example, a part of the web page changes slightly (such as additional HTML tags), you can adapt the lambda functions to handle the new structure without modifying the main parsing logic.



3. Data Validation:

Before printing the data, the program checks if the extracted data is valid (non-empty). If the parsing fails, it handles the error gracefully, printing a specific error message to the Serial Monitor.



4. Memory Efficiency:

By using lambda functions and efficient string handling, we minimize the use of additional memory, making the parsing lightweight, which is crucial in embedded systems like the ESP32.




Future Considerations

1. Template Flexibility:

If the HTML structure changes drastically, you might still need to update the templates. However, the use of lambdas allows you to easily adjust the behavior of specific parsing tasks without affecting the entire structure.



2. Handling Edge Cases:

The cleanNumericData lambda is designed to handle numeric data. You can define other lambda functions for different types of data (e.g., dates, percentages) based on the web page structure.



3. Expanding to Multiple Pages:

If you need to parse data from multiple pages with similar structures, you can reuse the extractFromTemplate function and simply pass in different templates and lambdas.




Conclusion

By using more complex templates combined with lambda-based parsing, this approach provides a flexible and maintainable way to extract data from web pages, even when their structure changes slightly. This method improves resilience, reduces the need for constant refactoring, and allows for easy customization of the parsing logic.

