---
title: "Datacollector für HomeMatic und Grafana"
date:  2017-03-31
draft: false
banner: "/assets/images/content/grafana-overview.png"
categories: ["homematic","automation"]
layout: post
---

Wer kennt das nicht. Heizt die Heizung richtig? Wie verhält sich die Raumtemperatur? Wie sieht es mit der Luftfeutigkeit im Zimmer aus?

Homematic bietet zwar die Möglichkeit Werte in einem Graphen darzustellen. Das war für mich allerdings sehr unbefriedigend. Und eine andere Lösung muss her:

## Visualisierung mit Grafana
Grafana bietet die beste Möglichkeit, Werte zur visualisieren. Dies setzt neben einer lauffähigen Grafana Instanz auch einen Datacollector voraus, der die Daten von der Homematic CCU2 ausliest, verarbeitet und an Grafana sendet.

Auch langes Suchen, brachte keinen Ergebnisse. So habe ich mich dazu entschlossen, einen eigenen Datacollector zu schreiben: [hm2grafana]( https://github.com/fabian-born/hm2grafana "hm2grafana on GitHub")

## Wie funktioniert der Datacollector?
Das Script liest via XML-RPC API die Werte von der Homematic aus, konvertiert die Daten für Grafana und sendet sie anschließend an Graphite Server.

In der ersten Version kann der Datacollector manuell ausgeführt werden

	$ ./hm2grafana.py -c conf/hm2grafana.conf

oder als Cron-Job

	$ */5 * * * * <path-to-hm2grafana>/hm2grafana.py -c <path-to-hm2grafana>conf/hm-collector.conf > /dev/null

Folgende Zeilen können im Config-File angepasst werden:

	[general]
	ccu = ccu2-fqdn
	username =
	password =
	loglevel = INFO
	# Graphite Server + Port|

	[grafana]
	graphiteserver = graphite-fqdn
	graphiteport = 2003
	grafanaroot = homematic

Für aktuellere Informationen zum Konfigurieren und ausführen des Datacollector findet man auf der GitHub-Projekt Seite:  [hm2grafana]( https://github.com/fabian-born/hm2grafana "hm2grafana on gitlab")

