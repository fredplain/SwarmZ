# 🚢 Câblage Raspberry Pi ↔ Cube Orange
### Guide de câblage UART — Battleboats 2026 

> Guide visuel étape par étape pour connecter le Raspberry Pi au Cube Orange (contrôleur de vol ArduPilot), le récepteur radio via un encodeur PPM, les servos et le BEC — à destination des débutants.

**Pour plus de détails, téléchargez les sources et ouvrez le fichier html (avec images, notamment des connecteurs).**
---

## 🧩 Les composants

| Composant | Rôle |
|-----------|------|
| 🍓 **Raspberry Pi 4B** | L'ordinateur principal. Calcule la trajectoire et envoie des ordres au Cube Orange via l'UART. |
| 🟠 **Cube Orange** | Le contrôleur de vol (autopilote). Reçoit les ordres du Pi et commande les servos avec précision. |
| 📻 **Récepteur RC (Joysway J5C01R)** | Reçoit les signaux radio de l'émetteur Joysway J4C05. Sort uniquement des signaux **PWM individuels** par canal (pas de SBUS ni de PPM natif). |
| 🔄 **Encodeur PPM** (ATMEGA328p, firmware Paparazzi) | Convertit les signaux **PWM** du J5C01R en un signal **PPM** unique compatible avec l'entrée RCIN du Cube. Alimenté directement par le Cube via le port RCIN (cavalier non soudé par défaut). |
| ⚙️ **Servos** | Les moteurs de précision qui bougent le gouvernail et la voile. Reçoivent leur signal du Cube Orange. |
| 🔋 **BEC 5V/3A** | Alimente le rail servo du Cube Orange (les sorties MAIN OUT ne sont pas alimentées par défaut). |

> ⚡ **Tension, attention !** Le Raspberry Pi fonctionne à **3,3V** sur ses broches GPIO. Le Cube Orange est compatible 3,3V sur son port TELEM1. **Ne branche jamais directement un signal 5V sur le GPIO du Pi** — tu grilleras la carte !

> ⚠️ **Pourquoi un encodeur PPM ?** Le récepteur Joysway J5C01R utilise un protocole propriétaire et ne sort que des canaux **PWM séparés** (un fil signal par canal). Or le port **RCIN du Cube Orange** n'accepte que du **PPM** ou du **SBUS**. L'encodeur fait donc le pont entre les deux.

---

## 🎨 Code couleur des fils

| Couleur | Fonction |
|---------|----------|
| 🔴 Rouge | Alimentation (+5V ou +3,3V) |
| ⚫ Noir | Masse (GND) |
| 🔵 Bleu | TX (données envoyées) |
| 🟢 Vert | RX (données reçues) |
| 🟡 Jaune | Signal servo / PWM |
| 🟠 Orange | Signal PPM (encodeur → Cube RCIN) |

---

## 📌 Pinout officiel TELEM1

Le connecteur TELEM1 est un **JST-GH 6 broches**. Les pins sont numérotées **6 → 1 de gauche à droite** sur le connecteur physique.

```
Pin  Nom       I/O   Tension        Couleur fil   Description
─────────────────────────────────────────────────────────────
 6   GND        -    GND            Noir          → Pi Pin 6
 5   MCU_RTS   IN    3,3-5V TTL    Gris/Noir     Non connecté
 4   MCU_CTS   OUT   3,3-5V TTL    Gris/Noir     Non connecté
 3   MCU_RX    IN    3,3-5V TTL    Vert/Noir     → Pi Pin 8 (TX du Pi)
 2   MCU_TX    OUT   3,3-5V TTL    Jaune/Noir    → Pi Pin 10 (RX du Pi)
 1   VCC_5V    OUT   5V            Rouge/Gris    Non connecté (Pi a sa propre alim)
```

> ✅ **On n'utilise que 3 fils** : GND (pin 6), RX (pin 3), TX (pin 2). Les pins 4 et 5 (CTS/RTS) ne sont pas nécessaires pour une connexion MAVLink simple.

---

## 📊 Tableau des connexions

