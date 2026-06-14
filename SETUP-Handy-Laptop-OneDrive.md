# Vom Handy arbeiten – Laptop dauerhaft an – Dateien in OneDrive

Ziel: Claude Code läuft **lokal auf dem Laptop** (Windows), du steuerst es **vom Handy** per SSH,
und die Dateien liegen im **OneDrive-Ordner** `C:\Users\lucas\OneDrive\Agenten`, der
automatisch in die Cloud synchronisiert.

```
Handy (Termius-App)  ──Internet (Tailscale)──►  Laptop (zugeklappt, dauerhaft an)
                                                  ├─ Windows + OpenSSH-Server
                                                  ├─ WSL2 (Ubuntu) + Claude Code (in tmux)
                                                  └─ C:\Users\lucas\OneDrive\Agenten
                                                        │
                                                        └── automatischer Sync ──► OneDrive-Cloud
```

Arbeite die Phasen **der Reihe nach** ab. Phase 1–5 macht man **einmalig** am Laptop,
Phase 6 ist der tägliche Ablauf vom Handy.

---

## Phase 1 – Laptop bleibt beim Zuklappen an

1. `Win` drücken, „Energieplan bearbeiten" suchen, öffnen.
2. Links auf **„Auswählen, was beim Zuklappen geschehen soll"**.
3. Bei **„Beim Zuklappen des Computers"** → Spalte **„Netzbetrieb"** auf **„Nichts unternehmen"** stellen.
4. Speichern. Laptop immer **am Stromkabel** lassen.
5. Zusätzlich unter *Einstellungen → System → Netzbetrieb & Energiesparen*: Energiesparmodus im
   Netzbetrieb auf **„Niemals"**.

> Test: Laptop zuklappen, 2 Minuten warten, aufklappen – er darf nicht „aufgewacht" wirken,
> sondern einfach weiterlaufen.

---

## Phase 2 – WSL2 (Linux) installieren

Claude Code läuft am stabilsten unter Linux. WSL2 bringt Linux direkt in Windows.

