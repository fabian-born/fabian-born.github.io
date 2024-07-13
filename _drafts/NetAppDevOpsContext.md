---
title: "NetApp im DevOps Kontext"
date:  2021-12-20T07:00:00+02:00
draft: true
categories: ["NetApp","Docker","Kubernetes","Trident","DevOps"]
banner: /img/ka.png
layout: post
---

# Einleitung
In der Cloud gibt es verschiedene Möglichkeiten Kubernetes Cluster zu erstellen und zu betreiben. Dieser Artikel beschreibt, wie man zusammen mit den native Cloudprovider Serives, AKS in Microsoft Azure, GKE in der Google Cloud Plattform und EKS in Amazon Web Service, eine optimale Architektur einer Kubernetes Umgebung gestalten kann. Für den Aufbau so einer Umgebung werden viele verschiedene Tools namentlich benannt und auch in den verschiedenen Codezeilen genutzt. Auch wird man feststellen, das zum heutigen Stand nicht alles bei allen Cloudprovidern genutzt werden kann.


# Architektur
Wie sieht die optimale Architektur für eine solche Kubernetes Infrastruktur aus? Zu erst wirft man einen Blick auf die Architektur - ohne sich auf den Cloud Provider festzulegen. neben den Kubernetes Nodes wird Storage für die persitenten Volumes benötigt. Idealerweise gibt es eine Backup Intergration und auch die Möglichkeit Kosteneffizient die Cluster zu betreiben.

Das Schaubild zeigt den Architekturaufbau. Kubernetesnodes bilden die Basis. 
Die persitant Volumes (PV) werden auf NetApp Cloud Volumes Service für AWS/GCP oder Azure NetApp Files angelegt. Bei allen drei Storage Möglichkeiten steht ein NetApp ONTAP mit all seinen bekannten Funktionen, wie SnapShots, Cloning, Storage Efficiency, automatiesierbarkeit dahinter. Für das automatisierte povisionieren und mappen von PVs wird der Provisioner Astra Trident genutzt. 

