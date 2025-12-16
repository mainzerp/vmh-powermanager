# Häufig gestellte Fragen - VM Host Power Manager

## Allgemeines

### Q: Was ist VM Host Power Manager?
**A:** Ein automatisiertes System, das VMs und Bare-Metal-Hosts kontrolliert herunter- und hochfahren kann basierend auf der verbleibenden Batterielaufzeit von USV-Systemen (UPS).

### Q: Welche Virtualisierungsplattformen werden unterstützt?
**A:** 
- VMware vCenter
- Proxmox VE
- Bare-Metal Linux Hosts (via SSH/IPMI)

### Q: Welche UPS-Systeme werden unterstützt?
**A:**
- SNMP-basierte UPS (APC, Eaton, etc.)
- NUT (Network UPS Tools)

## Installation & Setup

### Q: Wie installiere ich die Anwendung?
**A:** Mit Docker Compose:
```bash
git clone https://github.com/mainzerp/vm-host-power-manager
cd vm-host-power-manager
docker-compose up -d
```

### Q: Welche Ports werden benötigt?
**A:**
- **3000**: Frontend
- **8000**: Backend API
- **5432**: PostgreSQL
- **6379**: Redis

### Q: Kann ich custom Ports verwenden?
**A:** Ja, passe `docker-compose.yml` an:
```yaml
ports:
  - "9000:3000"  # Custom Frontend Port
  - "9001:8000"  # Custom Backend Port
```

## Konfiguration

### Q: Wie füge ich ein SNMP UPS-Gerät hinzu?
**A:**
1. Gehe zu "UPS Devices" im Dashboard
2. Klicke "Add Device"
3. Wähle "SNMP"
4. Gebe Hostname/IP, Community String (default: "public") ein
5. Optional: Custom OIDs für Batteriezeit, Ladung, etc.

### Q: Wie erstelle ich eine Shutdown-Regel?
**A:**
1. Gehe zu "Rules"
2. Klicke "Create Rule"
3. Definiere Name, Priorität, Systeme
4. Speichere
5. Verwende "Test" für Dry-Run

### Q: Was ist eine Shutdown-Abhängigkeit?
**A:** Eine Abhängigkeit definiert, dass System A vor System B heruntergefahren werden muss. Z.B. müssen Datenbanken vor Applikationsservern heruntergefahren werden.

## Monitoring & Alerts

### Q: Wie oft werden UPS-Geräte überwacht?
**A:** Standard: Alle 30 Sekunden (konfigurierbar via `UPS_POLL_INTERVAL`)

### Q: Kann ich Benachrichtigungen konfigurieren?
**A:** Ja, zwei Kanäle:
1. **E-Mail**: Direkt konfigurierbar
2. **Webhook**: Für Integration mit Slack, Teams, etc.

### Q: Welche Warnstufen gibt es?
**A:**
- **info**: Informationen
- **warning**: Warnung (z.B. Batterie < 15 Min)
- **critical**: Kritisch (z.B. Batterie < 5 Min)

## Troubleshooting

### Q: Die Datenbank wird nicht initialisiert
**A:** 
```bash
docker-compose down
docker-compose up -d postgres
docker exec vmpm-postgres psql -U vmpm_user -d vmpm_db -f /docker-entrypoint-initdb.d/init.sql
docker-compose up -d
```

### Q: Kann nicht auf API zugreifen
**A:**
1. Überprüfe ob Backend läuft: `docker-compose logs backend`
2. Überprüfe Firewall: Port 8000 offen?
3. Überprüfe Health: `curl http://localhost:8000/api/health`

### Q: Login funktioniert nicht
**A:**
1. Registriere neuen Benutzer
2. Überprüfe Datenbank-Verbindung
3. Überprüfe JWT_SECRET in Backend

### Q: UPS-Gerät wird nicht erkannt
**A:**
1. Überprüfe SNMP Erreichbarkeit: `snmpwalk -v 2c -c public 192.168.1.100`
2. Überprüfe Community String
3. Überprüfe OID Konfiguration
4. Überprüfe Firewall (Port 161 UDP)

## Performance

### Q: Wie viele Systeme kann ich verwalten?
**A:** Theoretisch unbegrenzt. Performance hängt ab von:
- Datenbankleistung
- Netzwerkbandbreite
- Monitoring-Intervall

### Q: Sollte ich Monitoring-Intervall erhöhen?
**A:** Für große Umgebungen ja:
- Standard: 30s
- Empfohlen (100+ Systeme): 60-120s

### Q: Kann ich mehrere Backend-Instanzen haben?
**A:** Ja, mit Load Balancer (nginx, traefik)

## Security

### Q: Wie sind Passwörter gespeichert?
**A:** Mit bcrypt gehasht (nicht reversibel)

### Q: Sind Credentials verschlüsselt?
**A:** Ja, UPS/Platform Credentials werden verschlüsselt gespeichert

### Q: Kann ich TLS/SSL aktivieren?
**A:** Ja, konfiguriere einen Reverse-Proxy (nginx)

### Q: Wie lange sind Sessions gültig?
**A:** 30 Minuten (konfigurierbar via `ACCESS_TOKEN_EXPIRE_MINUTES`)

## Wartung

### Q: Wie mache ich ein Backup?
**A:**
```bash
# Datenbank Backup
docker exec vmpm-postgres pg_dump -U vmpm_user vmpm_db > backup.sql

# Volumes Backup
docker run --volumes-from vmpm-postgres -v $(pwd):/backup alpine tar czf /backup/postgres_backup.tar.gz /var/lib/postgresql/data
```

### Q: Wie recover ich aus einem Backup?
**A:**
```bash
# Datenbank restore
docker exec -i vmpm-postgres psql -U vmpm_user vmpm_db < backup.sql
```

### Q: Wie update ich auf eine neue Version?
**A:**
```bash
git pull origin main
docker-compose down
docker-compose up -d --build
```

## Development

### Q: Kann ich custom Integrationen schreiben?
**A:** Ja, folge der [DEVELOPMENT.md](DEVELOPMENT.md) Anleitung

### Q: Wie füge ich neue UPS-Protokolle hinzu?
**A:** 
1. Implementiere in `backend/app/services/monitoring.py`
2. Füge neue Device Type in Models hinzu
3. Teste mit Unit Tests

## Support

### Q: Wo finde ich weitere Hilfe?
**A:**
- GitHub Issues: https://github.com/mainzerp/vm-host-power-manager/issues
- Dokumentation: Siehe [README_DE.md](README_DE.md)
- API Referenz: Siehe [API_REFERENCE.md](API_REFERENCE.md)

### Q: Kann ich einen Bug melden?
**A:** Ja, bitte erstelle einen Issue mit:
- Fehlermeldung
- Docker Logs
- Reproduktionsschritte

### Q: Kann ich Feature-Requests machen?
**A:** Ja, nutze GitHub Discussions oder Issues mit Tag "feature-request"

---

**Letzte Aktualisierung**: 2025-01-11
