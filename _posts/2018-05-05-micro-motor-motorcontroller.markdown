---
layout: post
title: µMotor
feature-img: "assets/images/micro-motor/umotor-pcb.jpg"
thumbnail: "assets/images/micro-motor/umotor-pcb.jpg"
tags: [elektronik, informatik, saison19]
author: Raphael Lehmann
excerpt: "Die Idee ist schon älter, aber nun haben wir uns an die Umsetzung gewagt:  \
Ein miniaturisierter Motorcontroller, welche dezentral direkt an jeden Motor verbaut wird..."
---

Dieser Blogpost ist veröffentlicht auf dem [Blog des Roboterclub Aachen e.V.](http://www.roboterclub.rwth-aachen.de/blog/2018/micro-motor-debugging-inbetriebnahme.html).

Im Rahmen des Roboterclubs hab ich mich an die Entwicklung eines miniaturisierter Motorcontroller gewagt, welche im folgenden beschrieben wird.

## Idee

Die Idee ist schon älter, aber nun haben wir uns an die Umsetzung gewagt:  
Ein miniaturisierter Motorcontroller, welche dezentral direkt an jeden Motor verbaut wird
und nur an Strom und CAN-Bus angeschlossen werden muss.

## Anforderungen

Die Anforderungen haben wir in den letzten Jahren gesammelt:
* ~~2 Phasen (nur DC Motoren) oder~~ 3 Phasen (DC und BLDC Motoren)
* ca. 10A pro Phase
* 20V Betriebsspannung (Akku) -> 40V oder 60V Auslegung
* Anschluss für Encoder mit AB(I)-Interface
* optional Magnetencoder auf dem PCB
* Anschluss für Hall-Sensor
* Temperatursensor (kontaktlos auf dem PCB verlötet oder I²C)
* Strommessung
* LEDs für Software-Status & Encoderstatus
* Konfigurierbare ID (ggf. STM32-Hardware-ID)
* Einfach lötbar und aufbaubar
* Abmessungen: Rund, Grund-Ø 20mm (Namiki-Motoren: Ø 22mm)

## Komponentenauswahl

![Er ist uns beim Waschen leider etwas eingegangen. Funktioniert aber trotzdem noch.](/assets/images/micro-motor/umotor-pcb.jpg)

Als nächsten Schritt mussten wir die Komponenten auswählen.
Die Wahl des Mikrocontrollers fiel schnell und wenig überraschen auf einen STM32, konkret einen [STM32L433CCU](http://www.st.com/en/microcontrollers/stm32l433cc.html) im 7x7mm²-UFQFPN48 Gehäuse.
Geeignete MOSFETs und MOSFET-Treiber auszuwählen gestaltete sich als deutlich komplizierter.
Eine große Herausforderung ist die kleine Baugröße von nur Ø 20mm.
Recht schnell war klar, dass dies nur realisierbar ist mit Komponenten welche Reflow-Löten erfordern,
somit waren DFN-Gehäuse nicht mehr wie bei früheren Projekten im RCA unerwünscht.

Das Anschluss ist ein 5-poliger Stecker mit CAN-Bus (2 Pins), Versorgungsspannung (12-40V), Logikspannung zu Versorgung des Controllers und natürlich Ground (GND).
Die Logikspannung ist von der Versorgungsspannung getrennt, da letztere möglicherweise durch Not-Aus-Schaltkreise deaktiviert werden kann, wobei die Controller aktiv bleiben sollen.

Schlussendlich haben wir nach wochenlanger Recherche den [DRV8323S](http://www.ti.com/product/drv8323) von Texas Intruments als MOSFET-Treiber ausgewählt.
Die entscheidenden Merkmale neben dem kompakten 6x6mm²-40WQFN Gehäuse die geringe Anzahl der benötigten externen Widerstände und Kondensatoren sowie das SPI-Interface über welches alle Parameter konfiguriert werden können.
Eines der Parameter ist der Gate-Drive-Strom, dadurch können Widerstände in den Gate-Leitungen zu den MOSFETs entfallen.
Ein weiterer Vorteil ist die Ladungspumpe zur Ansteuerung der High-Side-MOSFETs welche 100% PWM-Dutycycle erlaubt in Kontrast zu Bootstrap-Schaltungen bei Alternativkomponenten.
Mit Shunt-Widerstände in zwei der drei Motor-Phasen, einem Shunt-Widerstand welche den Gesamtstrom misst und den drei integrierten Strom-Messverstärkern des Gatetreibers wird die Strommesssung und eine Cycle-by-Cycle Strombegrenzung realisiert.

Die Wahl der MOSFETs fiel auf den Typ [FDMD8260L](http://www.onsemi.com/PowerSolutions/product.do?id=FDMD8260L):
* Zwei n-Channel MOSFETs im 3.3x5mm²-DFN12 Gehäuse
* V_DS 60V, R_DSon 5.8 mΩ
* I_D 40A (continuous max.)

Aus der Rückseite befindet sich der Magnetencoder [AS5045B](https://ams.com/AS5045) von AMS,
den wir vorher schon alleinstehend verwendet haben.
Falls der Motor einen Encoder mitbringt kann dieser Chip nicht bestückt werden und stattdessen der Motorencoder über Lötpads unterhalb des Magnetencoder-Footprints angelötet werden.
Für externe Motorencoder wurde zus#tzlich ein ansonsten nicht benötigter 5V-Spannungsregler vorgesehen.

Daneben sind auf der Platine nur ein CAN-Bus-Transceiver ([TCAN332](http://www.ti.com/product/tcan332)) und Spannungsregler verbaut,
welche aufgrund der kleinen Bauform ausgewählt wurden.

## Schaltplan und Layout

![Schaltplanübersicht micro-motor (µMotor) in KiCad](/assets/images/micro-motor/schematic-overview-screenshot.jpg)
![PCB Layout micro-motor (µMotor) in KiCad](/assets/images/micro-motor/pcb-layout-screenshot.jpg)

Aus den Anforderungen und Überlegungen sind in den vergangenen Monaten Schaltpläne und ein kompaktes Platinenlayout entstanden.
Das Platinenlayout ist 4-lagig ausgeführt, anders wäre die geforderte Bauform nicht umsetzbar gewesen.

## Aufbau und Inbetriebnahme

Der erste Aufbau mit Reflow-Ofen und die Inbetriebnahme gestalten sich als nicht trivial,
weil es auf der winzigen Platine kaum möglich ist elektrische oder Software- Probleme mit Multimeter, Oszilloskop oder Logicanalyser zu debuggen.
Einige Leitungen sind nur unter Bauteilen und auf den inneren Lagen geführt.

![micro-motor (µMotor) vor dem Reflow-Löten. Ein-Cent-Münze zum Größenvergleich.](/assets/images/micro-motor/pcb-assembled-before-reflow-onecent.jpg)
![micro-motor (µMotor) im Reflow Ofen](/assets/images/micro-motor/reflow-ofen.jpg)

Der erste und einzige aufgebaute Prototyp funktioniert weitgehend.
der bisher einzige Hardwarefehler konnte mit Hilfe von Fädeldraht und einem zusätzlichen Widerstand behoben werden.

## Software

Die Software auf dem Mikrocontroller wird mit [modm](http://modm.io/) entwickelt.
Bisher sind die Hardware-Abstraktion (Board-Support-Package) und einige Testprogramme für die Inbetriebnahme der Hardware fertiggestellt.

Die Hauptsoftware fehlt noch.

Auf dem Microcontroller sollen frei konfigurierbare Pid-Regler und ggf. auch "Motor-mit-Endschalter"-Komponenten,
welche den Aktor parametrierbar initialisieren und verfahren können, das Interface nach außen bilden.


Programiert werden muss der Micro-Motor über den 3D-gedruckten Programieradapter mit Federkontaktstiften.
Der Adapter verfügt außerdem über eine Pin zum abgreifen der UART-Ausgabe zu Debug-Zwecken.

![Programieradapter für den micro-motor (µMotor)](/assets/images/micro-motor/swd-adapter.jpg)

## Ausblick

In den nächsten Wochen werden wir die Hardware vollstandig testen, Software implementieren, Messungen vornehmen und Dokumentation schreiben.

Zeitnah werden wir den Quellcode, Schaltplan, Platinenlayout und mechanische CAD-Modelle unter einer OpenSource Lizenz veröffentlichen.

To be continued...

