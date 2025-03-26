# M347 Docker Praxis-Prüfung Cheatsheet

**Modul:** 347 - Dienst mit Container anwenden
**Kapitel:** 1-3 (Container, Docker, Images)
**Fokus:** Praktische Befehle und Konzepte

**Hinweis:** Abhängig von deiner Systemkonfiguration musst du Docker-Befehlen möglicherweise `sudo` voranstellen oder deinen Benutzer zur `docker`-Gruppe hinzufügen (`sudo usermod -aG docker $USER` und neu anmelden).

## Kernkonzepte Kurzübersicht

*   **Containerisierung:** Das Verpacken einer Anwendung mit ihren Abhängigkeiten in eine isolierte Umgebung (Container), die auf dem Kernel des Host-Betriebssystems läuft. Leichtgewichtiger als VMs.
*   **Image:** Eine schreibgeschützte Vorlage, die den Anwendungscode, Bibliotheken, Abhängigkeiten und die Laufzeitumgebung enthält. Wird verwendet, um Container zu erstellen. Wird aus einem `Dockerfile` gebaut.
*   **Layer (Schicht):** Images bestehen aus mehreren schreibgeschützten Schichten. Jede Anweisung in einem `Dockerfile` erzeugt typischerweise eine neue Schicht. Schichten werden zwischengespeichert und geteilt, was Builds und Pulls effizient macht.
*   **Container:** Eine ausführbare Instanz eines Images. Er ist isoliert, teilt sich aber den Kernel des Host-Betriebssystems. Hat sein eigenes Dateisystem (basierend auf dem Image + einer beschreibbaren Schicht), Prozesse und Netzwerkschnittstelle.
*   **Registry:** Ein Speicher- und Verteilungssystem für Docker-Images (z. B. Docker Hub, GitLab Container Registry).
*   **Dockerfile:** Eine Textdatei, die Anweisungen zum Erstellen eines Docker-Images enthält.

*   **Virtualisierung vs. Containerisierung:**
    *   **VMs:** Führen ein vollständiges Gast-Betriebssystem auf einem Hypervisor aus. Schwergewichtiger, starke Isolation, langsamerer Start.
    *   **Container:** Teilen sich den Kernel des Host-Betriebssystems. Leichtgewichtiger, schwächere Isolation (auf Prozessebene), schnellerer Start.

## Essenzielle Docker-Befehle

### Image-Verwaltung

*   **Images auflisten:**
    ```bash
    docker images
    ```
*   **Image herunterladen (Pull):**
    ```bash
    # Neueste Version herunterladen
    docker pull <image_name>

    # Spezifische Version (Tag) herunterladen
    docker pull <image_name>:<tag>
    # Beispiel:
    docker pull nginx:1.21-alpine
    ```
*   **Image entfernen:** (Container, die das Image verwenden, müssen zuerst entfernt werden)
    ```bash
    docker rmi <image_id_oder_name>[:tag]
    # Entfernen erzwingen (mit Vorsicht verwenden)
    docker rmi -f <image_id_oder_name>
    ```
*   **Image aus Dockerfile bauen:**
    ```bash
    # Im aktuellen Verzeichnis bauen, Image taggen
    docker build -t <dein_image_name>[:tag] .
    # Beispiel:
    docker build -t meine-web-app:v1 .
    ```

### Container-Lebenszyklus

*   **Container ausführen (Erstellen & Starten):** (Siehe `docker run` Optionen unten)
    ```bash
    docker run [OPTIONS] <image_name>[:tag] [COMMAND] [ARG...]
    # Beispiel: Nginx im Hintergrund ausführen, Port 8080 auf 80 mappen
    docker run -d --name mein-nginx -p 8080:80 nginx
    ```
*   **Laufende Container auflisten:**
    ```bash
    docker ps
    ```
*   **Alle Container auflisten (laufende und gestoppte):**
    ```bash
    docker ps -a
    ```
*   **Einen laufenden Container stoppen:**
    ```bash
    docker stop <container_id_oder_name>
    ```
*   **Einen gestoppten Container starten:**
    ```bash
    docker start <container_id_oder_name>
    ```
*   **Einen Container neu starten:**
    ```bash
    docker restart <container_id_oder_name>
    ```
*   **Einen gestoppten Container entfernen:**
    ```bash
    docker rm <container_id_oder_name>
    # Mehrere gestoppte Container entfernen
    docker rm $(docker ps -a -q)
    # Einen laufenden Container zwangsweise entfernen (mit Vorsicht verwenden)
    docker rm -f <container_id_oder_name>
    ```
