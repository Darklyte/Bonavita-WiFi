I Added Wifi to my Bonavita Kettle


This morning when, I picked up my phone and began browsing reddit while the drowsiness wore off. After 5 minutes on my phone, my white-noise generator shut off, knowing I was actually getting trying to wake up. 10 minutes later, the blinds in my room opened, and my bonavita kettle began preheating my water to 210ºF. 5 minutes later, I got up, did my morning routine, and headed out of my bedroom, triggering my coffee grinder to grind 24g of beans for cup of coffee I was about to make.  

This is done using Home Assistant and ESPHome. 

## ["SHUT UP AND GET TO THE POINT!"](https://www.youtube.com/watch?v=3dwbkuEiDSw)

### Tools

* Soldering Iron
* Solder
* Wire cutters
* Voltometer

### Prototyping

* Breadboard
* Dupont cables
* LEDs

### Consumables
* [ESP8266-wifi board](https://www.amazon.com//dp/B08N5WTQ4Q/) 
* 2-6 [PC817 optoisolators](https://www.ebay.com/sch/i.html?_from=R40&_trksid=m570.l1313&_nkw=PC817&_sacat=0) (2 minimum, 3 recommended)
* 1 decent-quality USB wall charger
* 1 wall power cable that you don't mind destroying (optional)
* 1 USB-Micro cable
* 1 Tri-wing screwdriver ([Recommended multipurpose option](https://www.amazon.com/gp/product/B0925G9FFH))
* 1 ~127 Ohm resistor 
* A small Prototype board or breadboard ([Recommended](https://www.amazon.com/dp/B081MSKJJX)
* [Assorted lengths of wire](https://i.imgur.com/83lc2kr.jpg) (solid core recommended)

[Circuit Diagram](https://github.com/Darklyte/Bonavita-WiFi/blob/main/Bonavita%20Circuit%20Diagram.png)

## Background

A few days ago there was a discussion about [automating the bonavita kettle](https://redd.it/t32q3i) and how the bonavita wasn't worth automating because it had low watt output.  After Buying a Fellow Stagg EKG+ with bluetooth, I discovered my 1.7L bonavita is actually 1500W and more worth automating that figuring out how to [hijack the Stagg's bluetooth signals](https://redd.it/ek47k2). 

I knew all I needed to do was somehow bridge the circuit between the switches. I figured an esp32 could do this and this would be my first project with one. Going in, I had no idea how they worked and so learning about them took a fair bit of time. I want to write something detailed enough that a beginner could understand it.

During the process, I came across [this post on home-barista.com](https://www.home-barista.com/brewing/adding-wifi-to-bonavita-digital-kettle-t40682.html) by luma, where they had done the same thing. I reached out to them with some questions and they were extremely helpful, introducing me to new electronic components that I didn't know existed and providing advice along the way.


## Safety

If you are going to do this, please be as safe as possible. You will be using a soldering iron, which can potentially reach over 480ºC (900ºF) and will burn you instantly. You will be working with both high and low voltage. The high voltage ** will kill you. ** Wires and cutters are sharp. You'll potentially have water in the area with electricity, which is another hazard.

* If you are working on the circuits, make sure everything is unplugged and turned off.
* Keep the kettle away from exposed circuits
* Keep your hands away from the hot end of the soldering iron
* Be careful about sharp things!

In addition, take the time to test every step of the way to avoid backtracking. Every time you double check you reduce your chance of making a mistake and not knowing where it is.  This project shouldn't take more than a few hours, but even after I had all of my parts it took me probably FIFTY HOURS because I kept having to backtrack and fix issues.

## The Components

### Home Assistant and ESPHome

I've had a lot of issues getting my computer to read the esp8266 boards. There are a lot of steps like installing drivers, getting the right port, installing board settings, downloading arduino... ESPHome makes things much easier. It helps you get things done very quickly and uses yaml to program, rather than the arduino IDE.

It is available through [Home Assistant](https://www.home-assistant.io/) which is my preferred Home Automation platform. I will assume you use home assistant, but that you've never used ESPHome before.  Setting it up explained in the guide later.

### ESP8266 NodeMCU

I had a few [NodeMCU boards](https://www.amazon.com//dp/B08N5WTQ4Q/) on hand from the last time I considered trying this. Pretty much any esp board will work, as long as it has at least two pins you can use as events and one as data.

The board takes 5V to run. Some of them can take up to 12V and regulate it down to the voltage it needs. The control board on the bonavita inputs 5.5V, but wiring the ESP8266 directly do it only provided 2V. I don't know why, so ultimately I ended up using a USB brick to power the board like luma did.

The [GPIO Pins](https://en.wikipedia.org/wiki/General-purpose_input/output) output 3.3V when they are on (or 'the signal is high' as it is often referred). The bonavita circuit uses 5.5V, and the 3.3V is not enough to trigger the circuits on the board. Incidentally, if the ESP8266 were to receive 5.5V unregulated into its ground rather than the regulated VIN pin, it would likely fry the board.

Getting the ESP8266 started has always been complicated for me. I'll explain how to do it properly in the setup steps.

### Optoisolators (PC817)

[luma](/u/svideo) told me about Optoisolators. I had never heard of them before and honestly they are absolutely amazing. You wire them to two completely isolated circuits. They do not allow the circuits to cross, but they take information from one circuit and act as a switch for the other.

Here is the [PC817 Pinout](https://github.com/Darklyte/Bonavita-WiFi/blob/main/PC817-Pinout-1.png?)^[Source](https://www.theengineeringprojects.com/2017/07/introduction-to-pc817.html). Note that this image shows what each pin is relevant to the text on the chip. Pin 1 will be your input signal (3.3v from the ESP8266, or the 5.5V from the Bonavita board). Pin-2 goes to GND.

[Here is a diagram of how they work](https://github.com/Darklyte/Bonavita-WiFi/blob/main/PC817%20Diode%20bridge%20working.gif)^[Source](https://microcontrollerslab.com/pc817-optocoupler-pinout-working-examples-datasheet/). They grounds and power sources should be isolated, each circuit having their own. The optoisolator take the input signal from the control circuit (left side) and cause an inverse signal in the receiving circuit (right side). When the control circuit is high, the receiving circuit is low.

On the receiving circuit side, however, you can reverse the signal. If you wire Pin 4 to ground and Pin 3 to signal, the receiving circuit will output will be the same as the controlling circuit (high when high, low when low).

(Note:  I believe I did this accidentally. in luma's notes he points out that the LED is off when it is receiving a signal. In my initial attempt I did have to invert the binary sensor, but afterwards I did not.)


### Bonavita's Electronics

[Here is a control board for the bonavita](https://github.com/Darklyte/Bonavita-WiFi/blob/main/Bonavita%20Circuit%20Markup.png). Specifically, this is the back of the board where the buttons are. 

* **Ground (Black)**: The entire lower section is ground. You can connect everything that is returning to ground to one solder point, or if you want you can break it up, but ultimately every point on the gold section is the same point.

* **Power Button (Orange)**: These are the positive terminals on the power button, meaning they have a high (5.5v) signal. When the button is pressed, they connect the circuit to the ground point allowing the signal to pass through. Closing this circuit in any way is the same as pressing the On/Off button on the bonavita.

* **Power LED (Red)**: This is the positive terminals on the power LED. The LED is strangely off when this is high (5.5v). Turning on the power changes it to lower and causes the LED to turn on. Reading this will let us know if the kettle is on or not.

* **Hold Button (Yellow)**: Completing this circuit will cause the bonavita to hold the temperature for 1 hour, or until the kettle is lifted off the base. This only works if the kettle is active and no other buttons are being pressed, so we have to make sure the power button isn't being pressed when we have the esp8266 press this button.

* **Hold LED (Green)**: I did not wire this, but I thought I'd highlight it for anyone that wants it. You can check the hold state by reading this LED. I assume it functions the same way as the power LED.


[A note about buttons](https://i.imgur.com/6nhCtqE.gif)^[Source](https://components101.com/switches/push-button): The buttons have four pins, but they are paired. Pressing the button bridges each positive pin to ground. A single button can trigger two things, technically, but these do not.

### USB Wall charger

I thought I could power through ESP8266 through the 5.5V input that the bonavita's control board takes, but the voltage dropped too much when I added the board. Instead, I took an old USB wall charger and removed the case. I cut the ends off of an old AC cable and solders one end to the inputs (it doesn't matter which wire is which since it is AC) and the other ends the main AC input on the bonavita. 

** I cannot express how important it is that you make sure your bonavita is unplugged while you are adding this. This is high voltage main input.  It will kill you. **

### Breadboard or Prototype board

You should use a breadboard for testing, and then switch everything to a Prototype board (ideally a printed circuit board, but who has the time/money for that?). 

When you [look at a breadboard](https://github.com/Darklyte/Bonavita-WiFi/blob/main/13217.Jpg), You'll notice it has deep channel dividing it in half. It then has rows of holes. The rows are connected together on the bottom of the board, and the channel divides them in half. For example, on this board Pin 1A connects to 1B, 1C, 1D, and 1E, but not to any other pins.


## Building the circuit.

[The circuit diagram is here](https://github.com/Darklyte/Bonavita-WiFi/blob/main/Bonavita%20Circuit%20Diagram.png), but I want to make it as readable as possible for those beginning. [Here is an edited version of the circuit with physical devices](https://github.com/Darklyte/Bonavita-WiFi/blob/main/Physical%20Circuit.png).

The black lines are wires. When wires cross path, there is a dot if they are connect. If there is no dot and it isn't touching a component, the wires are not connected.

Things to note:

* The third PC817 is rotated 180º. It uses the bonavita as its controlling circuit and the ESP8266 is the receiving circuit.
* The bottom pins on the ESP8266 side are wired to ground. The top pins on the ESP8266 are wired to D1, D2, and D5 respectively. (I'll explain why later)
* The top pins Bonavita side are wired to ground. 
* The other pins are wired to the power buttons's signal (5.5v), the hold buttons signal, and the power LEDs signal.
* There is a 127 ohm resistor wired on the bonavita side of the power LED, between the top pin and ground. This, is to ensure the powers flows through the LED instead of the non-resistant PC817. (electricity prefers the path of least resistance)

## Actual Setup Guide

### Prepare the ESP8266 with Home Assistant and ESPHome

In your Home Assistant Instance, Go to Configuration -> Add-ons, Backups & Supervisor -> Add-on Store (lower right). Search for ESPHome. There is is no configuration to worry about. Simply click `Install`. It will take a few minutes to install then a few minutes to boot up. Click `Open Web UI`. If you get a `Bad Gateway` error, it hasn't finished booting up.  Wait a minute, go back, then try agian.

Plug your ESP8266 via USB directly into the device running Home Assistant. Once you get the board connected to wifi, you won't have to keep it connected to your Home Assistant device. ESPHome will pretty much guide you through initial setup.  Select your board and give the device a name (I called mine bonavita) and allow it to do the initial install. Verify that it is able to connect to your network and that ESPHome can see it. 

If ESPHome cannot connect to it through `bonavita`.local (or `whatever-you-named-it`.local), you may need to specify an address. Edit the config and change the wifi section to look something like this:

```
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  // Specify a static address that is available so you'll know where it will be 
  use_address: 192.168.1.180
  # Optional manual IP
  manual_ip:
	// The same address specified above
    static_ip: 192.168.1.180 	
	// Your router's address
    gateway: 192.168.1.1
	// Your subnet mask. This is probably correct.
    subnet: 255.255.255.0
```

Install your new config. If that works, great! We're done here for now. You can unplug the board from your home assistant device. Add the following code at the bottom of the config, then install and make sure it still connects.

```
switch:
  - platform: template
    name: "[Switch] Bonavita Kettle"
    lambda: |-
      if (id(status).state) {
        return true;
      } else {
        return false;
      }
    turn_on_action:
      then:
        if:
          condition:
            binary_sensor.is_off: status
          then:
            - switch.turn_on: power
            - delay: 500ms
            - switch.turn_on: hold
    turn_off_action: 
      then:
        if:
          condition:
            binary_sensor.is_on: status
          then:
            - switch.turn_on: power
  - platform: gpio
    id: power
    internal: true
    on_turn_on:
    - delay: 250ms
    - switch.turn_off: power
    pin: 
      number: D1
    name: "Bonavita Power"
  - platform: gpio
    id: hold
    internal: true
    on_turn_on:
    - delay: 250ms
    - switch.turn_off: hold
    pin: 
      number: D2
    name: "Bonavita Hold Temperature"
    
binary_sensor:
  - platform: gpio
    id: status
    pin: 
      number: D5
      mode:
        input: true
        pullup: true
      inverted: false
    name: "Bonavita Status"
```

Once this works, go back to Home Assistant, and go to Configuation -> Devices & Services > Add Integration (Bottom Right). Search for `ESPHome` and add it.  Give it the IP address or identifier of your bonavita. It should add the device and entities.

### Preparing the Bonavita

Unplug your bonavita and empty the kettle, leaving maybe 300ml to absorb heat when testing. With a tri-wing screwdriver, remove the [5 outer-most screws on the bottom (two are recessed near the front. Ignore the three in the middle)](https://github.com/Darklyte/Bonavita-WiFi/blob/main/PXL_20220308_192411538.MP.jpg). Pull the top and bottom apart, pushing the outlet cord through to give slack to allow the bottom to fall away more.

Solder a wire length to a ground point. Solder one length to each of the, Power Button, and Hold button points. Optionally, solder an additional lengths to the Temp+, and Temp- points. 

Solder another length of wire to the Power LED. Be especially careful with the LEDs as their ground solder points are very close. Accidentally soldering those two points together will make the LEDs not work, but you can clean it off.  If you wish, do the same for the Hold LED.

### Prepare the ESP8266 Wiring

[How to Remove Header Pins](https://www.youtube.com/watch?v=MRpF1IlzVtw) (if you want)

Solder a length of wire each to the D1, D2, and D5 pins. D3, D4 and D8 are used on startup to determine the boot mode, therefore these pins should not be used for LED monitoring. You can, however, still use them for the switches. 

Choose a GND and solder a wire to that as well. If you are going to add the other LEDs and buttons, connect them to D6, D7, and one of the `OUTPUT ONLY` pins above. 

Be careful and ensure that none of your solder points are touching other pins.


### Prepare the Breadboard

We'll solder the ESP8266 wires to the left side, and the bonavita to the right.

Place the three PC817's crossing the central channel on the breadboard. The third should be rotated 180º.

[IMAGE]

Take your ground wire and solder it the bottom-left pin on the first PC817. Connect this row to bottom-left pin of the second and third PC817 so that the lowest pin on the left side of each PC817 is grounded on this side.

Connect D1 to the top-left pin of the first PC817.

Connect D2 to the top-left pin of the second PC817.

Connect D6 to the top-left pin on the third PC817.


#### Bonavita Side

Connect your ~127 ohm resistor between the top-right pin on the lowest PC817. Solder the other end into an empty row. We'll call this "resistor row"

Connect  the ground wire from the Bonavita to the top-right pin on the first PC817. Connect this row to the top-right  pin of the second PC817 and to the resistor row. Each PC817 should be grounded on this side, the the bottom-most one having to pass through the resistor to get to ground. 

Connect the Power Button to the bottom-right pin on the first PC817

Connect the Hold Button to the bottom-right pin on the second PC817

Connect the Power LED to the bottom-right pin on the third PC817

**Test**

Plug the USB cable into the ESP8266. Make sure it boots properly. You can check by checking connecting to the logs wirelessly in ESPHome. 

Once it connects, plug in the Bonavita. **BE VERY CAREFUL AS THE BONAVITA IS OPEN, EXPOSING WIRES**. The Power sensor should change from `ON` to `OFF`. If it does, then the power sensor is working!

In Home Assistant, go to Configuation -> Devices & Services. Click on the bonavita device. The entities should show their status. Toggle the power. The Status should change from `OFF` to `ON`. If it does not, carefully move the Bonavita's control board so you can see the display.  It should be lit up. If it is, then the button worked! If not, check your wiring.

Now we will check the hold button.  Unplug the esp8266 and the bonavita. Carefully place the Bonavita's control board back in place. Ensure the part that connects to the kettle is reasonably in place, and place the bottom back on the bonavita. You want your wiring to be hanging out since we aren't putting everything together yet. flip the bonavita base so it is right-side up, and make sure the kettle connector is perfectly in place. The base should fit together about perfectly, and this is necessary as the hold button can only be used while the kettle is on. Put the kettle on top of the base and plug everything back up. Again, be very careful. 

in ESPHome, make sure it is able to connect to the ESP8266. Once it does, go to Home Assistant and turn on the bonavita. The kettle should turn on, flashing the selected temperature, and both the power and hold LED should turn on. If the display shows only `00 00` or similar, the kettle is not completely set on the base. If both the power and hold LED are on, everything works!

## Getting it self-contained

** THIS PART IS EXTREMELY DANGEROUS **

Actually, this part is so dangerous I don't really feel comfortable instructing how to do it. luma did it [here](https://www.home-barista.com/brewing/adding-wifi-to-bonavita-digital-kettle-t40682.html) and I did the same thing he did, almost exactly. 

## Finishing up

Once everything is done, you can wrap all the parts in electrical tape to prevent them from shorting. Stuff all the parts into the ample space of the bonavita base, then seal it up.  Enjoy your wifi kettle. 

