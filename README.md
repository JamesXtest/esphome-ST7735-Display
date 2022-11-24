# esphome-ST7735-Display
esphome ST7735 Display  
Under construction, will be update soon...  

Draft :

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
    glyphs: 年月日時分秒間一二三四五六七八九十百千萬室內外溫濕光度空氣質素香港電台第μ³!"%()+,-_.:°/0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz
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
          it.strftime(x, y, id(font_d1), green, TextAlign::CENTER, "%I:%M", id(esptime).now()); 
          it.strftime(x, y, id(font_c1), green, TextAlign::TOP_LEFT, "%m月%d日", id(esptime).now());
    
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