| Départ | Broche départ | Arrivée | Broche arrivée | Couleur | Rôle |
|--------|--------------|---------|----------------|---------|------|
| Raspberry Pi | Pin 8 – GPIO14 (TX) | Cube Orange TELEM1 | Pin 3 – RX | 🔵 Bleu | Le Pi envoie des données au Cube |
| Raspberry Pi | Pin 10 – GPIO15 (RX) | Cube Orange TELEM1 | Pin 2 – TX | 🟢 Vert | Le Cube envoie des données au Pi |
| Raspberry Pi | Pin 6 – GND | Cube Orange TELEM1 | Pin 6 – GND | ⚫ Noir | Masse commune obligatoire ! |
| Récepteur J5C01R | CH1 – signal | Encodeur PPM | IN1 – signal | 🟡 Jaune/Blanc | Stick gouvernail |
| Récepteur J5C01R | CH2 – signal | Encodeur PPM | IN2 – signal | 🟡 Jaune/Blanc | Stick voile |
| Récepteur J5C01R | CH6 – signal | Encodeur PPM | IN3 – signal | 🟡 Jaune/Blanc | Interrupteur mode RC/Auto |
| Encodeur PPM | PPM OUT – signal | Cube Orange RCIN | Signal | 🟠 Orange | Trame PPM (tous canaux) |
| Encodeur PPM | VCC | Cube Orange RCIN | +5V | 🔴 Rouge | Alimentation de l'encodeur par le Cube |
| Encodeur PPM | GND | Cube Orange RCIN | GND | ⚫ Noir | Masse de l'encodeur |
| BEC 5V | +5V | Cube Orange MAIN OUT3 | + (milieu) | 🔴 Rouge | Alimente le rail servo |
| BEC 5V | GND | Cube Orange MAIN OUT3 | − (bas) | ⚫ Noir | Masse du rail servo |
| Cube Orange MAIN OUT1 | Signal PWM | Servo Gouvernail | Signal | 🟡 Jaune | Commande de position |
| Cube Orange MAIN OUT1 | +5V | Servo Gouvernail | +5V | 🔴 Rouge | Alimentation du servo |
| Cube Orange MAIN OUT1 | GND | Servo Gouvernail | GND | ⚫ Noir | Masse du servo |
| Cube Orange MAIN OUT2 | Signal PWM | Servo Voile | Signal | 🟡 Jaune | Commande de la voile |

> 💡 **Astuce câblage côté récepteur** : seuls les fils **signal** (blanc/jaune) des canaux CH1, CH2 et CH6 sont à tirer vers l'encodeur. Les fils rouge (+5V) et noir (GND) de ces connecteurs peuvent rester non connectés côté encodeur puisque celui-ci est alimenté par le Cube. Le canal **CH5** du récepteur reste branché sur la **batterie** (c'est lui qui alimente le récepteur).

---

## 🔧 Étapes de câblage

> ⚠️ **Avant de commencer :** éteins tout ! Raspberry Pi débranché de son alimentation, batterie du Cube déconnectée. On ne branche jamais sur une carte sous tension.

### 1. Active l'UART sur le Raspberry Pi

```bash
sudo raspi-config
# → Interface Options → Serial Port
# → Login shell accessible over serial : NON
# → Serial port hardware enabled : OUI
# → Reboot
```

Après reboot, vérifie que le port est disponible :
```bash
ls /dev/ttyAMA0
```

> Si `/dev/ttyAMA0` n'apparaît pas, ajoute ces lignes dans `/boot/firmware/config.txt` :
> ```
> dtoverlay=disable-bt
> enable_uart=1
> ```
> Puis : `sudo systemctl disable bluetooth && sudo reboot`

### 2. Identifie les bonnes broches sur le Pi

```bash
pinout  # affiche le schéma des 40 broches dans le terminal
```

- **Pin 6** → GND (noir)
- **Pin 8** → GPIO14 / TX (bleu)
- **Pin 10** → GPIO15 / RX (vert)

### 3. Branche l'UART Pi ↔ Cube Orange TELEM1

Connecte les 3 fils sur le connecteur JST-GH du port **TELEM1** du Cube.

> **⚠️ Attention au croisement :** TX du Pi va sur **RX** du Cube, et RX du Pi va sur **TX** du Cube. C'est toujours comme ça entre deux appareils qui parlent !

