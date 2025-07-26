---
layout: post
title: "Tuning of the SimplIQ Solo Whistle 2.5-60I Servo Drive"
date: 2025-07-24 00:00:00 -0400
---

The [SimplIQ Solo Whistle](https://www.elmomc.com/product/solo-whistle/) by [Elmo Motion Control](https://www.elmomc.com/) is a lightweight and highly compact servo drive powered by a DC source, suitable for DC brush and brushless, sinusoidal and
trapezoidal motors.

<img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/elmo-servo-drive.png" width="300" style="display: block; margin: 0 auto;"/>


The block diagram below shows the architecture of the servo drive which features a highly configurable controller, RS-232 serial communication and [CANopen](https://en.wikipedia.org/wiki/CANopen) for fast communication in a multi-axis distributed environment, fault protection, and several feedback options including analog encoder.

<img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/elmo-servo-drive-architecture.png" width="500" style="display: block; margin: 0 auto;"/>

The AMALIA rovers 2.0, 2.1, and 3.0, developed by [Team DIANA]({{ site.baseurl }}{% link _projects/2022-07-21-diana.md %}), employed the SimplIQ Solo Whistle servo drives  to control the four in-wheel custom sinusoidal brushless motors featuring 11 pole pairs and an external rotor configuration designed and manufactured by the [Technai Team](https://www.technai.it/) {% cite marenco2014elettronica %}. The two images below show the elastic wheel of the *AMALIA* rover 2.1 during a test at the [ALTEC Mars Terrain Simulator](https://ieeexplore.ieee.org/document/8869653) (left) and a CAD cross-sectional rendering highlighting several components, including the optical disc and encoder (right).

<img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/amalia-wheel-2.1.png" width="700" style="display: block; margin: 0 auto;"/>

During my [PhD]({{ site.baseurl }}{% link _projects/2022-07-24-phd.md %}), I tuned the SimplIQ Solo Whistle servo drive, using the [Elmo Application Studio](https://www.elmomc.com/product/easii/) software, as part of the preliminary experimental campaign to assess the tractive performance of the elastic wheel of the AMALIA rover. Later, to test the wheel under higher loads and slip ratios, I replaced the native in-wheel motor with a more powerful [Maxon brushless EC 60 motor](https://www.maxongroup.com/medias/sys_master/root/8831018893342/2018EN-270.pdf) controlled via the open source [VESC](https://vedder.se/2015/01/vesc-open-source-esc/) platform.


In this post I would like to briefly describe the tuning procedure for the SimplIQ Solo Whistle drive, which I learned while working with Team DIANA, thanks to the guidance of my talented and patient teammates.

The laboratory setup for tuning the servo drive includes the following components, also displayed in the picture below:
* DC power supply.
* Laptop running the Elmo Application Studio software.
* Wheel rim containing the motor and mounted on a custom-made metal frame.
* Arduino UNO microcontroller board and a solderless breadboard to capture motor temperature data.
* USB-to-RS232 adapter enabling the communication between the servo drive and the laptop.
* Servo drive.

<img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/setup.png" width="700" style="display: block; margin: 0 auto;"/>

The picture and rendering {% cite marenco2014elettronica %} below illustrate the wire connections, in particular:
* The three motor phases are connected to pins 1.b, 1.c, and 1.d in arbitrary order, as the tuning software automatically detects the correct sequence.
* The DC power supply is connected via connector 2.
* The encoder is connected via connector 3.
* Serial communication is connected through connector 5: pin 5.a carries the RX signal, 5.b the TX signal, and 5.c the ground

<img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/wire-connections.png" width="700" style="display: block; margin: 0 auto;"/>

The tuning procedure consists of five main steps: current gain tuning, commutation, digital inputs and outputs, velocity gain tuning and position tuning. The full tuning procedure is presented below through screenshots of the Elmo Application Studio wizard, together with brief descriptions. Moreover, a [video](https://youtu.be/DWn1CmHsZuc) of the procedure is available on my YouTube channel; mid-video,  whistles and tones are clearly audible as the drive injects test signals to identify resonances.

1. Enter the application name and select the communication type (RS-232).
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/1.png" width="700" style="display: block; margin: 0 auto;"/>

2. Enter the motor name, type and parameters.
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/2.png" width="700" style="display: block; margin: 0 auto;"/>

3. Enter the commutation feedback (encoder).
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/3.png" width="700" style="display: block; margin: 0 auto;"/>

4. Review/enter the driver parameters.
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/4.png" width="700" style="display: block; margin: 0 auto;"/>

5. Select the tuning steps to be performed. For this application, the position tuning was not required.
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/7.png" width="700" style="display: block; margin: 0 auto;"/>

6. Run the tuning current loop.
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/8.png" width="700" style="display: block; margin: 0 auto;"/>
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/8_3.png" width="700" style="display: block; margin: 0 auto;"/>

7. Run the establishing commutation process.
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/9.png" width="700" style="display: block; margin: 0 auto;"/>
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/9_1.png" width="700" style="display: block; margin: 0 auto;"/>
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/9_2.png" width="700" style="display: block; margin: 0 auto;"/>
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/9_5.png" width="700" style="display: block; margin: 0 auto;"/>

8. Run the tuning velocity loop. Response is set to *Slow and Stable* and Noise to *Slow and Quiet*.
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/10.png" width="700" style="display: block; margin: 0 auto;"/>
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/10_1.png" width="700" style="display: block; margin: 0 auto;"/>
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/11.png" width="700" style="display: block; margin: 0 auto;"/>

9. At the end of the tuning procedure a summary of the tuned parameters is displayed.
   <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/12.png" width="700" style="display: block; margin: 0 auto;"/>

10. Using the *Smart Terminal* tool, it is possible to test the tuned servo drive by setting the desired rotational velocity.
  <img src="/img/posts/2025-07-24-elmo-servo-drive-tuning/13.png" width="700" style="display: block; margin: 0 auto;"/>

During the tuning procedure, some common errors may occur. Below are the most frequent ones and their solutions:
* **Can not identify num/dem controller**: ensure the encoder wires are securely connected to connector 3.
* **Please check the encoder connections**: verify the correct wiring order of the encoder.
* **Failure – can't measure motor current**: check whether the motor phase wires have come loose.

## References
{% bibliography --cited %}