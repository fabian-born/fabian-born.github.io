---
title: "UPDATE: Datacollector für HomeMatic und Grafana mit PoSH"
date:  2018-09-04
draft: false
banner: "/assets/images/content/grafana-overview.png"
categories: ["homematic","automation","Powershell"]
layout: post
---

## Update: Jetzt mit Powershell
Nach dem das Powershell Skript eine weile lief, habe ich mich daran gemacht, es einwenig umzubauen. Anforderung war, dass es in bestimmten Abständen laufen soll, aber man hierfür nicht die Cron von Linux nehmen muss. Ich habe mich dann Entschieden Powershell 6 Core zu verweden. 

## Wie bekomme ich die Daten aus der CCU?
Die Daten werden aus den verschiedenen URLS 

 + http://ip/config/xmlapi/roomlist.cgi
  - Auflistung der Rooms
 + http://ip/config/xmlapi/state.cgi
  - aktueller State aller angelernter Geräte
  - Sortierbar nach z.B. channel_id oder device_id

in ein XML Array geladen und dann verarbeitet. 

Sobald man die gewünschten Daten zur Verfügung hat, muss man sie an Grafana/Graphite schicken. Hierzu möchte ich nun meine Powershell Funktion zur Verfügung stellen:
```posh
function Send-ToGraphite {
    param(
        [string]$carbonServer,
        [string]$carbonServerPort,
        [string[]]$metrics
    )
      try
        {
        $socket = New-Object System.Net.Sockets.TCPClient
        $socket.connect($carbonServer, $carbonServerPort)
        $stream = $socket.GetStream() 
        $writer = new-object System.IO.StreamWriter($stream)
        foreach ($metric in $metrics){
          #  Write-Host $metric
            $newMetric = $metric.TrimEnd()
           $writer.WriteLine($newMetric)
            }
        $writer.Flush()
        $writer.Close()
        $stream.Close()
        }
        catch
        {   
            Write-Error $_
        }
}
```
Der Aufruf im Skript ist:
```posh
    $ Send-ToGraphite -carbonServer $carbonServer -carbonServerPort $carbonServerPort -metric "$($base).$($roomname).$($devicename).$($datapoint_name) $($datapoint_value) $($date)"
```
Für aktuellere Informationen zum Konfigurieren und ausführen des Datacollector findet man auf der gitlab-Projekt Seite:  [hm2grafana-posh]( https://github.com/fabian-born/hm2grafana-posh "hm2grafana-posh on github)



