# PowerShell-PC-Inventar
Powershellskript zur automatisierten Inventarisierung von Hardwarekomponenten innerhalb einer Netzwerkumgebung

# PowerShell CSV-PC-Inventar

Dieses PowerShell-Skript erfasst das Datum der Bestandsaufnahme, die IP- und MAC-Adresse, die Seriennummer, das Modell, die CPU, den RAM, die Gesamtspeichergröße, die GPU(s), das Betriebssystem, den Betriebssystem-Build, den angemeldeten Benutzer und die angeschlossenen Monitore ein Computer.

Nachdem diese Informationen gesammelt wurden, werden sie in einer CSV-Datei ausgegeben. Zunächst wird die CSV-Datei (sofern vorhanden) überprüft, um festzustellen, ob der Hostname bereits in der Datei vorhanden ist.

Wenn der Hostname in der CSV-Datei vorhanden ist, wird er mit den neuesten Informationen überschrieben, sodass das Inventar auf dem neuesten Stand ist und keine doppelten Informationen vorhanden sind. Es ist für die Ausführung als Anmeldeskript und/oder als geplante/unmittelbare Aufgabe durch einen Domänenbenutzer konzipiert. Erweiterte Berechtigungen sind nicht erforderlich.


# Verteilung des Scripts
Standardmäßig sind die CSV-Datei und das Fehlerprotokoll so eingestellt, dass sie in das aktuelle Arbeitsverzeichnis ($pwd) schreiben. Es wird jedoch empfohlen, diesen Wert auf einen Speicherort festzulegen, an dem alle Benutzer die volle Kontrolle über die CSV-Datei haben.

Wenn die CSV-Datei nicht vorhanden ist, versucht das Skript, sie zu erstellen und automatisch die erforderlichen Berechtigungen festzulegen. Es wird jedoch empfohlen, die CSV-Datei (standardmäßig „Inventory.csv“) zunächst manuell zu erstellen und sicherzustellen, dass die richtigen Berechtigungen festgelegt sind Es.

Hier sind einige Möglichkeiten, dieses Skript bereitzustellen:

- mit einer GPO die das PowerShell Skript beim Anmelden auführt 
- Eine geplante Aufgabe
- Eine unmittelbare Aufgabe
- Fügen es zu „shell:startup“ hinzu


_________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________



# Mögliche Fragen und Antworten

### Was passiert, wenn ich mehr als zwei GPUs oder mehr als drei Monitore habe?
Das Skript kann leicht geändert werden, um diese Komponenten zu mit zu erfassen

Wenn man beispielsweise über drei GPUs verfügt, ändere einfach diesen Teil des Skripts so, dass er Folgendes enthält:

       $GPU0 = GetGPUInfo | Select-Object-Index 0
       $GPU1 = GetGPUInfo | Select-Object-Index 1
       $GPU2 = GetGPUInfo | Select-Object-Index 2

Und füge Folgendes zum OutputToCSV-Funktionsteil des Skripts hinzu:

     Add-Member -InputObject $infoObject -MemberType NoteProperty -Name "GPU 1" -Value $GPU1

Die gleiche Logik gilt für den Monitorteil des Skripts. Wenn Sie vier Monitore haben, fügen Sie Folgendes hinzu:

     $Monitor4 = GetMonitorInfo | Select-Object-Index 9,10
     $Monitor4SN = GetMonitorInfo | Select-Object-Index 11
     ...
     $Monitor4 = $Monitor4 -join ''
     ...
Add-Member -InputObject $infoObject -MemberType NoteProperty -Name "Monitor 4" -Value $Monitor4
     Add-Member -InputObject $infoObject -MemberType NoteProperty -Name "Monitor 4 Seriennummer" -Value $Monitor4SN

### Einige Systeme melden Fehler, aber für die Inventardatei ist die vollständige Kontrolle aktiviert
Dies wird wahrscheinlich durch ein anderes System verursacht, das gleichzeitig die Bestandsaufnahme ausführt, wodurch die CSV-Datei für die Bearbeitung gesperrt wird. Dies sollte kein Problem darstellen, solange das System die Möglichkeit hat, das Skript erneut auszuführen.

### Wie genau funktioniert das Skript?
Es funktioniert in der folgenden Reihenfolge:
- Zunächst werden mithilfe von WMI Informationen über das System gesammelt.
- Es wird geprüft, ob die CSV-Datei bereits vorhanden ist. Wenn ja, wird es weitergehen. Wenn nicht, wird es erstellt.
- Als nächstes wird die CSV-Datei überprüft, um festzustellen, ob der Hostname des Systems bereits existiert. Wenn dies der Fall ist, wird diese Zeile gelöscht, damit sie mit den neuesten Informationen gefüllt werden kann.
- Zuletzt werden die neuesten Systeminformationen in die CSV-Datei geschrieben.

### Welche Version von PowerShell ist erforderlich?
PowerShell 3 oder höher ist erforderlich. Ich habe es unter Windows 7 mit PowerShell 3 und Windows 10 mit PowerShell 5.1 getestet und es hat auf beiden funktioniert.

### Soll ich das Gruppenrichtlinienobjekt als Computerkonfiguration oder als Benutzerkonfiguration bereitstellen?
In meinen Tests hat es nur unter Benutzerkonfiguration funktioniert. Wenn Sie es mit der Computerkonfiguration geschafft haben, es zum Laufen zu bringen, würde mich interessieren, wie Sie es gemacht haben.