### 4. Branche le récepteur J5C01R sur l'encodeur PPM, puis l'encodeur sur le Cube

Le récepteur **Joysway J5C01R** ne sort que des signaux **PWM individuels**, incompatibles avec le port RCIN du Cube qui attend du **PPM** ou du **SBUS**. Il faut donc insérer un **encodeur PPM** entre les deux.

**Schéma de principe :**

```
Récepteur J5C01R         Encodeur PPM           Cube Orange

  CH1 (stick gouv.) ────→ IN1 ──┐
  CH2 (stick voile) ────→ IN2   │
  CH6 (mode RC/Auto)────→ IN3 ──┤
                                 ├──→ PPM OUT ──────→ RCIN
                                 │                     (alimente l'encodeur)
                                 └────── VCC/GND ←─────
  CH5 (batterie) ────→ reste branché sur la batterie de bord
```

**Marche à suivre :**

1. **Côté récepteur** : tire uniquement le **fil signal** (blanc/jaune) de CH1, CH2 et CH6 vers les entrées IN1, IN2, IN3 de l'encodeur. Les fils rouge et noir de ces trois connecteurs ne sont pas utilisés côté encodeur.
2. **Côté encodeur** : vérifie que le **cavalier d'alimentation est bien NON soudé** (c'est la configuration par défaut) — l'encodeur sera alors alimenté par le Cube via le port RCIN.
3. **Encodeur → Cube** : branche la sortie **PPM OUT** de l'encodeur sur le port **RCIN** du Cube Orange avec un câble 3 broches (Signal, +5V, GND). C'est ce même câble qui fournit l'alimentation à l'encodeur.
4. **CH5 du récepteur** reste branché sur la **batterie** — c'est l'alimentation du récepteur, elle n'a rien à voir avec l'encodeur.

> ✅ **Aucune configuration spéciale dans ArduPilot** : le Cube détecte automatiquement le signal PPM sur RCIN. Les canaux non connectés de l'encodeur (IN4 à IN8) envoient une valeur neutre (1500 µs) dans la trame PPM, ce qui ne pose aucun problème.

> ⚠️ **Remarque firmware** : le micrologiciel de l'encodeur est pré-configuré pour lire 8 canaux. Si tu veux un jour inverser le signal PPM ou modifier les valeurs de failsafe, il faut un programmateur AVR ISP pour reprogrammer la puce. Ce n'est pas nécessaire pour notre usage.

### 5. Branche le BEC sur le rail MAIN OUT

Le Cube Orange ne fournit **pas** de 5V sur le rail servo par défaut. Branche le BEC 5V sur **OUT3** (ou tout OUT libre) :
- Fil rouge (+5V) → broche milieu
- Fil noir (GND) → broche bas

Un seul BEC alimente tous les OUT simultanément (rail commun).

### 6. Branche les servos sur les sorties Main

- Servo gouvernail → **MAIN OUT 1**
- Servo voile → **MAIN OUT 2**

> **⚠️ Vérifier le sens des connecteurs au multimètre** — sur le Cube Orange Plus, l'ordre est inversé par rapport au schéma standard (voir section suivante).

### 7. Configure ArduPilot (Mission Planner)

Installer Mission Planner sur votre poste de travail : https://docs.cubepilot.org/user-guides/autopilot/the-cube/setup/mission-planner

```
SERIAL1_PROTOCOL = 2    # MAVLink2
SERIAL1_BAUD     = 57   # 57600 bauds
BRD_SAFETY_DEFLT = 0    # Safety switch désactivé
FS_THR_ENABLE    = 0    # Failsafe radio désactivé
```

