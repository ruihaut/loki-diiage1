# 1. Installation et configuration de Grafana

Le service Grafana est utilisé pour visualiser de la donnée (des métrics ou des logs). Dans notre contexte, il est utilisé uniquement pour lire les logs stockés dans Loki. La connexion au service se fera à la fin.

Dans cette documentation, nous allons installer et configurer le package Grafana.

## Prérequis

- Disposer d'un serveur Linux avec une distribution compatible (Debian ou Ubuntu)
- Être connecté à ce serveur en root ou, à défaut, avec un utilisateur avec les droits sudo
- Avoir installé les packages `zip` et `curl` :

    ```bash
    apt update && apt install -y curl zip
    ```

## Installation du service via apt

1. Commencer par installer les services requis pour installer le dépôt officiel de Grafana :

    ```bash
    apt update && apt install -y apt-transport-https software-properties-common gnupg gnupg2 wget
    ```

2. Installer la clé publique du dépôt de Grafana :

    ```bash
    sudo mkdir -p /etc/apt/keyrings/
    wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | tee /etc/apt/keyrings/grafana.gpg > /dev/null
    ```

3. Ajout du dépôt en spécifiant la clé téléchargée précédemment :

    ```bash
    echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | tee -a /etc/apt/sources.list.d/grafana.list
    ```

4. Mettre à jour les apt et installer Grafana :

    ```bash
    apt update && apt install grafana
    ```

5. Activer le daemon et lancer le service Grafana

    ```bash
    systemctl daemon-reload
    systemctl enable grafana-server
    systemctl start grafana-server
    ```

6. Vérifier que le service est up :

    ```bash
    systemctl status grafana-server
    ```

Le service est maintenant accessible sur le port `3000` de la machine.
