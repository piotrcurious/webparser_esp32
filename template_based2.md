Expanding on the previous example, let's look at how to extract more complex data from a web page like the population data from Worldometer's population page. This page contains dynamic and structured content, including world population counters, which are continuously updated.

To parse data from such a page using template-based parsing, you can follow the same principles but adjust for more intricate structures. Specifically, you’ll define multiple templates for different substructures within the page and use these templates to extract data like the total population, births, and deaths today.

Overview of the Strategy

1. Identify static patterns: Look for static HTML markers that surround the data you want to extract. Even if the page has dynamic data (such as real-time counters), the HTML structure itself often remains consistent.


2. Define templates: Use these static patterns to create templates with placeholders for the data you want to extract.


3. Implement parsing logic: Extract data based on the templates by matching the static HTML around the dynamic data.



Step 1: Inspecting the HTML Structure

Let’s start by looking at the structure of the Worldometer population page. Here's an example of the HTML structure for the world population counter:

<div class="maincounter-number">
    <span style="color:#1E7D22;" class="rts-counter">8,045,250,330</span>
</div>

You can use the <div class="maincounter-number"> as the static marker for the world population, with the actual number wrapped inside the <span class="rts-counter"> tag.

Step 2: Define Templates

We can define the following templates for parsing data from the page:

1. World Population Template

const char populationTemplate[] PROGMEM = R"rawliteral(
<div class="maincounter-number">
    <span class="rts-counter">%POPULATION%</span>
</div>)rawliteral";


2. Births Today Template

For the births today counter, the structure might look like this:

<div class="sec-counter">
    <span class="rts-counter">224,673</span>
</div>

We can create the following template:

const char birthsTodayTemplate[] PROGMEM = R"rawliteral(
<div class="sec-counter">
    <span class="rts-counter">%BIRTHS_TODAY%</span>
</div>)rawliteral";


3. Deaths Today Template

Similarly, we can define a template for the deaths today counter:

const char deathsTodayTemplate[] PROGMEM = R"rawliteral(
<div class="sec-counter">
    <span class="rts-counter">%DEATHS_TODAY%</span>
</div>)rawliteral";



Step 3: Implement Parsing Logic

Now let’s implement the logic to extract data from these templates. The process is similar to what we used earlier:

Use the static parts of the template to locate the start and end of the relevant sections in the HTML.

Extract the dynamic data by identifying the portion between the placeholders.


Full Code Example

#include <WiFi.h>
#include <HTTPClient.h>

// WiFi credentials
const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";

// Templates for world population, births, and deaths
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

// Extract data from the HTML using template markers
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

// Function to fetch and parse world population data
void fetchWorldPopulationData() {
    HTTPClient http;
    http.begin("https://www.worldometers.info/world-population/");
    int httpCode = http.GET();

    if (httpCode > 0) {
        // Get the HTML payload
        String payload = http.getString();

        // Extract data using templates
        String population = extractFromTemplate(payload, FPSTR(populationTemplate), "%POPULATION%");
        String birthsToday = extractFromTemplate(payload, FPSTR(birthsTodayTemplate), "%BIRTHS_TODAY%");
        String deathsToday = extractFromTemplate(payload, FPSTR(deathsTodayTemplate), "%DEATHS_TODAY%");

        // Print extracted data to the Serial Monitor
        Serial.println("World Population: " + population);
        Serial.println("Births Today: " + birthsToday);
        Serial.println("Deaths Today: " + deathsToday);
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

How This Works:

1. Templates: The templates define static sections of the HTML and placeholders (%POPULATION%, %BIRTHS_TODAY%, and %DEATHS_TODAY%) for the dynamic data.


2. Data Extraction: The extractFromTemplate function searches for the static start and end markers in the HTML and extracts the data in between them. This data corresponds to the dynamic content (e.g., population numbers).


3. HTTP Request: The ESP32 sends a GET request to the Worldometer population page, retrieves the HTML, and parses it according to the templates.



Step 4: Adapting to Page Changes

One of the key advantages of this template-based approach is that if the page structure changes slightly, you can simply modify the templates without having to rewrite the core parsing logic. For example, if Worldometer changes how they structure their population data, you can update the template accordingly, and the rest of the code will still work as expected.

Improvements and Considerations

Handling More Complex Pages: If the page contains JavaScript-rendered content (i.e., data populated by JavaScript on the client side), this approach might not work since the ESP32 doesn’t execute JavaScript. In such cases, you might need a different approach, such as fetching data from an API or using a server-side solution.

Error Handling: It’s good practice to add more robust error handling, such as checking if the extracted data is valid before using it, or retrying requests if the server is down.


Conclusion

Using template-based parsing on an ESP32 to extract data from a web page like Worldometer’s population page is a scalable, maintainable approach. You separate the page structure (stored in templates) from the extraction logic, allowing you to easily adapt to changes in the web page without modifying the core parsing code. This method is resource-efficient and works well for embedded systems like the ESP32 with limited computational power.