**Calibration RC** (une fois l'encodeur branché) :
- Dans Mission Planner : **Initial Setup → Mandatory Hardware → Radio Calibration**
- Vérifier que les sticks CH1 (gouvernail) et CH2 (voile) bougent bien les barres correspondantes
- Vérifier que l'interrupteur CH6 bascule clairement entre deux positions (RC / Auto)
- Cliquer sur **Calibrate Radio** et faire les mouvements demandés

### 8. Teste avant de refermer le boîtier

```bash
# Sur le Pi — vérifie la réception MAVLink
python3 testPiCube.py
```

---

## ⚠️ Erreurs fréquentes à éviter

### 🔄 Sens des connecteurs inversé sur Cube Orange Plus !
Sur ce carrier board, l'ordre des broches MAIN OUT est **inversé** par rapport au standard :

| Position | Cube Orange Standard | Cube Orange **Plus** ⚠️ |
|----------|---------------------|------------------------|
| Haut (intérieur PCB) | Signal (S) | **GND (−)** |
| Milieu | +5V (+) | **+5V (+)** |
| Bas (bord PCB) | GND (−) | **Signal (S)** |

> ✅ **Si les servos ne répondent pas** malgré une config correcte → retourne tous les connecteurs (servo ET BEC) à 180°. Vérifie toujours au multimètre : +5V entre broche milieu et GND.

### 🔌 Cavalier d'alimentation soudé sur l'encodeur PPM ?
Si le cavalier est soudé, l'encodeur attend d'être alimenté **par le récepteur** — ce qui ne fonctionne pas dans notre montage puisqu'on ne tire que les fils signal côté J5C01R. **Laisser le cavalier NON soudé** pour que l'alimentation vienne du Cube via RCIN.

### 🚫 TX sur TX, RX sur RX ?
C'est la faute numéro 1 ! TX = "j'envoie", RX = "je reçois". Le TX d'un appareil doit toujours aller sur le RX de l'autre — ils se **croisent** toujours.

### ⚠️ Oublier la masse commune (GND) !
Sans le fil GND entre Pi et Cube, rien ne fonctionnera même si tous les autres fils sont corrects.

### 🔋 Alimenter les servos depuis le Pi ?
Mauvaise idée. Les servos consomment trop de courant. Utilise un **BEC externe 5V** branché sur le rail MAIN OUT.

### 📻 Pas de signal PPM détecté par le Cube ?
- Vérifier qu'une LED s'allume sur l'encodeur quand le Cube est alimenté (preuve qu'il reçoit bien ses 5V via RCIN)
- Vérifier que le récepteur J5C01R est bien appairé avec l'émetteur J4C05 et sous tension (LED fixe)
- Vérifier au multimètre/oscilloscope que les sorties CH1/CH2/CH6 du récepteur bougent bien quand on actionne les sticks/interrupteur
- Dans Mission Planner → **Flight Data → Status** → les barres RC IN doivent bouger

---

## 🔌 Sens des connecteurs RCIN & Servos

### Cube Orange Plus — ordre réel vérifié en pratique

```
┌──────────────────────────────────┐
│  MAIN OUT (vue de face)          │
│  ┌───┐                           │
│  │ − │  ← GND       (haut)      │
│  │ + │  ← +5V       (milieu)    │
│  │ S │  ← Signal    (bas)       │
│  └───┘                           │
│           ↑ bord du PCB          │
└──────────────────────────────────┘
```

**Câble servo branché sur Cube Orange Plus :**
```
Noir / Brun  →  broche haute  (GND)
Rouge        →  broche milieu (+5V)
Jaune/Blanc  →  broche basse  (Signal)
```

---

## 🐍 Scripts de test Python

### Installation (une seule fois)

```bash
# Activer l'UART
sudo raspi-config
# → Interface Options → Serial Port → Login shell: NON → Port série: OUI → Reboot

# Installer pymavlink
pip3 install pymavlink --break-system-packages

# Vérifier
python3 -c "from pymavlink import mavutil; print('OK')"
```

---

### Script 1 — Test GPS & IMU (`testPiCube.py`)

Vérifie que le Pi reçoit bien les données du Cube : GPS, vitesse, cap, attitude (roulis/tangage).

