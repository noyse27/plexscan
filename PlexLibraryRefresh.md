# Plex Library Refresh Script

Dieses PowerShell-Skript ermöglicht es Ihnen, eine Bibliothek in Ihrem Plex-Server auszuwählen und zu aktualisieren. Sie können auch einen Pfad auswählen, um nur bestimmte Medien zu aktualisieren.

## Voraussetzungen

- Windows-Betriebssystem
- PowerShell 5.0 oder höher
- Netzwerkzugriff auf den Plex-Server

## Verwendung

1. Klonen Sie dieses Repository oder laden Sie die Skript-Datei herunter.
2. Öffnen Sie PowerShell und navigieren Sie zum Speicherort des Skripts.
3. Führen Sie das Skript mit dem Befehl `.\PlexLibraryRefresh.ps1` aus.

## Skriptinhalt

```PowerShell
# Initiale Variablen
$plexip = ""   # IP-Adresse Ihres Plex-Servers
$token = ""  # Plex-Token für die Authentifizierung

# Funktion zur Auswahl der Bibliothek
function Get-LibraryChoice {
    $menuOptions = @{
        "1"  = @("Filme", 1)
        "2"  = @("Serien", 2)
    }
    Write-Host "Bitte wählen Sie eine Bibliothek aus:"
    foreach ($key in ($menuOptions.Keys | Sort-Object {[int]$_})) {
        Write-Host "$key - $($menuOptions[$key][0])"
    }
    $choice = Read-Host "Ihre Auswahl"
    if ($menuOptions.ContainsKey($choice)) {
        return $menuOptions[$choice][1]
    } elseif ($choice -eq "") {
        return "exit"
    } else {
        Write-Host "Ungültige Auswahl."
        return $null
    }
}

# Funktion zur Auswahl des Pfads
function Get-PathChoice {
    $pathOptions = @{
        "1"  = "Y:\Filme\1080"
        "2"  = "Y:\Filme\720"
    }
    Write-Host "Bitte wählen Sie einen Pfad aus (Drücken Sie Enter für keinen Pfad oder ESC für das vorherige Menü):"
    foreach ($key in ($pathOptions.Keys | Sort-Object {[int]$_})) {
        Write-Host "$key - $($pathOptions[$key])"
    }
    $choice = Read-Host "Ihre Auswahl (ESC zum Abbrechen)"
    if ($choice -eq "") {
        return ""
    } elseif ($choice -eq "ESC") {
        return "esc"
    } elseif ($pathOptions.ContainsKey($choice)) {
        return $pathOptions[$choice]
    } else {
        Write-Host "Ungültige Auswahl."
        return $null
    }
}

# Hauptlogik
while ($true) {
    $library = Get-LibraryChoice
    if ($library -eq "exit" -or $library -eq $null) {
        Write-Host "Keine gültige Bibliothek gewählt."
        break
    } else {
        Write-Host "Sie haben die Bibliothek mit dem Wert $library gewählt."
        if ($library -eq 1) {
            while ($true) {
                $path = Get-PathChoice
                if ($path -eq "esc") {
                    break
                } elseif ($path -ne $null) {
                    Write-Host "Sie haben den Pfad gewählt: $path"
                    break
                } else {
                    Write-Host "Kein gültiger Pfad ausgewählt."
                }
            }
        } else {
            $path = ""
        }

        # URL erstellen
        $url = "http://"
        $url += $plexip
        $url += ":32400/library/sections/$library/refresh?"
        if ($path -ne '') {
            $url += "path=$path&"
        }
        $url += "X-Plex-Token=$token"
        Write-Host "Die erzeugte URL ist: $url"

        # URL abschicken
        try {
            $response = Invoke-WebRequest -Uri $url -UseBasicParsing
            if ($response.StatusCode -eq 200) {
                Write-Host "Anfrage erfolgreich gesendet."
            } else {
                Write-Host "Fehler beim Senden der Anfrage. Statuscode: $($response.StatusCode)"
            }
        } catch {
            Write-Host "Fehler beim Senden der Anfrage: $_"
        }
        Write-Host "Drücken Sie eine beliebige Taste, um das Fenster zu schließen."
        $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
        break
    }
}
```

## Tipps

- Stellen Sie sicher, dass die IP-Adresse des Plex-Servers korrekt ist.
- Ersetzen Sie das Plex-Token durch Ihres.
- Fügen Sie nach Bedarf weitere Bibliotheken oder Pfade hinzu.

## Lizenz

Dieses Projekt ist lizenziert unter der [MIT-Lizenz](LICENSE).