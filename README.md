# VotronicMPPT2HomeAssistant
Votronic Mppt Integration for Home Assistant with ESP32

# Projektübersicht
Dieses Projekt dokumentiert die Integration eines Votronic MPPT Solarladereglers über die Displayschnittstelle in ein ESPHome-gestütztes Home Assistant System. Ziel ist es, die relevanten Daten (Batteriespannung, Solarspannung, Solarstrom und daraus berechnete Leistung und Energieertrag) zuverlässig über UART auszulesen und für Home Assistant bereitzustellen. 

# HINWEIS 1: 
Für dieses Projekt wird die Displayschnittstelle des Votronic verwendet. Ein ggf. vorhandenes Display kann wahrscheinlich nicht weiterverwendet werden
# HINWEIS 2: 
für dieses Projekt erfolgte kein Sponsering o.ä., noch sind die hier aufgeführten Links Affiliate sondern dienen rein zu Informationszwecken. Ich verdiene hier kein Geld! 
# PRINTABLES.COM: 
Nicht alle 3D-Druckteile wurden von mir selbst entworfen, Links zu Printables könnten sowohl auf eigene als auch auf fremde Designs verweisen. „DoKa“  ist meines, der Rest ist fremd. Die Nutzungsbedingungen der 3d-Modelle sind zu beachten.
# ACHTUNG: 
Nachahmungstätern sei gesagt, dass sie auf eigene Verantwortung handeln. Ich habe grundsätzlich keine Ahnung was ich tue, alle Erkenntnisse sind ergoogelt und mit Hilfe von ChatGPT ergänzt. Ich übernehme keine Haftung für Schäden jeglicher Art, egal ob Brände, Nuklearkatastrophen oder Hirnexplosionen, die durch nachbauen meiner Anleitung entstehen.

# Verwendete Hardware
- ESP32-S2 Mini  (ein erster Test auf einem ESP-WROOM-32 lief nicht zuverlässig, ergründen kann ich das aus Mangel an Verständnis nicht) 
- Votronic MPP Solar-Laderegler mit RJ12 Display-Schnittstelle (UART, 1000 Baud)
- UART-Verbindung via GPIO17 (RX)
- 3D-gedrucktes Gehäuse , eigene Lötkabelverbindung (nicht Teil dieses Dokuments)

# Adapter und Info zu RJ12
Für den Adapter verwende ich einen RJ12 Stecker auf 6 Pins Schraubklemmenblock Adapter .
Bei Unsicherheit bzgl. des GND kann die Masse zwischen z.b. Batterie (-) und GND auf Durchgang geprüft werden.
GND (Pin1) wird mit GND des ESP verbunden, Data (Pin6) geht auf GPIO17.

# Funktionen
- Auslesen der Rohdatenpakete vom Votronic-Laderegler
- Prüfung der Checksumme zur Gültigkeit des Datenpakets
- Umrechnung der Rohdaten in reale Messwerte
- Berechnung:
  - Solarleistung (Watt)
  - Energieertrag in Wh und Ah (täglich + historisch)
- Automatische Rücksetzung auf 0, wenn keine gültigen Daten mehr empfangen werden
- Home Assistant Integration via API (verschlüsselt)

# Kommunikation mit dem Laderegler
Der Votronic MPPT Regler sendet regelmäßig 16-Byte-Datenpakete via UART. Das erste Byte des Pakets ist immer 0xAA (Start-Byte). Die Prüfsumme zur Validierung des Pakets ergibt sich durch XOR aller vorherigen Bytes des Pakets, exklusive des Start-Bytes (0xAA).
Das bedeutet: Von Byte 1 bis Byte 14 wird ein bitweises XOR durchgeführt, das Ergebnis muss mit Byte 15 (der Checksumme) übereinstimmen.
Beispiel für gültiges Paket:
[0xAA] [0x1A] [0x3B] [0x05] [0xC8] [0x05] ... [0x00] [0xXX]
C++ Code zur Berechnung:
uint8_t checksum = 0;
for (int i = 1; i < 15; i++) checksum ^= data[i];

# Hinweise zum Einsatz
- Bei Ausfall der Sonne / Winterlager werden 0,0 W / 0,0 V angezeigt.
- Die Energiewerte (Ah, Wh) werden dauerhaft mit restore: true gespeichert.
- Der Votronic MPPT Laderegler liefert nur Daten, wenn eine Solarspannung größer ca. 14 V anliegt.
- In Home Assistant kann auf Basis der Sensoren ein eigenes Energiedashboard erstellt werden.
- Bei fehlender Datenübertragung zeigt das Log:

[E][uart:015]: Reading from UART timed out at byte 0!



