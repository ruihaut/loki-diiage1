# 2. Installation et configuration de Loki

Dans cette documentation, nous allons installer et configurer le package Loki.

## Prérequis

- Disposer d'un serveur Linux avec une distribution compatible (Debian ou Ubuntu)
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
    curl -O -L "https://github.com/grafana/loki/releases/download/v2.9.4/loki-linux-amd64.zip"
    ```

4. Dézipper le zip téléchargé dans le répertoire `/usr/local/bin/` :

    ```bash
    unzip -d /usr/local/bin/ loki-linux-amd64.zip
    ```

5. Une fois le binaire dézippé, il faudra le renommer en `loki` :

    ```bash
    mv /usr/local/bin/loki-linux-amd64  /usr/local/bin/loki
    ```

6. Rajouter des droits d'exécution au binaire avec la commande `chmod` :

    ```bash
    chmod +x /usr/local/bin/loki
    ```

Le binaire est maintenant installé et prêt à être utilisé. La prochaine étape consistera à créer le fichier de configuration nécessaire à Loki.

## Création du fichier de configuration

1. Créer le répertoire `/etc/loki` et s'y déplacer :

    ```bash
    mkdir /etc/loki && cd /etc/loki
    ```

2. Récupérer la dernière version du fichier de configuration de Loki en le renommant en `loki.yaml` :

    ```bash
    wget -O loki.yaml https://raw.githubusercontent.com/grafana/loki/main/cmd/loki/loki-local-config.yaml
    ```

    > **Note**  
    > L'option `-O` permet de specifierr l'**O**utput file

3. Créer un répertoire `/data/loki` :

    ```bash
    mkdir /data/loki -p
    ```

4. Editer le fichier `loki.yaml` et modifier le répertoire utilisé par Loki pour stocker ses données par `/data/loki` :

    ```yaml
    common:
      instance_addr: 127.0.0.1
      path_prefix: /data/loki
      storage:
        filesystem:
          chunks_directory: /data/loki/chunks
          rules_directory: /data/loki/rules
    ```

5. A la fin du fichier, renseigner les éléments suivants pour configurer la rétention :

    ```yaml
    compactor:
      working_directory: /data/loki/retention
      compaction_interval: 10m
      retention_enabled: true
      retention_delete_delay: 2h
      retention_delete_worker_count: 150
    storage_config:
        boltdb_shipper:
            active_index_directory: /data/loki/index
            cache_location: /data/loki/boltdb-cache

    limits_config:
      retention_period: 180d
    #  retention_stream:
    #  - selector: '{job="syslog"}'
    #    priority: 1
    #    period: 24h
    ```

    > **Note**  
    > La dernière partie commentée défini la période de rétention à 24h pour les flux correspondant au label `{job="syslog"}`

6. Sauvegarder le fichier.

Le fichier de configuration est maintenant prêt à être utilisé par Loki. La prochaine étape consistera à créer le service pour Loki avec `systemd`.

## Création du fichier service et du compte de service dédié

1. Création de l'utilisateur système :

    ```bash
    useradd --system loki
    ```

2. Modifier le propriétaire des dossiers `/data/loki` et `/etc/loki` par loki :

    ```bash
    chown loki: /data/loki -R && chown loki: /etc/loki -R
    ```

3. Editer le fichier `nano /etc/systemd/system/loki.service` :

    ```bash
    nano /etc/systemd/system/loki.service
    ```

4. Renseigner les informations suivantes :

    ```conf
    [Unit]
    Description=Loki Service
    After=network.target

    [Service]
    Type=simple
    User=loki
    ExecStart=/usr/local/bin/loki -config.file /etc/loki/loki.yaml

    [Install]
    WantedBy=multi-user.target
    ```

    > **Note**  
    > Faire correspondre la ligne `ExecStart` aux noms de fichiers donnés s'ils ne sont pas comme indiqué dans la procédure.

5. Activer et démarrer le service :

    ```bash
    systemctl enable loki & systemctl start loki
    ```

6. Vérifier l'état du service :

    ```bash
    systemctl status loki
    ```

    > **Note**  
    > En cas de problème, vérifier le fichier `/etc/systemd/system/loki.service` créé précédemment. En cas de modification de ce fichier, effectuer la commande `systemctl deamon-reload` pour que les changements prennent effet.

Le service est désormais prêt à être utilisé.