```python
from pymavlink import mavutil
import math

print("Connexion...")
master = mavutil.mavlink_connection('/dev/ttyAMA0', baud=57600)

print("En attente du heartbeat...")
msg = master.recv_match(type='HEARTBEAT', blocking=True, timeout=5)

if msg is None:
    print("❌ Pas de heartbeat - vérifier câblage/baud rate")
else:
    print(f"✅ Cube connecté ! Système: {master.target_system}")

    print("Lecture GPS...")
    msg = master.recv_match(type='GPS_RAW_INT', blocking=True, timeout=5)
    if msg is None:
        print("❌ Pas de données GPS")
    else:
        print(f"Lat: {msg.lat/1e7}, Lon: {msg.lon/1e7}, Fix: {msg.fix_type}")

        msg = master.recv_match(type='VFR_HUD', blocking=True, timeout=5)
        print(f"Vitesse: {msg.groundspeed} m/s, Cap: {msg.heading}°")

        msg = master.recv_match(type='ATTITUDE', blocking=True, timeout=5)
        print(f"Roulis: {math.degrees(msg.roll):.1f}°, Tangage: {math.degrees(msg.pitch):.1f}°")

        msg = master.recv_match(type='HEARTBEAT', blocking=True, timeout=5)
        print(f"Mode: {msg.custom_mode}")
```

```bash
python3 testPiCube.py
```

✅ Tu dois voir : heartbeat, coordonnées GPS, vitesse, cap, roulis/tangage.
❌ Si "Pas de heartbeat" → vérifier câblage TELEM1 et `SERIAL1_PROTOCOL=2` / `SERIAL1_BAUD=57` dans Mission Planner.

---

### Script 2 — Test Servos (`test_servos.py`) sans télécommande

Envoie des commandes PWM directes sur les sorties MAIN OUT.

> **⚠️ Configuration ArduPilot requise :** mettre `SERVO5_FUNCTION=0` et `SERVO6_FUNCTION=0` dans Mission Planner. Un canal avec une fonction assignée retournera l'erreur *"Channel X is already in use"*.

```python
import time
from pymavlink import mavutil

PORT  = '/dev/ttyAMA0'
BAUD  = 57600

# Canaux avec SERVO5_FUNCTION=0 et SERVO6_FUNCTION=0
CANAL_GOUVERNAIL = 5   # MAIN OUT 5
CANAL_VOILE      = 6   # MAIN OUT 6

PWM_NEUTRE = 1500
PWM_MIN    = 1100
PWM_MAX    = 1900

master = mavutil.mavlink_connection(PORT, baud=BAUD)
master.wait_heartbeat()
print("✅ Cube connecté !")

def bouger_servo(canal, pwm):
    pwm = max(PWM_MIN, min(PWM_MAX, pwm))
    master.mav.command_long_send(
        master.target_system, master.target_component,
        mavutil.mavlink.MAV_CMD_DO_SET_SERVO,
        0, canal, pwm, 0, 0, 0, 0, 0
    )
    print(f"  → OUT {canal} : {pwm} µs")

print("Neutre...")
bouger_servo(CANAL_GOUVERNAIL, PWM_NEUTRE)
bouger_servo(CANAL_VOILE,      PWM_NEUTRE)
time.sleep(2)

print("Gouvernail gauche...")
bouger_servo(CANAL_GOUVERNAIL, PWM_MIN)
time.sleep(2)

print("Gouvernail droite...")
bouger_servo(CANAL_GOUVERNAIL, PWM_MAX)
time.sleep(2)

print("Voile fermée...")
bouger_servo(CANAL_VOILE, PWM_MIN)
time.sleep(2)

print("Voile ouverte...")
bouger_servo(CANAL_VOILE, PWM_MAX)
time.sleep(2)

print("Balayage gouvernail...")
for pwm in range(PWM_MIN, PWM_MAX + 1, 40):
    bouger_servo(CANAL_GOUVERNAIL, pwm)
    time.sleep(0.1)

bouger_servo(CANAL_GOUVERNAIL, PWM_NEUTRE)
bouger_servo(CANAL_VOILE,      PWM_NEUTRE)
print("✅ Tests terminés !")
```

```bash
python3 test_servos.py
```

✅ Les servos doivent bouger : neutre → gauche → droite → voile fermée → voile ouverte → balayage progressif.
❌ Si "Channel X is already in use" → mettre `SERVO5_FUNCTION=0` et `SERVO6_FUNCTION=0` dans Mission Planner.
❌ Si les servos ne bougent pas → vérifier le BEC (5V sur rail MAIN OUT) et le sens des connecteurs.