*   **Container-Logs anzeigen:**
    ```bash
    docker logs <container_id_oder_name>
    # Logs verfolgen (wie tail -f)
    docker logs -f <container_id_oder_name>
    ```
*   **Einen Befehl in einem laufenden Container ausführen:**
    ```bash
    docker exec [OPTIONS] <container_id_oder_name> <command>
    # Beispiel: Eine interaktive Shell im Container erhalten
    docker exec -it <container_id_oder_name> /bin/bash
    # Beispiel: Dateien im Verzeichnis /app auflisten
    docker exec <container_id_oder_name> ls /app
    ```
*   **Container-/Image-Details inspizieren:** (Liefert detaillierte JSON-Infos)
    ```bash
    docker inspect <container_id_oder_name_oder_image_id>
    ```

## `docker run` Optionen (Wichtige Optionen)

*   **`-d` oder `--detach`:** Container im Hintergrund ausführen (detached mode). Gibt die Container-ID aus.
    ```bash
    docker run -d nginx
    ```
*   **`-it`:** Interaktives TTY. Hält STDIN offen und weist ein Pseudo-TTY zu. Wird für interaktive Shells verwendet.
    ```bash
    docker run -it ubuntu /bin/bash
    ```
*   **`--name <name>`:** Dem Container einen benutzerdefinierten Namen zuweisen.
    ```bash
    docker run -d --name webserver nginx
    ```
*   **`-p <host_port>:<container_port>`:** Einen Port des Containers auf dem Host veröffentlichen (mappen).
    ```bash
    # Mappe Host-Port 8080 auf Container-Port 80
    docker run -d -p 8080:80 --name mein-web nginx
    ```
*   **`-v <quelle>:<ziel>`:** Volumes mounten. Daten persistieren oder Daten/Code teilen.
    *   **Benanntes Volume (Named Volume):** Docker verwaltet das Volume. `quelle` ist der Volume-Name.
        ```bash
        # Ein benanntes Volume erstellen
        docker volume create meine-daten
        # Container mit dem benannten Volume ausführen
        docker run -d -v meine-daten:/app/data --name daten-app mein-image
        ```
    *   **Bind Mount:** Mountet eine Datei oder ein Verzeichnis vom Host. `quelle` ist ein absoluter Pfad auf dem Host oder `$(pwd)/relativer/pfad`.
        ```bash
        # Mountet das aktuelle Host-Verzeichnis 'html' nach '/usr/share/nginx/html' im Container
        docker run -d -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html --name dev-web nginx
        ```
*   **`-e <VAR_NAME>=<wert>` oder `--env <VAR_NAME>=<wert>`:** Umgebungsvariablen im Container setzen.
    ```bash
    docker run -d -e MYSQL_ROOT_PASSWORD=secret --name db mysql
    ```
*   **`--network <netzwerk_name>`:** Den Container mit einem spezifischen Netzwerk verbinden.
    ```bash
    docker run -d --network mein-custom-net --name app1 mein-app-image
    ```
*   **`--ip <ip_adresse>`:** Dem Container eine statische IP-Adresse *innerhalb eines benutzerdefinierten Netzwerks* zuweisen. Erfordert `--network`.
    ```bash
    docker run -d --network mein-custom-net --ip 172.18.0.22 --name app2 mein-app-image
    ```

## Docker Networking

*   **Standardnetzwerk:** `bridge` (normalerweise `docker0`). Container auf der Standard-Bridge können über IP kommunizieren, aber die DNS-Auflösung nach Container-Namen funktioniert nicht zuverlässig.
*   **Benutzerdefinierte Netzwerke (User-Defined Networks):** Empfohlen für Anwendungen mit mehreren Containern. Bieten bessere Isolation und eingebaute DNS-Auflösung (Container können sich gegenseitig über Namen erreichen).
*   **Netzwerke auflisten:**
    ```bash
    docker network ls
    ```
*   **Ein Netzwerk erstellen (Bridge ist üblich):**
    ```bash
    docker network create <netzwerk_name>
    # Beispiel:
    docker network create meine-app-net
    ```
*   **Ein Netzwerk inspizieren:** (Zeigt verbundene Container, Gateway, Subnetz)
    ```bash
    docker network inspect <netzwerk_name>
    ```
*   **Einen laufenden Container mit einem Netzwerk verbinden:**
    ```bash
    docker network connect <netzwerk_name> <container_name_oder_id>
    ```
