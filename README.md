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
> Show next page by press a button connect to GPIO25:   
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
  ......
    pages:
      - id: page1
        lambda: |-
          auto red = Color(255, 0, 0);
          auto lime = Color(0, 255, 0);
          auto blue = Color(0, 0, 255);
```
> Display time:   
```
time:
  - platform: homeassistant
    id: esptime
    timezone: Asia/Hong_Kong
    
display:
  ......
    pages:
      - id: page1
        lambda: |-
          // Green
          auto green = Color(0, 128, 0);
          it.strftime(x, y, id(font_d1), green, TextAlign::CENTER, "%I:%M", id(esptime).now()); 
          it.strftime(x, y, id(font_c1), green, TextAlign::TOP_LEFT, "%m月%d日", id(esptime).now());
    
```








