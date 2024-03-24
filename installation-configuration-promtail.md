# 3. Installation et configuration de Promtail

Dans cette documentation, nous allons installer et configurer Promtail.

Toutes les étapes suivies sont globalement les mêmes que pour l'installation de Loki.

## Prérequis

- Disposer d'un serveur Linux avec une distribution compatible
- Être connecté à ce serveur en root ou, à défaut, avec un utilisateur avec les droits sudo
- Avoir installé les packages `zip` et `curl` :

    ```bash
    apt update && apt install -y curl zip
    ```

## Installation du binaire

1. Se rendre sur le dépôt officiel de Loki, dans la section [Releases](https://github.com/grafana/loki/releases/)
2. Repérer la version de Loki que l'on souhaite installer. Dans notre cas, il s'agit de la version `2.9.4`
3. Effectuer la commande `curl` suivante avec la version du package correspondant :

    ```bash
    curl -O -L "https://github.com/grafana/loki/releases/download/v2.9.4/promtail-linux-amd64.zip"
    ```

4. Dézipper le zip téléchargé dans le répertoire `/usr/local/bin/` :

    ```bash
    unzip -d /usr/local/bin/ promtail-linux-amd64.zip
    ```

5. Une fois le binaire dézippé, il faudra le renommer en `promtail` :

    ```bash
    mv /usr/local/bin/promtail-linux-amd64  /usr/local/bin/promtail
    ```

6. Rajouter des droits d'exécution au binaire avec la commande `chmod` :

    ```bash
    chmod +x /usr/local/bin/promtail
    ```

Le binaire est maintenant installé et prêt à être utilisé. La prochaine étape consistera à créer le fichier de configuration nécessaire à Promtail.

## Création du fichier de configuration

1. Créer le répertoire `/etc/promtail` et s'y déplacer :

    ```bash
    mkdir /etc/promtail && cd /etc/promtail
    ```

2. Récupérer la dernière version du fichier de configuration de Loki en le renommant en `loki.yaml` :

    ```bash
    wget -O promtail.yaml https://raw.githubusercontent.com/grafana/loki/main/clients/cmd/promtail/promtail-local-config.yaml
    ```

    > **Note**  
    > L'option `-O` permet de specifierr l'**O**utput file

3. Editer le fichier `/etc/loki/loki.yaml`. A la fin du fichier, rajouter le bloc de configuration suivant :

    ```yaml
    [...]
    - job_name: syslog
      syslog:
        listen_address: 0.0.0.0:1514
        listen_protocol: tcp
        idle_timeout: 12h
        labels:
          job: syslog
      relabel_configs:
        - source_labels: [__syslog_message_hostname]
          target_label: host
        - source_labels: [__syslog_message_severity]
          target_label: severity
        - source_labels: [__syslog_message_app_name]
          target_label: application
        - source_labels: [__syslog_message_facility]
          target_label: facility
    ```

    > **Note**  
    > Ici, on renomme les labels récupérés de syslog pour qu'ils soient envoyés correctement nommés à Loki.

4. Sauvegarder le fichier.

Le fichier de configuration est maintenant prêt à être utilisé par Promtail. La prochaine étape consistera à créer le service pour Promtail avec `systemd`.

## Création du fichier service

1. Création de l'utilisateur système :

    ```bash
    useradd --system promtail
    ```

2. Modifier le propriétaire du dossier `/etc/promtail` par promtail :

    ```bash
    chown promtail: /etc/promtail -R
    ```

3. Editer le fichier `nano /etc/systemd/system/promtail.service` :

    ```bash
    nano /etc/systemd/system/promtail.service
    ```

4. Renseigner les informations suivantes :

    ```conf
    [Unit]
    Description=Promtail Service
    After=network.target

    [Service]
    Type=simple
    User=promtail
    ExecStart=/usr/local/bin/promtail -config.file /etc/promtail/promtail.yaml

    [Install]
    WantedBy=multi-user.target
    ```

    > **Note**  
    > Faire correspondre la ligne `ExecStart` aux noms de fichiers donnés s'ils ne sont pas comme indiqué dans la procédure.

5. Activer et démarrer le service :

    ```bash
    systemctl enable promtail & systemctl start promtail
    ```

6. Vérifier l'état du service :

    ```bash
    systemctl status promtail
    ```

    > **Note**  
    > En cas de problème, vérifier le fichier `/etc/systemd/system/promtail.service` créé précédemment. En cas de modification de ce fichier, effectuer la commande `systemctl deamon-reload` pour que les changements prennent effet.

Le service est désormais prêt à être utilisé.
