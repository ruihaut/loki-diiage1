# 4. Configuration de rsyslog

Le service rsyslog est configuré de sorte à recevoir les logs de manière chiffrée et non chiffrée.

Pour des raisons techniques, la réception des logs se fera par les protocoles TCP et UDP.

## Ajouter la configuration du relai Promtail

L'architecture retenue requiert la configuration d'un relai pour que les logs reçus par rsyslog puissent être transférés à Promtail.

1. Editer un nouveau fichier de configuration `/etc/rsyslog.d/promtail-relay.conf` :

    ```bash
    nano /etc/rsyslog.d/promtail-relay.conf
    ```

2. Renseigner les lignes suivantes dans ce fichier :

    ```bash
    ruleset(name="remote"){
        # https://grafana.com/docs/loki/latest/clients/promtail/scraping/#rsyslog-output-configuration
        
        action(
            type="omfwd"
            protocol="tcp"
            target="localhost"
            port="1514"
            Template="RSYSLOG_SyslogProtocol23Format"
            TCP_Framing="octet-counted"
            KeepAlive="on"
        )
    }
    ```

3. Sauvegarder le fichier.

## Ajout des éléments de logging dans la configuration rsyslog

Editer le fichier `/etc/rsyslog.d/remote-logging-tcp.conf` (nouveau fichier) :

```bash
nano /etc/rsyslog.d/remote-logging-tcp.conf
```

Rajouter ensuite les éléments de configuration suivants :

```conf
## Tous les messages concernant l'authentification
auth,authpriv.*                 @@(z4)192.168.0.1:514

## Tous les messages en warn et plus sont envoyes dans syslog
*.warn;auth,authpriv.none       @@(z4)192.168.0.1:514

## Tous les messages kern sont envoyes dans kern
kern.*                          @@(z4)192.168.0.1:514

## Tous les messages user sont envoyes dans user
user.*                          @@(z4)192.168.0.1:514

## Tous les autres messages qui ne matchent pas plus haut
## Specificite : on ne les envoie pas au serveur syslog
*.=info;*.=notice;\
        auth,authpriv.none;\
        cron,daemon.none;\
        mail.none               -/var/log/messages

## Ligne speciale : tous les messages emerg sont envoyes aux terminaux ouverts
*.emerg                         @@(z4)192.168.0.1:514
```

> **Note**  
> Cette configuration est la même que pour les clients rsyslog pour avoir une configuration unifiée.  
> Les certificats renseignés dans la configuration vont être créés dans la section suivante.

### Création du fichier de configuration TCP

Maintenant que l'envoie des logs est configuré, il faut appliquer les éléments de configuration au serveur rsyslog.

1. Editer le nouveau fichier `/etc/rsyslog.d/tcp.conf` :

    ```bash
    nano /etc/rsyslog.d/tcp.conf
    ```

2. Renseigner les lignes suivantes :

    ```bash
    # Disabling excaping characters (#015, #011)
    $EscapeControlCharactersOnReceive off

    # Configure the TCP listener
    module(
        load="imtcp"
        KeepAlive="on"
        NotifyOnConnectionClose="on"
    )

    input(
        type="imtcp"
        port="514"
        ruleset="remote"
    )
    ```

    > **Note**  
    > Pour rappel, le ruleset `remote` a été créé lors de la configuration de promtail.

3. Valider la configuration rsyslog avec la commande `rsyslogd -N 1` :

    ```bash
    $ rsyslogd -N 1
    rsyslogd: version 8.2102.0, config validation run (level 1), master config /etc/rsyslog.conf
    rsyslogd: End of config validation run. Bye.
    ```

4. Redémarrer le service rsyslog :

    ```bash
    systemctl restart rsyslog
    ```

Le service est maintenant prêt à recevoir les logs via TCP.

## Ajouter des clients

Pour ajouter des clients rsyslog (sur des machines avec rsyslog d'installés), il faut faire la même chose que pour le serveur syslog.

Sur le serveur à configurer, éditer le fichier `/etc/rsyslog.d/remote-logging-tcp.conf` (nouveau fichier) :

```bash
nano /etc/rsyslog.d/remote-logging-tcp.conf
```

Rajouter ensuite les éléments de configuration suivants :

```conf
## Tous les messages concernant l'authentification
auth,authpriv.*                 @@(z4)192.168.0.1:514

## Tous les messages en warn et plus sont envoyes dans syslog
*.warn;auth,authpriv.none       @@(z4)192.168.0.1:514

## Tous les messages kern sont envoyes dans kern
kern.*                          @@(z4)192.168.0.1:514

## Tous les messages user sont envoyes dans user
user.*                          @@(z4)192.168.0.1:514

## Tous les autres messages qui ne matchent pas plus haut
## Specificite : on ne les envoie pas au serveur syslog
*.=info;*.=notice;\
        auth,authpriv.none;\
        cron,daemon.none;\
        mail.none               -/var/log/messages

## Ligne speciale : tous les messages emerg sont envoyes aux terminaux ouverts
*.emerg                         @@(z4)192.168.0.1:514
```