*   **Einen Container von einem Netzwerk trennen:**
    ```bash
    docker network disconnect <netzwerk_name> <container_name_oder_id>
    ```
*   **Ein Netzwerk entfernen:** (Container müssen zuerst getrennt/entfernt werden)
    ```bash
    docker network rm <netzwerk_name>
    ```

## Dockerfile Anweisungen (Häufige)

*   **`FROM <image>[:tag]`:** Gibt das Basis-Image an. *Muss die erste Anweisung sein.*
    ```dockerfile
    FROM ubuntu:22.04
    ```
*   **`WORKDIR /pfad/zum/verzeichnis`:** Setzt das Arbeitsverzeichnis für nachfolgende Anweisungen (`RUN`, `CMD`, `ENTRYPOINT`, `COPY`, `ADD`).
    ```dockerfile
    WORKDIR /app
    ```
*   **`RUN <befehl>`:** Führt Befehle während des Image-Build-Prozesses aus. Erzeugt eine neue Schicht. Oft verwendet, um Pakete zu installieren.
    ```dockerfile
    # Verwende &&, um Befehle in einer Schicht zu verketten und die Image-Grösse zu reduzieren
    RUN apt-get update && apt-get install -y \
        nginx \
        curl \
     && rm -rf /var/lib/apt/lists/*
    ```
*   **`COPY <quelle> <ziel>`:** Kopiert Dateien oder Verzeichnisse aus dem Build-Kontext (deine lokale Maschine) in das Dateisystem des Images. Transparenter als `ADD`.
    ```dockerfile
    # Kopiere lokale Datei 'app.py' nach '/app/app.py' im Image
    COPY app.py /app/
    # Kopiere alle Dateien aus dem lokalen 'html'-Verzeichnis nach '/usr/share/nginx/html' im Image
    COPY html/ /usr/share/nginx/html/
    ```
*   **`ADD <quelle> <ziel>`:** Ähnlich wie `COPY`, aber mit zusätzlichen Funktionen:
    *   Kann Dateien von URLs herunterladen.
    *   Kann komprimierte Dateien (tar, gzip, etc.) automatisch extrahieren, wenn die Quelle ein lokales Archiv ist.
    *   **Empfehlung:** Bevorzuge `COPY`, es sei denn, du benötigst explizit die URL- oder Auto-Extraktionsfunktionen von `ADD`.
    ```dockerfile
    # Beispiel: Herunterladen und Extrahieren
    # ADD https://example.com/file.tar.gz /usr/src/
    ```
*   **`EXPOSE <port> [<port>/<protokoll>...]`:** Informiert Docker, dass der Container zur Laufzeit auf den angegebenen Netzwerkports lauscht. *Veröffentlicht den Port nicht tatsächlich.* Dient als Dokumentation und kann von anderen Tools verwendet werden.
    ```dockerfile
    EXPOSE 80
    EXPOSE 5432/tcp
    ```
*   **`ENV <schlüssel>=<wert>`:** Setzt Umgebungsvariablen, die während des Builds (`RUN`) und beim Ausführen von Containern aus dem Image verfügbar sind.
    ```dockerfile
    ENV APP_VERSION=1.0
    ENV DATA_DIR=/var/data
    ```
*   **`CMD ["ausführbare_datei","param1","param2"]` (exec-Form - bevorzugt)** oder `CMD befehl param1 param2` (shell-Form): Gibt den Standardbefehl an, der ausgeführt wird, wenn ein Container startet.
    *   Es kann nur eine `CMD`-Anweisung geben. Wenn mehr als eine `CMD` aufgeführt wird, wirkt nur die letzte.
    *   Kann beim Starten eines Containers leicht überschrieben werden (`docker run image <neuer_befehl>`).
    ```dockerfile
    # Exec-Form (bevorzugt)
    CMD ["python", "app.py"]
    # Shell-Form
    # CMD python app.py
    ```
*   **`ENTRYPOINT ["ausführbare_datei","param1","param2"]` (exec-Form - bevorzugt)** oder `ENTRYPOINT befehl param1 param2` (shell-Form): Konfiguriert einen Container, der als ausführbare Datei läuft.
    *   Argumente, die über `docker run <image> arg1 arg2` übergeben werden, werden nach den `ENTRYPOINT`-Parametern angehängt.
    *   Weniger leicht zu überschreiben als `CMD`. Zum Überschreiben verwende `docker run --entrypoint <neuer_entrypoint> ...`.
    *   Wird oft in Kombination mit `CMD` verwendet, um Standardargumente für den `ENTRYPOINT` festzulegen.
    ```dockerfile
    ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
    # Kombination: ENTRYPOINT definiert Ausführbares, CMD definiert Standard-Args
    # ENTRYPOINT ["ping"]
    # CMD ["localhost"]
    # Das Ausführen von `docker run mein-ping-image google.com` würde `ping google.com` ausführen
    ```