1. **PowerShell als Administrator** öffnen (Rechtsklick auf Start → „Terminal (Administrator)").
2. Befehl eingeben:
   ```powershell
   wsl --install
   ```
3. Laptop **neu starten**, wenn er dazu auffordert.
4. Nach dem Neustart öffnet sich automatisch ein Ubuntu-Fenster. Es fragt nach einem
   **Benutzernamen** und **Passwort** für Linux – frei wählbar, **merken!**
   (Das Passwort sieht man beim Tippen nicht – das ist normal.)

> Falls sich kein Ubuntu öffnet: im Startmenü „Ubuntu" suchen und starten.

---

## Phase 3 – Claude Code in WSL installieren

Im **Ubuntu-Fenster** (das ist deine Linux-Kommandozeile) nacheinander eingeben:

1. System aktualisieren + Node.js installieren:
   ```bash
   sudo apt update && sudo apt upgrade -y
   curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
   sudo apt install -y nodejs
   ```
2. Prüfen, dass Node da ist (sollte eine Versionsnummer zeigen):
   ```bash
   node --version
   ```
3. Claude Code installieren:
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```
4. Erstmals starten und anmelden:
   ```bash
   claude
   ```
   Es erscheint ein Link/Code zum Anmelden mit deinem Claude-Konto – Anmeldung im Browser
   bestätigen. Danach läuft Claude Code.

---

## Phase 4 – OneDrive-Ordner aus Linux erreichbar machen

OneDrive liegt auf der Windows-Seite. Aus WSL erreichst du ihn über `/mnt/c/...`.

1. In den OneDrive-Agenten-Ordner wechseln (existiert er noch nicht, wird er erstellt):
   ```bash
   mkdir -p "/mnt/c/Users/lucas/OneDrive/Agenten"
   cd "/mnt/c/Users/lucas/OneDrive/Agenten"
   ```
2. Zum bequemen Wiederfinden eine Abkürzung anlegen:
   ```bash
   echo 'alias agenten="cd \"/mnt/c/Users/lucas/OneDrive/Agenten\""' >> ~/.bashrc
   source ~/.bashrc
   ```
   Ab jetzt bringt dich der Befehl `agenten` immer direkt dorthin.

> Wichtig: Alles, was Claude Code in diesem Ordner anlegt, synchronisiert der OneDrive-Client
> automatisch in die Cloud. Du musst dafür nichts tun – nur sicherstellen, dass OneDrive in
> Windows angemeldet und aktiv ist.

---

## Phase 5 – Fernzugriff einrichten (damit das Handy den Laptop erreicht)

### 5a) Tailscale (sicheres privates Netz, von überall erreichbar – empfohlen)

1. Am Laptop: https://tailscale.com herunterladen, installieren, mit z. B. Google-/Microsoft-Konto
   anmelden.
2. Auf dem **Handy** dieselbe Tailscale-App installieren und mit **demselben Konto** anmelden.
3. In der Tailscale-App siehst du den Laptop mit einer festen IP-Adresse (Form `100.x.y.z`).
   **Diese Adresse notieren** – darüber erreichst du den Laptop von überall, ohne Router-Einstellungen.

### 5b) OpenSSH-Server auf dem Laptop aktivieren

1. *Einstellungen → System → Optionale Features → Feature hinzufügen*.
2. **„OpenSSH-Server"** suchen, installieren.
3. SSH-Dienst dauerhaft starten – in **PowerShell (Administrator)**:
   ```powershell
   Start-Service sshd
   Set-Service -Name sshd -StartupType 'Automatic'
   ```

> Damit verbindest du dich per SSH zunächst auf **Windows**; von dort startest du mit dem
> Befehl `wsl` die Linux-Umgebung.

---

## Phase 6 – Täglicher Ablauf vom Handy

### Einmalig: Termius-App einrichten

1. Auf dem Handy **Termius** installieren (iOS/Android).
2. Neuen Host (Verbindung) anlegen:
   - **Address/Hostname**: die Tailscale-IP des Laptops (`100.x.y.z`)
   - **Username**: dein **Windows**-Benutzername (`lucas`)
   - **Password**: dein Windows-Passwort
3. Speichern.

### So arbeitest du jeden Tag

1. Tailscale auf dem Handy ist aktiv (einmal antippen, falls aus).
2. In Termius die Verbindung antippen → du bist auf dem Laptop.
3. In Linux wechseln und Sitzung öffnen/fortsetzen:
   ```bash
   wsl
   tmux attach || tmux
   ```
   (`tmux` hält die Sitzung am Leben, auch wenn das Handy die Verbindung verliert.
   `attach` setzt eine bestehende Sitzung fort, sonst wird eine neue gestartet.)
4. In den OneDrive-Ordner und Claude Code starten:
   ```bash
   agenten
   claude
   ```
5. Anweisungen eintippen. Erzeugte Dateien landen im OneDrive-Ordner → automatisch in der Cloud.

> Verbindung verloren? Einfach neu verbinden und `tmux attach` – Claude arbeitet im Hintergrund weiter.
> Sitzung bewusst verlassen, ohne sie zu beenden: Tasten `Strg`+`b`, dann `d`.

---

## Kurz-Spickzettel (Handy)

| Schritt | Befehl |
|---|---|
| In Linux wechseln | `wsl` |
| Sitzung fortsetzen/öffnen | `tmux attach \|\| tmux` |
| Zum OneDrive-Ordner | `agenten` |
| Claude Code starten | `claude` |
| Sitzung „im Hintergrund" lassen | `Strg`+`b`, dann `d` |

---

## Häufige Stolpersteine

- **Laptop nicht erreichbar:** Läuft Tailscale auf beiden Geräten und sind beide mit demselben
  Konto angemeldet? Laptop wirklich an und nicht im Energiesparmodus (Phase 1)?
- **Dateien tauchen nicht in OneDrive-Cloud auf:** Ist der OneDrive-Client in Windows angemeldet
  und der Ordner `Agenten` wirklich unter dem OneDrive-Pfad? Symbol unten rechts in Windows prüfen.
- **`claude`-Befehl nicht gefunden:** Phase 3 Schritt 3 erneut ausführen.
- **WSL/OneDrive langsam:** Bei sehr vielen kleinen Dateien kann der `/mnt/c`-Zugriff träge sein –
  für normale Dokumente/Texte aber unproblematisch.