---

## ⚙️ Paramètres ArduPilot

Fichier complet : [`ListeParamStandards.param`](ListeParamStandards.param)

Pour charger dans Mission Planner : **Config → Full Parameter List → Load from file → Write Params**

### Paramètres clés

#### Communication série
```ini
SERIAL1_PROTOCOL = 2    # MAVLink2 — TELEM1 → Raspberry Pi
SERIAL1_BAUD     = 57   # 57600 bauds
SERIAL2_PROTOCOL = 2    # MAVLink2 — TELEM2
SERIAL3_PROTOCOL = 5    # GPS — port GPS1
```

#### Entrée RC (via encodeur PPM)
```ini
# Mapping par défaut — à ajuster après calibration radio
RCMAP_ROLL       = 1    # CH1 encodeur = gouvernail (stick)
RCMAP_THROTTLE   = 2    # CH2 encodeur = voile (stick)
# CH3 encodeur = interrupteur mode RC/Auto (CH6 du récepteur)
```

#### Sorties servo
```ini
SERVO1_FUNCTION  = 33   # Main Sail (OUT1)
SERVO2_FUNCTION  = 89   # Main Sail winch (OUT2)
SERVO3_FUNCTION  = 35   # Main Sail 2 (OUT3)
SERVO9_FUNCTION  = 26   # Steering — gouvernail (AUX1)
SERVO5_FUNCTION  = 0    # Disabled — libre pour tests MAVLink
SERVO6_FUNCTION  = 0    # Disabled — libre pour tests MAVLink
BRD_PWM_VOLT_SEL = 1    # Sorties MAIN OUT en 5V (pas 3,3V)
```

#### Configuration voilier
```ini
SAIL_ENABLE      = 1    # Active le mode voilier ArduSail
SAIL_ANGLE_IDEAL = 25   # Angle de voile idéal / vent (degrés)
SAIL_ANGLE_MAX   = 80   # Angle voile maximum
SAIL_NO_GO_ANGLE = 45   # Zone interdite face au vent
SAIL_HEEL_MAX    = 20   # Gîte maximale tolérée
FRAME_CLASS      = 2    # Boat
WNDVN_TYPE       = 4    # Type de girouette
```

#### Sécurité & failsafe
```ini
BRD_SAFETY_DEFLT = 0    # Safety switch désactivé au démarrage
BRD_SAFETYOPTION = 0    # Options safety switch
FS_THR_ENABLE    = 0    # Failsafe radio désactivé
FS_GCS_ENABLE    = 0    # Failsafe GCS désactivé
```

---

## 🔄 Flux de données

```
📱 Émetteur Joysway J4C05
      ↓ radio 2,4GHz (protocole propriétaire Joysway)
📻 Récepteur Joysway J5C01R
      ↓ PWM (CH1, CH2, CH6 — fils signal uniquement)
🔄 Encodeur PPM (ATMEGA328p)
      ↓ PPM (trame unique)
🟠 Cube Orange — RCIN (ArduPilot ArduRover)
      ↕ MAVLink UART (TELEM1, 57600 bauds)
🍓 Raspberry Pi (navigation autonome)
      ↓
🟠 Cube Orange
      ↓ PWM MAIN OUT
⚙️ Servos (gouvernail + voile)
```

---

## 📁 Fichiers du projet

| Fichier | Description |
|---------|-------------|
| `rpi_cube_orange_cablage.html` | Guide de câblage complet avec schémas SVG interactifs |
| `ListeParamStandards.param` | Paramètres ArduPilot complets du Cube Orange |
| `testPiCube.py` | Script de test GPS/IMU via MAVLink |
| `test_servos.py` | Script de test des servos via MAVLink |
| `cube_connectors_photo.png` | Photo officielle des connecteurs Cube Orange |
| `telem_pinout_table.png` | Tableau pinout officiel TELEM1/TELEM2 |

---

## 🏆 Battleboats 2026

Compétition de voiliers autonomes · Toulon, Fort Saint-Louis · **9-10 mai 2026**

---

*Guide rédigé et testé sur Cube Orange Plus + Raspberry Pi 4B · ArduRover V4.6.3*
