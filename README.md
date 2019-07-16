# OVMS SWCAN add-on board

This add-on board for [OVMS (Open Vehicle Monitoring System)](https://www.openvehicles.com/) adds Single-Wire CAN (GMLAN) functionality to OVMS 3.0 module.

This board uses the same MCP2515 CAN controller as OVMS module, but instead of SN65 transceiver, it employs TH8056 which fully supports SWCAN functionality. While SN65 (and other CAN transceivers) can be used to receive/transmit CAN messages on SWCAN bus, they unfortunately are unable to wake the car with the so called high-voltage wake up (to raise CAN bus line to 12v). Thus OVMS hasn't been so useful with those cars with most of the functionality on SWCAN bus (such as GM/Chevrolet brands) so far. 

This add-on board however adds the full support for SWCAN communication and allows such remote functionality as locking/unlocking the car, cabinet/engine preheating, polling environmental variables (temperature) etc even when the car is asleep. The SWCAN circuitry is heavily inspired by same implementation found on [Macchina M2](https://www.macchina.cc/) interface board. However this board in designed connect directly to the OVMS extension header and to fit inside the existing plastic case on top of the optional GSM/GPS module.

In addition to SWCAN bus functionality, there's also 3 LEDS, various debugging test points and a 13 pin header to facilitate easier connection to logic analyzer. 

### Connection to SWCAN bus

To actually wire the SWCAN bus lines to the board, there are several ways. 
  1. There is 2-pin header on the board for SWCAN (SWCAN_LO is GND by standard) 
  2. The SWCAN_HI on this this header is connected to the transceiver as well as to GEP_1, so the pin 19 on DB25 extension port can also be used.
  3. Create OBD cable that connects the SWCAN bus to the unused pin 1 on OVMS DB9 port. Then solder a jumper wire from that pin1 to the GEP_1 on OVMS extension header. 
  4. Create OBD cable so that GND is connected to CAN2_LO, SWCAN_HI to CAN2_HI. Then jumper pin 16 (CAN2_HI) to pin 19 on DB25 (or use the dongle mentioned below). This actually then replaces the third CAN bus (CAN2) and the second internal MCP2515, which is put to sleep with modified OVMS code. This approach doesn't require internal modification of the OVMS module.

The cable I use with my Opel Ampera / Chevrolet Volt uses the approach #4:

| OBD-pin | DB9-pin | Comment |
| ------ | ------ | ------- |
| 1 | 8 | SWCAN_HI --> CAN2_HI  | 
| 4 | 6 | CHASSIS & SIGNAL GND --> CAN2_LO | 
| 4 | 3 | CHASSIS & SIGNAL GND | 
| 6 | 7 | CAN0_HI (Primary High-Speed CAN) | 
| 14 | 2 | CAN0_LO (Primary High-Speed CAN) | 
| 12 | 5 | CAN1_HI (Secondary High-Speed CAN) | 
| 13 | 4 | CAN1_LO (Secondary High-Speed CAN) | 
| 16 | 9 | +12V | 


### Technical details and debugging

The TH8056 transceiver modes (normal, sleep, high-voltage, high speed) are controlled either indirectly with MCP2515 controller (default setting, changable with M0_SEL and M1_SEL jumpers) or directly with MAX7317 EGPIO pins. This latter approach was a backup option in case using MCP2515 was finicky, but it seems this is not the case. So, the available MAX7317 pins can be used for other uses, for example test buttons connected to GEP_4 and GEP_3 (connect jumpers BTN1_SEL and BTN2_SEL in this case). 

In case you want to debug the SPI bus with all the CS lines from various peripherals connected at the same time, it's easier to solder jumper wires from the ESP32 to GEP #2-#4, which are then routed on the the add-on board to the 13 pin header. Then hooking/unhooking the logic analyzer wires becomes a breeze. Note that the CS line of MCP2515 on add-on board is wired already to the header, so no need for soldering if you need to debug only that.

For software, see [my GitHub repository](https://github.com/mjuhanne/Open-Vehicle-Monitoring-System-3) for support for this board, as well as upgraded functionality for Chevrolet Volt / Opel Ampera.


### Optional debugging dongle

Included in this repository is also an optional debugging dongle that contains following functionality:
   - Placeholders for 3 hardware buttons, of which the first two (BTN1 and BTN2) can be tailored in varied pull-up/pull-down/RC debouncing configurations. They can be polled with MAX7317. BTN3 is a pull-down button connected to EXP_1/SWCAN_INT that can induce interrupt if needed (for example to awake the OVMS from sleep state for testing purposes)
   - STATUS/TX/RX leds (wired in parallel with internal LEDS) so that they can be perceived also with the case on.
   - 12V / EXT_12V and 3.3V pins
   - CAN0, CAN1, CAN2 and SWCAN bus pins
   - By default the CAN2_HI is connected to SWCAN_HI to avoid internal jumper wires. This connection can be severed by breaking the SWC_SEL jumper
