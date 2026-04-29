# EPR Hub – Benutzerverwaltung

Die Seite `https://ki-textil-mode.de/epr/` ist mit nginx HTTP Basic Auth geschützt.
Beim Aufruf erscheint ein Browser-Login-Dialog (Benutzername + Passwort).

## Aktuelle Benutzer

| Benutzername  | Person              |
| ------------- | ------------------- |
| `jonas`       | Jonas Stracke       |
| `mschuckert`  | Michael Schuckert   |

> Passwörter werden ausschließlich als bcrypt-Hashes auf dem Server gespeichert
> (`/etc/nginx/.htpasswd-epr`, Modus `640`, Eigentümer `root:www-data`).

## Pflege (auf dem VPS, als root)

Voraussetzung: SSH-Zugang `root@srv1400170.hstgr.cloud` und Tool `htpasswd`
(im Paket `apache2-utils`, bereits installiert).

```bash
# Datei ansehen (zeigt Hashes, keine Klartext-Passwörter)
cat /etc/nginx/.htpasswd-epr

# Neuen User anlegen oder Passwort ändern (bcrypt via -B, interaktive Eingabe)
htpasswd -B /etc/nginx/.htpasswd-epr <benutzername>

# Wenn die Datei noch nicht existiert (erster User), zusätzlich -c verwenden:
htpasswd -cB /etc/nginx/.htpasswd-epr <benutzername>

# User entfernen
htpasswd -D /etc/nginx/.htpasswd-epr <benutzername>

# Berechtigungen nach Änderung sicherstellen
chmod 640 /etc/nginx/.htpasswd-epr
chown root:www-data /etc/nginx/.htpasswd-epr

# Kein nginx-Reload nötig – die Datei wird bei jedem Request neu gelesen.
```

## Nginx-Konfiguration (Referenz)

In `/etc/nginx/sites-available/ki-textil-mode.de`:

```nginx
location /epr/ {
    alias /var/www/epr/;
    index index.html;
    try_files $uri $uri/ /epr/index.html;
    add_header Cache-Control "no-cache, must-revalidate";
    auth_basic "Textile EPR";
    auth_basic_user_file /etc/nginx/.htpasswd-epr;
}
```

## Tests

```bash
# Erfolgreich (200)
curl -sI -u 'jonas:<pw>' https://ki-textil-mode.de/epr/

# Ohne Auth (401)
curl -sI https://ki-textil-mode.de/epr/
```

## Backup

Vorgängerversion der htpasswd-Datei (vor Umstellung auf bcrypt) liegt als
Backup auf dem VPS unter `/root/.htpasswd-epr.bak-2026-04-29`.
