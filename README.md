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
    device_height: 161
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
