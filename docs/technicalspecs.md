---
layout: page 
title: Diedrichsen Lab Wiki
---

## Hand devices used for behavioural/imaging experiments

### Parts list for hand device
- Force transducers (Honeywell FSG15N1A) [spec sheet](https://www.digikey.ca/product-detail/en/honeywell-sensing-and-productivity-solutions/FSG15N1A/480-2621-5-ND/1248956)
- Plastic connectors for connecting force transducers outputs to amplifier (400 Series Buccaneer) [spec sheet](https://www.alliedelec.com/m/d/3b2e791d6f12f84357e18c8912681635.pdf). Wiring diagram for connectors can be found [here](/assets/fingerboxes/connectors.zip).

### Parts list for amplifier
- Printed circuit board for amplifier circuit. Connection diagrams and files for PCB fabrication can be found [here](/assets/fingerboxes/circuit_schematics.zip). 
- We also have a newer version of the amplifier circuit board with the the op-amps and the filter banks on the same circuit. The schematic files for the board can be found [here](/assets/fingerboxes/s978_files.zip). 
- Inputs from the force transducers are amplified and fed to a data acquisition card. We use a PCI sensoarray (S626), [spec sheet](http://www.sensoray.com/products/626.htm), card for this. The wiring diagram for connecting force transducer inputs to the breakout-box connecting to the S626 is shown below: ![stimulator box wiring](/assets/fingerboxes/stimBox_wiring.jpg)
- The amplifier uses 10 OpAmps part number AD627AN, datasheet [here](http://pdf1.alldatasheet.com/datasheet-pdf/view/48100/AD/AD627AN.html).
- In the backpart a 50 pin connector is attached to a ribon cable connector. the part number is RS403-342 datasheet [here](https://www.artisantg.com/TestMeasurement/81593-1/RS_403_342_50_Way_IDC_Header_DIN_Rail_Terminal).
- An extra National Instruments I/O card can be included, part number NI USB-6218 datasheet [here](http://www.ni.com/en-ca/support/model.usb-6218.html).


