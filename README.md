# Passage piéton intelligent (réseau sans fil XBee / Zigbee)

Système de **feux de circulation à la demande** pour un passage piéton, construit autour de **trois nœuds sans fil** : un coordinateur (ESP32-S3) et deux nœuds Arduino Uno équipés chacun d'un bouton et d'un feu tricolore. Quand un piéton appuie sur un bouton, le coordinateur déclenche la séquence des feux sur les deux nœuds. Une matrice LED affiche un piéton qui marche pendant la traversée.

## Architecture

```
  Nœud 2 (Arduino Uno)                Nœud 1 (ESP32-S3)               Nœud 3 (Arduino Uno)
  bouton + feu + matrice LED          COORDINATEUR                    bouton + feu
         │                                   │                               │
         ├────────── 'X' (bouton) ──────────►│◄────────── 'Z' (bouton) ──────┤
         │◄──────── 'Y' / 'R' / 'G' ─────────┴──────── 'Y' / 'R' / 'G' ─────►│
```

## Protocole

Les nœuds échangent un seul caractère par message, à 9600 bauds.

| Caractère | Sens | Signification |
|:---:|---|---|
| `X` | Nœud 2 vers Nœud 1 | Le bouton du nœud 2 a été pressé |
| `Z` | Nœud 3 vers Nœud 1 | Le bouton du nœud 3 a été pressé |
| `Y` | Nœud 1 vers Nœuds 2 et 3 | Feu jaune (avertissement) |
| `R` | Nœud 1 vers Nœuds 2 et 3 | Feu rouge, le piéton traverse |
| `G` | Nœud 1 vers Nœuds 2 et 3 | Feu vert, la circulation reprend |

## Séquence d'une traversée

1. Un piéton appuie sur un bouton : le nœud envoie `X` ou `Z` au coordinateur.
2. Le coordinateur envoie `Y` : les feux passent au **jaune** pendant 2 s.
3. Il envoie `R` : les feux passent au **rouge** pendant 7 s. Sur le nœud 2, la matrice LED affiche un piéton animé.
4. Il envoie `G` : les feux repassent au **vert**, la matrice s'éteint. Un délai de sécurité de 3 s est respecté avant d'accepter une nouvelle demande.

Pendant une traversée, le coordinateur est marqué « occupé » pour ne pas relancer une séquence.

## Matériel

- 1 x ESP32-S3 (nœud 1, coordinateur)
- 2 x Arduino Uno (nœuds 2 et 3)
- 3 x module XBee (un par nœud) avec adaptateur
- 2 x bouton-poussoir
- 6 x LED (2 rouges, 2 jaunes, 2 vertes) avec résistances
- 1 x matrice LED 8x8 avec driver MAX7219 (sur le nœud 2)

## Branchements

**Nœud 1 (ESP32-S3)**

| Élément | Broche |
|---|---|
| XBee `DOUT` vers ESP32 (RX1) | GPIO 18 |
| XBee `DIN` depuis ESP32 (TX1) | GPIO 17 |

**Nœuds 2 et 3 (Arduino Uno)**

| Élément | Broche |
|---|---|
| XBee `DOUT` vers Arduino (RX) | D2 |
| XBee `DIN` depuis Arduino (TX) | D3 |
| Bouton (l'autre côté à GND) | D4 |
| LED rouge | D5 |
| LED jaune | D6 |
| LED verte | D7 |

**Matrice MAX7219 (nœud 2 uniquement)**

| Broche MAX7219 | Broche Arduino |
|---|---|
| DIN | D10 |
| CLK | D11 |
| CS | D12 |

Les XBee fonctionnent en 3,3 V : sur les Arduino Uno (5 V), un diviseur de tension sur la broche `DIN` du XBee protège le module.

## Configuration des XBee

Avec le logiciel XCTU, régler les trois modules en **9600 bauds**, sur le **même réseau** (même PAN ID), avec un module configuré en coordinateur.

## Installation

1. Installer l'[Arduino IDE](https://www.arduino.cc/en/software).
2. Pour le nœud 2, installer la bibliothèque **LedControl** (`Croquis > Inclure une bibliothèque > Gérer les bibliothèques`). `SoftwareSerial` est déjà incluse.
3. Pour le nœud 1, installer le paquet de cartes **esp32** (Espressif) et choisir la carte *ESP32S3 Dev Module*.
4. Téléverser :
   - `noeud1/noeud1.ino` sur l'ESP32-S3
   - `noeud2/noeud2.ino` sur le premier Arduino Uno
   - `noeud3/noeud3.ino` sur le second Arduino Uno

## Utilisation

1. Alimenter les trois nœuds : les feux des nœuds 2 et 3 sont au **vert**.
2. Appuyer sur le bouton de l'un des deux nœuds.
3. Les feux passent au jaune, puis au rouge (avec le piéton animé sur la matrice du nœud 2), puis reviennent au vert.

## Structure du dépôt

```
passage-pieton-xbee/
├── noeud1/noeud1.ino   # Coordinateur (ESP32-S3)
├── noeud2/noeud2.ino   # Feu + bouton + matrice LED (Arduino Uno)
├── noeud3/noeud3.ino   # Feu + bouton (Arduino Uno)
├── .gitignore
└── README.md
```

## Auteur

Kawthar Derouich
