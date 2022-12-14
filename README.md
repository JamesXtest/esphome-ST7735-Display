# esphome-ST7735-Display
esphome ST7735 Display  
Under construction, will be update soon...  

<br>
![Screenshot](https://github.com/JamesXtest/esphome-ST7735-Display/main/images/openweathermap.png)
<br>

> ST7735 Display drivers:   
```
captive_portal:

# Connections:
# Display VCC → Board 3.3v
# Display GND → Board GND
# Display SCL → D15  Board SCK GPIO14
# Display SDA → D14 Board MOSI GPIO13
# Display RES → D13 Board GPIO27
# Display DC → D12 Board GPIO26
# Display CS → D11 Board SS GPIO25
# Display BL → N.C
spi:
  clk_pin: GPIO14
  mosi_pin: GPIO13

display:
  - platform: st7735
    model: "INITR_18BLACKTAB"
    id: esp32_display
    reset_pin: GPIO27
    dc_pin: GPIO26
    cs_pin: GPIO25

    rotation: 0
    device_width: 130
    device_height: 162
    col_start: 0
    row_start: 0
    # eight_bit_color: True
    # use_bgr: True
    update_interval: 5s
    setup_priority: -100
```
> Pages:   
```
display:
  ......
    pages:
      - id: page1
        lambda: |-
          ......
      - id: page2
        lambda: |-
          ......
```
> Show next page after 180s:   
```
interval:
  - interval: 180s
    then:
      - display.page.show_next: esp32_display
```
> Show next page by press a button connected to GPIO25:   
```
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO25
      inverted: true
      mode:
        input: true
        pullup: true
    name: "ESP32 Button 1"
    internal: true
    # device_class: "occupancy"
    on_press:
      then:
        - display.page.show_next: esp32_display
        - component.update: esp32_display
```
> Define color:   
```    
display:
  - platform: st7735
    ......
    pages:
      - id: page1
        lambda: |-
          auto red = Color(255, 0, 0);
          auto lime = Color(0, 255, 0);
          auto blue = Color(0, 0, 255);
```
> Define fonts (English), put ttf file into \\HOMEASSISTANT\config\esphome\fonts:   
```    
font:
  - file: 'fonts/MYRIADPRO-SEMIBOLD.ttf'
    id: font_e1
    size: 12
    glyphs: μ³!"%()+,-_.:°/0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz
```
> Define fonts (Chinese), put ttf file into \\HOMEASSISTANT\config\esphome\fonts:   
```    
font:
  - file: 'fonts/msjh.ttf'
    id: font_c1
    size: 12
    glyphs: 年月日時分秒星期間一二三四五六七九十百千萬室內外溫濕光度空氣質素香港電台第μ³!"%()+,-_.:°/0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz
```
> Define fonts (Icon), put ttf file into \\HOMEASSISTANT\config\esphome\fonts:   
```    
font:
  - file: 'fonts/materialdesignicons-webfont.ttf'
    id: weather_font
    size: 96
    glyphs:
      - "\U000F0594" # clear-night
      - "\U000F0590" # cloudy
```
> Display time:   
```
time:
  - platform: homeassistant
    id: esptime
    timezone: Asia/Hong_Kong
    
display:
  - platform: st7735
    ......
    pages:
      - id: page1
        lambda: |-
          // Green
          auto green = Color(0, 128, 0);
          ......          
          it.strftime(x, y, id(font_d2), green, TextAlign::CENTER, "%l:%M", id(esptime).now());    
          it.strftime(x, y, id(font_c1), green, TextAlign::TOP_LEFT, "%-m月 %-d日", id(esptime).now());
          it.strftime(z, y+110, id(font_e1), green, TextAlign::TOP_RIGHT, "%p", id(esptime).now());
    
```
> Display weekday in Chinese:
> - Copy below to your configuration.yaml and then restart
```   
sensor:
    ......        
  - platform: template
    sensors:
      weekday_chinese:
        friendly_name: "Chinese weekday"
        icon_template: 'mdi:calendar-clock'
        value_template: >-
            {% set wk = now().strftime('%w') | int %}
            {% set dayz = ["星期日", "星期一","星期二","星期三","星期四","星期五","星期六"] %}
            {{dayz[wk + 0]}}
```
> - copy below to your esphome.yaml   
```    
text_sensor:
    ......
  - platform: homeassistant
    id: weekday_chinese
    entity_id: sensor.weekday_chinese
    internal: true

display:
  - platform: st7735
    ......
    pages:
      - id: page1
        lambda: |-
          ...... 
          it.printf(x, y, id(font_c1), green, TextAlign::TOP_LEFT, "%s", id(weekday_chinese).state.c_str());
```
> Display WiFi signal icon:   
```
sensor:
  - platform: wifi_signal
    name: "ESP32_D1 WiFi signal"
    update_interval: 20s   

  - platform: homeassistant
    id: esp32_d1_wifi_sig
    entity_id: sensor.esp32_d1_wifi_signal
    internal: true

font:
  - file: 'fonts/materialdesignicons-webfont.ttf'
    id: icon_font_16
    size: 16
    glyphs:
      - "\U000F092E" # mdi-wifi-strength-off-outline
      - "\U000F0920" # mdi-wifi-strength-1-alert
      - "\U000F0922" # mdi-wifi-strength-2
      - "\U000F0925" # mdi-wifi-strength-3
      - "\U000F0928" # mdi-wifi-strength-4
      ......   
    
display:
  - platform: st7735
    ......
    pages:
      - id: page1
        lambda: |-
          // Green
          auto green = Color(0, 128, 0);
          ......          
          if(id(esp32_d1_wifi_sig).has_state()) {
            if (id(esp32_d1_wifi_sig).state >= -50) {
                //Excellent
                it.print(z, y, id(icon_font_16), green, TextAlign::TOP_RIGHT, "\U000F0928");
            } else if (id(esp32_d1_wifi_sig).state  >= -60) {
                //Good                
                it.print(z, y, id(icon_font_16), green, TextAlign::TOP_RIGHT, "\U000F0925");
            }
            ...... 
            else {
                //Unlikely working signal
                it.print(z, y, id(icon_font_16), green, TextAlign::TOP_RIGHT, "\U000F092E");
            }
          }

```
> Display info from OpenWeatherMap:   
> - Found out your location and copy below to your configuration.yaml and then restart
```   
homeassistant:
  name: my_home                 # as per your own setting
  latitude:                     # as per your location
  longitude:                    # as per your location
  unit_system: metric           # as per your own setting
  time_zone: Asia/Hong_Kong     # as per your location
```
> - https://www.home-assistant.io/integrations/openweathermap/
> - Look for the data from OpenWeatherMap as below,    
  Developer Tools > STATES
  <p>This is a testing.....<b>IMAGE</b></p>
  
  <img src = “images/openweathermap.png”>
  <br>


> - Copy below to your configuration.yaml and then restart
```   
template:
    sensor:
      - name: Todays Forecast Pressure
        icon: "mdi:arrow-collapse-down"
        state: >
          {{ states.weather.openweathermap.attributes.pressure }}
      - name: Todays Forecast Wind Bearing
        icon: "mdi:windsock"
        state: >
          {{ states.weather.openweathermap.attributes.wind_bearing }}
      - name: Todays Forecast Wind Speed
        icon: "mdi:windsock"
        state: >
          {{ states.weather.openweathermap.attributes.wind_speed }}
```
> - copy below to your esphome.yaml   
```    
text_sensor:
    ......
  - platform: homeassistant
    entity_id: weather.openweathermap
    id: weather_icon
    internal: true

sensor:
    ......
  - platform: homeassistant
    id: outside_temp_openweathermap
    entity_id: sensor.openweathermap_temperature
    internal: true

  - platform: homeassistant
    id: outside_humi_openweathermap
    entity_id: sensor.openweathermap_humidity
    internal: true

  - platform: homeassistant
    id: outside_rain_openweathermap
    entity_id: sensor.openweathermap_rain
    internal: true

display:
  - platform: st7735
    ......
    pages:
      - id: page2
      ......
          auto carolina_blue = Color(81, 160, 213);
          auto ylnmn_blue = Color(44, 82, 140);
          auto yellow = Color(204, 255, 0);
          auto gray = Color(128, 128, 128);

          int x, y, z;
          x = 65, y = 0;
          if (id(weather_icon).state == "clear-night") {
            it.printf(x, y, id(weather_font), yellow, TextAlign::TOP_CENTER, "\U000F0594");}
          if (id(weather_icon).state == "cloudy") {
            it.printf(x, y, id(weather_font), gray, TextAlign::TOP_CENTER, "\U000F0590");}
          if (id(weather_icon).state == "fog") {
            it.printf(x, y, id(weather_font),  gray, TextAlign::TOP_CENTER, "\U000F0591");}
          if (id(weather_icon).state == "hail") {
            it.printf(x, y, id(weather_font), gray, TextAlign::TOP_CENTER, "\U000F0592");}
          if (id(weather_icon).state == "lightning") {
            it.printf(x, y, id(weather_font), gray, TextAlign::TOP_CENTER, "\U000F0593");}
          if (id(weather_icon).state == "lightning-rainy") {
            it.printf(x, y, id(weather_font), gray, TextAlign::TOP_CENTER, "\U000F067E");}
          if (id(weather_icon).state == "partlycloudy") {
            it.printf(x, y, id(weather_font), yellow, TextAlign::TOP_CENTER, "\U000F0595");}
          if (id(weather_icon).state == "pouring") {
            it.printf(x, y, id(weather_font), gray, TextAlign::TOP_CENTER, "\U000F0596");}        
          if (id(weather_icon).state == "rainy") {
            it.printf(x, y, id(weather_font),  gray, TextAlign::TOP_CENTER, "\U000F0597");}
          if (id(weather_icon).state == "snowy") {
            it.printf(x, y, id(weather_font), gray, TextAlign::TOP_CENTER, "\U000F0F36");}
          if (id(weather_icon).state == "snowy-rainy") {
            it.printf(x, y, id(weather_font), gray, TextAlign::TOP_CENTER, "\U000F067F");}
          if (id(weather_icon).state == "sunny") {
            it.printf(x, y, id(weather_font), yellow, TextAlign::TOP_CENTER, "\U000F0599");}
          if (id(weather_icon).state == "windy") {
            it.printf(x, y, id(weather_font), gray, TextAlign::TOP_CENTER, "\U000F059D");}
          if (id(weather_icon).state == "windy-variant") {
            it.printf(x, y, id(weather_font), gray, TextAlign::TOP_CENTER, "\U000F059E");}
          if (id(weather_icon).state == "exceptional") {
            it.printf(x, y, id(weather_font), gray, TextAlign::TOP_CENTER, "\U000F0A39");}

          x = 65, y = 100;
          if (id(outside_temp_openweathermap).has_state()) {
            it.printf(x, y, id(font_e3), TextAlign::TOP_CENTER, "%.1f°", id(outside_temp_openweathermap).state);
          }

          x = 5, y = 157, z =80;
          it.printf(x, y, id(icon_font_16), ylnmn_blue, TextAlign::BOTTOM_LEFT, "\U000F1A80");
          it.printf(z, y, id(icon_font_16), ylnmn_blue, TextAlign::BOTTOM_LEFT, "\U000F058C");
          if (id(outside_humi_openweathermap).has_state()) {
            it.printf(x+20, y-2, id(font_e1), TextAlign::BOTTOM_LEFT, "%.0f%%", id(outside_humi_openweathermap).state);
          }
          if (id(outside_rain_openweathermap).has_state()) {
            it.printf(125, y-2, id(font_e1), TextAlign::BOTTOM_RIGHT, "%.0fmm", id(outside_rain_openweathermap).state);
          }

  - file: 'fonts/materialdesignicons-webfont.ttf'
    id: weather_font
    size: 96
    glyphs:
      - "\U000F0594" # clear-night
      - "\U000F0590" # cloudy
      - "\U000F0595" # partlycloudy
      - "\U000F0591" # fog      
      - "\U000F0592" # hail
      - "\U000F0593" # lightning
      - "\U000F067E" # lightning-rainy
      - "\U000F0596" # pouring
      - "\U000F0597" # rainy
      - "\U000F0F36" # snowy
      - "\U000F067F" # snowy-rainy
      - "\U000F0599" # sunny
      - "\U000F059D" # windy
      - "\U000F059E" # windy-variant
      - "\U000F0A39" # exceptional

  - file: 'fonts/materialdesignicons-webfont.ttf'
    id: icon_font_16
    size: 16
    glyphs:
      - "\U000F058C" # mdi-water
      - "\U000F1A80" # mdi-thermometer-water
```