*   **`VOLUME ["/pfad/im/container"]`:** Erstellt einen Mountpunkt mit dem angegebenen Namen und markiert ihn als Halter für extern gemountete Volumes vom nativen Host oder anderen Containern. Kann mit benannten Volumes oder Bind Mounts verwendet werden. Daten, die hier geschrieben werden, bleiben auch nach dem Entfernen des Containers erhalten (wenn ein benanntes Volume oder Bind Mount verwendet wird).
    ```dockerfile
    VOLUME /var/log
    VOLUME /app/data
    ```
*   **`USER <benutzer>[:<gruppe>]`:** Setzt den Benutzernamen (oder UID) und optional den Gruppennamen (oder GID), der beim Ausführen des Images und für alle nachfolgenden `RUN`, `CMD` und `ENTRYPOINT` Anweisungen verwendet wird.
    ```dockerfile
    RUN useradd -ms /bin/bash meinbenutzer
    USER meinbenutzer
    ```
*   **`ARG <name>[=<standardwert>]`:** Definiert eine Variable, die Benutzer zur Build-Zeit mit dem Flag `--build-arg <varname>=<wert>` beim `docker build`-Befehl übergeben können.
    ```dockerfile
    ARG USER=gast
    RUN echo "Baue als Benutzer: $USER"
    ```

## Dockerfile Beispiel: Einfacher Webserver

Erstelle ein Verzeichnis, z. B. `meine-simple-web`. Darin erstelle:

1.  `Dockerfile`:
    ```dockerfile
    # Verwende ein leichtgewichtiges Nginx-Image als Basis
    FROM nginx:alpine

    # Setze Maintainer-Label (optionale gute Praxis)
    LABEL maintainer="Dein Name <deine.email@example.com>"

    # Setze Arbeitsverzeichnis (optional, Nginx-Standard ist oft ok)
    WORKDIR /usr/share/nginx/html

    # Entferne die Standard-Nginx-Seite
    RUN rm index.html

    # Kopiere deine Website-Dateien aus einem lokalen 'public'-Verzeichnis
    # in das Nginx-Web-Root-Verzeichnis im Container
    COPY public/ .

    # Exponiere Port 80 (Standard-HTTP-Port, auf dem Nginx lauscht)
    EXPOSE 80

    # Standardbefehl zum Starten von Nginx, wenn der Container läuft
    # Das Basis-Nginx-Image hat bereits ein passendes CMD oder ENTRYPOINT,
    # daher ist dies möglicherweise nicht unbedingt notwendig, zeigt aber die Syntax.
    # Nginx muss für Docker im Vordergrund laufen.
    CMD ["nginx", "-g", "daemon off;"]
    ```
2.  Ein Verzeichnis namens `public`.
3.  Innerhalb von `public`, erstelle eine `index.html`-Datei:
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Meine Docker Webseite</title>
    </head>
    <body>
        <h1>Hallo aus meinem Docker Container!</h1>
        <p>Diese Webseite wird von Nginx in einem Docker Container bereitgestellt.</p>
    </body>
    </html>
    ```

**Bauen und Ausführen:**

```bash
# Navigiere im Terminal zum Verzeichnis 'meine-simple-web'
cd meine-simple-web

# Baue das Image
docker build -t meine-webseite:latest .

# Führe den Container aus
docker run -d --name meine-laufende-seite -p 8080:80 meine-webseite:latest

# Greife auf deine Seite im Browser unter http://localhost:8080 zu
```

**Aufräumen:**

```bash
# Stoppe den Container
docker stop meine-laufende-seite

# Entferne den Container
docker rm meine-laufende-seite

# Entferne das Image (optional)
docker rmi meine-webseite:latest

# Entferne ungenutzte Volumes (optional)
docker volume prune -f

# Entferne ungenutzte Netzwerke (optional)
docker network prune -f

# Entferne alle ungenutzten Images, Container, Netzwerke, Build-Cache (vorsichtig verwenden)
# docker system prune -a -f --volumes
```
