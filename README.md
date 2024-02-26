# Lab d'installation de la stack Loki / Grafana

## Objectifs de ce lab

- Savoir installer et configurer Grafana
- Savoir installer et configurer Loki
- Savoir installer et configurer Promtail avec rsyslog
- Savoir récupérer les logs d'un serveur Linux
- Savoir récupérer des logs via Grafana

## Introduction

Ce lab a pour but de vous guider dans l'installation d'un environnement de logging **basique**.

Cet environnement pourra, **avec prise en compte [des éléments d'attention](#points-dattention)**, être implémenté dans votre environnement de manière sécurisé.

Voici l'architecture cible :

![Architecture cible](_media/logging-architecture-cible.svg)

### Points non abordés dans ce lab

- **Sécurisation du serveur** - La sécurisation du serveur même n'est pas abordée dans ce lab

  Je vous laisse tout de même un lien vers les [recommandations de sécurité relatives à un système GNU/Linux](https://cyber.gouv.fr/publications/recommandations-de-securite-relatives-un-systeme-gnulinux) publié par l'ANSSI pour vous en inspirer

- **Sécurisation des échanges entre les serveurs** - La partie chiffrement des communications n'est pas abordée dans ce lab

  Je vous laisse pour cela prendre connaissance du lien suivant :

  - [15.5. Configuring TLS-encrypted remote logging](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/assembly_configuring-a-remote-logging-solution_security-hardening#proc_configuring-tls-encrypted-remote-logging_assembly_configuring-a-remote-logging-solution)

### Points d'attention

Dans ce lab, vous allez voir comment installer un environnement simple de logging basé sur loki et rsyslog. Cependant, **cette configuration n'est pas faite pour ingérer et exploiter un grand nombre de logs**. Ainsi, certains éléments sont à prendre en compte pour rendre cette configuration "Production ready" :

- **Loki est un processus gourmand en ressources** - Lorsque loki effectue une recherche de logs dans le temps, il consomme du CPU
- **Couple Loki / rsyslog** - Pour un environnement de production, la meilleure pratique est de séparer rsyslog et Loki.

  > **Important**  
  > Lorsque Loki effectue une recherche de logs, vu qu'il peut consommer 100% du CPU (périodiquement), il peut arriver que rsyslog soit bloqué par manque de ressources, ce qui empêche la réception des logs.

- **Espace disque** - Il faut prendre en compte le volume de logs ingérés *mais aussi* la stratégie de rétention appliquée

## Déroulé du lab

### [1. Installation et configuration de Grafana](./installation-configuration-grafana.md)

### [2. Installation et configuration de Loki](./installation-configuration-loki.md)

### [3. Installation et configuration de Promtail](./installation-configuration-promtail.md)

### [4. Configuration de rsyslog](./configuration-rsyslog.md)

### [5. Ajout et utilisation de Loki dans Grafana](./ajout-et-utilisation-loki-grafana.md)
