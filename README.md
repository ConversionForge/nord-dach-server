# Nord-Dach App – Server-Paket

Vorübergehende Ablage für die Einrichtung der Synology. Wird danach gelöscht.

## 1. App herunterladen

**[nord-dach-app-2.0.0.tar herunterladen](https://github.com/ConversionForge/nord-dach-server/raw/main/nord-dach-app-2.0.0.tar)** (ca. 80 MB)

In DSM: Container Manager › Image › Aktion › Importieren › Aus Datei hinzufügen › vom Computer hochladen.

## 2. docker-compose.yml

In DSM: Container Manager › Projekt › Erstellen › Quelle „docker-compose.yml erstellen“ und diesen Inhalt einfügen
(oben rechts am Kasten auf „Copy“ klicken):

```yaml
# Nord-Dach App auf der Synology DS925+
# Container Manager → Projekt „nord-dach“, Ordner /docker/nord-dach (diese Datei liegt dort).
#
#   Handy ──HTTPS──► FritzBox (Port 443) ──► Synology Reverse Proxy (Port 8443, Let's-Encrypt-Zertifikat) ──► App (Port 8090)
#
# Fotos und Dokumente: freigegebener Ordner „Nord-Dach“ (im Büro als \\NORD-DACH-NAS\Nord-Dach sichtbar).
# Datenbank, Vorschaubilder und die tägliche Datenbank-Sicherung: Unterordner „daten“ neben dieser Datei.
# Beides sichert Hyper Backup jede Nacht auf die USB-Festplatte.

name: nord-dach

services:
  app:
    image: nord-dach-app:2.0.0
    container_name: nord-dach-app
    restart: always
    ports:
      # Im Büronetz: Ersteinrichtung und Ziel des Reverse Proxys. NICHT in der FritzBox freigeben!
      - '8090:8080'
    environment:
      TZ: Europe/Berlin
      # Internet-Adresse der App (nur Anzeige auf der Technik-Seite)
      PUBLIC_URL: https://nord-dach.synology.me
      FIRMA: Nord-Dach Service
      SERVER_NAME: Synology DS925+
      # So sieht man den Foto-Ordner am Büro-PC im Explorer (in der App unter „Technik“ änderbar)
      WINDOWS_PFAD: '\\NORD-DACH-NAS\Nord-Dach'
      # Der Synology Reverse Proxy verbindet sich aus dem eigenen Netz – nur dem wird vertraut
      TRUST_PROXY: 'loopback,uniquelocal'
    volumes:
      - ./daten:/data
      - /volume1/Nord-Dach:/ablage
```

## 3. Nach dem Festplatten-Umzug (Daten auf Volume 2)

Gleiche Datei, nur der Foto-Ordner liegt jetzt auf Volume 2. Diesen Inhalt nach dem Umzug verwenden:

```yaml
# Nord-Dach App auf der Synology DS925+
# Container Manager → Projekt „nord-dach“, Ordner /docker/nord-dach (diese Datei liegt dort).
#
#   Handy ──HTTPS──► FritzBox (Port 443) ──► Synology Reverse Proxy (Port 8443, Let's-Encrypt-Zertifikat) ──► App (Port 8090)
#
# Fotos und Dokumente: freigegebener Ordner „Nord-Dach“ (im Büro als \\NORD-DACH-NAS\Nord-Dach sichtbar).
# Datenbank, Vorschaubilder und die tägliche Datenbank-Sicherung: Unterordner „daten“ neben dieser Datei.
# Beides sichert Hyper Backup jede Nacht auf die USB-Festplatte.

name: nord-dach

services:
  app:
    image: nord-dach-app:2.0.0
    container_name: nord-dach-app
    restart: always
    ports:
      # Im Büronetz: Ersteinrichtung und Ziel des Reverse Proxys. NICHT in der FritzBox freigeben!
      - '8090:8080'
    environment:
      TZ: Europe/Berlin
      # Internet-Adresse der App (nur Anzeige auf der Technik-Seite)
      PUBLIC_URL: https://nord-dach.synology.me
      FIRMA: Nord-Dach Service
      SERVER_NAME: Synology DS925+
      # So sieht man den Foto-Ordner am Büro-PC im Explorer (in der App unter „Technik“ änderbar)
      WINDOWS_PFAD: '\\NORD-DACH-NAS\Nord-Dach'
      # Der Synology Reverse Proxy verbindet sich aus dem eigenen Netz – nur dem wird vertraut
      TRUST_PROXY: 'loopback,uniquelocal'
    volumes:
      - ./daten:/data
      - /volume2/Nord-Dach:/ablage
```

## Falls der Import über den Browser nicht klappt

DSM: Systemsteuerung › Aufgabenplanung › Erstellen › Geplante Aufgabe › Benutzerdefiniertes Skript, Benutzer **root**,
dieses Skript einfügen, speichern, die Aufgabe markieren und **Ausführen**:

```sh
PATH=$PATH:/usr/local/bin
cd "$(ls -d /volume*/docker/nord-dach | head -n 1)" || exit 1
curl -fL -o app.tar https://github.com/ConversionForge/nord-dach-server/raw/main/nord-dach-app-2.0.0.tar || exit 1
docker load -i app.tar && rm -f app.tar
```

Nach ein bis zwei Minuten steht im Container Manager unter Image **nord-dach-app**.
