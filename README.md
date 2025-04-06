
# Votronic MPPT Home Assistant Integration mit ESPHome

> by Manuel Laupp & ChatGPT, April 2025  
> *DIY-Projekt zur Anbindung eines Votronic MPPT-Solarladereglers an Home Assistant via ESPHome.*
>
> for english look @README_en.MD (translated by ChatGPT)

---

## 📦 Projektübersicht

Dieses Projekt dokumentiert die Integration eines **Votronic MPP Solar-Ladereglers** über dessen **Displayschnittstelle** in ein ESPHome-basiertes Home Assistant System. Ziel ist es, **Batteriespannung, Solarspannung und Solarstrom** auszulesen und daraus **Leistung und Energieertrag** zu berechnen.

---

## ⚠️ Hinweise

- 🧠 **Kein Sponsoring** – keine Affiliate-Links, keine Werbung
- 📎 Verwendete 3D-Druckteile stammen teils von externen Quellen (z. B. Printables.com)
- 💥 Nachbauen auf eigene Gefahr – keine Haftung für Schäden oder hirntechnische Überlastungen
- 🖥️ Es wird die **RJ12 Displayschnittstelle** des Reglers verwendet – ein Originaldisplay kann *nicht gleichzeitig* betrieben werden.

---

## 🛠️ Verwendete Hardware

- **ESP32-S2 Mini**
  - (Hinweis: WROOM32 funktionierte im Test instabil)
- **Votronic MPPT Solarregler** mit RJ12-Schnittstelle (UART 1000 baud)
- UART-Verbindung: **RX auf GPIO17**, GND auf GND
- RJ12-Adapter auf Schraubklemmen (z. B. von AZ-Delivery)
- Eigenes 3D-Gehäuse für Montage (optional)

---

## 🔌 Anschlussbelegung RJ12

```
PIN 1: GND → ESP GND  
PIN 6: DATA → ESP RX (GPIO17)
```

> ✅ GND sicherheitshalber mit Multimeter gegen Batterie-Pol (-) prüfen

<img width="95" alt="Votronic_RJ12" src="https://github.com/user-attachments/assets/7975447d-dcf5-4fe4-bb4c-20b1bbd0a9e3" />


---

## 📡 Funktionen

- Empfang & Auswertung der **UART-Datenpakete**
- **Checksummenvalidierung** (XOR-Check)
- Umrechnung in:
  - Batteriespannung [V]
  - Solarspannung [V]
  - Solarstrom [A]
  - Leistung [W]
  - Energieertrag [Wh / Ah]
- Automatisches Setzen auf 0 bei Inaktivität
- Integration in Home Assistant (verschlüsselt)

---

## 🔁 Kommunikation mit dem Regler

Der Votronic MPPT sendet alle paar Sekunden ein 16-Byte-Datenpaket via UART.

### Aufbau des Pakets

- **Startbyte:** `0xAA`
- **Länge:** 16 Byte
- **Checksumme:** XOR von Byte 1–14 muss mit Byte 15 übereinstimmen

### Beispielpaket

```
[0xAA] [0x1A] [0x3B] [0x05] [0xC8] [0x05] ... [0x00] [0xXX]
```

### Checksumme berechnen

```cpp
uint8_t checksum = 0;
for (int i = 1; i < 15; i++) {
  checksum ^= data[i];
}
```

Nur bei **gültiger Prüfsumme** werden die Daten weiterverarbeitet.

---

## 🧠 Verhalten bei Sonnenausfall / Inaktivität

- Bei fehlender Sonneneinstrahlung oder Nachtbetrieb → automatische Anzeige von `0.0`
- Wird keine Datenübertragung empfangen, zeigt das ESPHome-Log:

```
[E][uart:015]: Reading from UART timed out at byte 0!
```

- Alle Messdaten bleiben in Home Assistant erhalten, sofern `restore: true` aktiviert ist

---

## 📊 Integration in Home Assistant

Alle Sensoren können direkt im Energy-Dashboard verwendet werden (z. B. `solar_energy_wh`, `solar_power_calc`, etc.).  
Durch Kombination mit anderen Victron-Komponenten (z. B. SmartShunt, VE.Direct) lassen sich vollständige Energieflüsse darstellen.

---

## 📄 YAML-Konfiguration

Die vollständige YAML-Datei findest du in diesem Repository unter [`votronic-mppt.yaml`](https://github.com/Doka85/VotronicMPPT2HomeAssistant/blob/main/votronic_mppt.yaml).  
Bitte beachte: **Passwörter und Keys sind in dieser Version aus Sicherheitsgründen entfernt.**

---

## 📷 Weitere Infos & Bilder

- Fotos des Einbaus
- Pinout-Grafiken
- 3D-Druck-Vorlagen

> 🔜 folgen bald!

---

## 🧪 Projektstatus

- ✅ Prototyp erfolgreich im Einsatz
- 📈 Langzeittest läuft seit April 2025

---

## ✍️ Lizenz & Haftungsausschluss

Dieses Projekt steht unter der MIT-Lizenz.  
Verwendung auf eigene Gefahr. Keine Haftung für Schäden oder defekte Hardware.

---

## 🤝 Mitwirkende

- **Manuel Laupp** – Idee, Umsetzung, Dokumentation  
- **ChatGPT** – Code- und YAML-Unterstützung, Dokumentationshilfe

---
