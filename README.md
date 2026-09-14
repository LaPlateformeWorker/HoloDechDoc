## Holodeck

# Introduction :
Ce document présente l’infrastructure « Holodeck », conçue pour fournir aux ingénieurs de Starfleet un environnement de développement web complet, conformément au cahier des charges de La Plateforme.
Le projet comprend deux VM Debian : un serveur hébergeant les services (DHCP, DNS, Web, BDD, LDAP, FTP) et un client permettant de les tester via un navigateur.
Le réseau est cloisonné entre un LAN interne et un WAN, avec un pare-feu limitant les accès aux ports nécessaires.
Le document présente également l’exportation des VM pour le rendu ainsi que leur installation et utilisation sur un autre poste

# Architecture du projet : 
L’architecture reproduit une petite passerelle d’entreprise ou le serveur est connecté au WAN (Internet) et au LAN interne, qu’il fournit en adresses IP et résolution DNS, tout en hhébergeant les différents services. 
Le tableau suivant récapitule les services exposés et le sous-domaine associé. Tous servis en HTTPS derrière un unique port 443, à l'exception du FTP et du LDAP qui conservent leurs ports dédiés :

Sous domaine
Role
Port
Service
www8.starfleet.lan
Site web en PHP 8.x
443/tcp
Nginx + PHP8-FPM
www7.starfleet.lan
Site web en PHP 7.x
443/tcp
Nginx + PHP7-FPM


php.starfleet.lan
Administration de la base de données
443/tcp
phpMyAdmin 
admin.starfleet.lan
Administration système de la VM
443/tcp
Cockpit
vscore.starfleet.lan 
Éditeur de code distant (bonus)
443/tcp
code-server


Dépôt de fichiers web, chiffré
21 / 990
vsftpd (FTPS)


Annuaire des utilisateurs
389/tcp
OpenLDAP


Contraintes de sécurité respectées
Aucun compte sudo : administration exclusivement en root.
Pare-feu nftables : politique par défaut « deny », seuls les ports nécessaires sont autorisés.
Nginx, PHP et MariaDB : installés depuis leurs dépôts officiels.
Certificat SSL interne : sécurise le serveur Web et le serveur FTP.

# 3. Démarrage et connexion
1.   Démarrer la VM starfleet-srv en premier et attendre la fin du démarrage des services (une à deux minutes).
2.   Démarrer ensuite la VM starfleet-client.
3.   Se connecter avec les identifiants fournis dans le tableau récapitulatif (section 5).
4.   Sur la VM cliente, importer le certificat racine starfleetCA.crt dans le magasin de certificats de confiance (ou directement dans Firefox : Paramètres → Vie privée et sécurité → Certificats → Afficher les certificats → Importer), afin que les sites en HTTPS s'affichent sans avertissement.
# 4. Accès aux services
Une fois les deux VM démarrées et le certificat importé, ouvrir un navigateur sur la VM cliente et accéder aux adresses suivantes (résolues automatiquement par le DNS interne, aucune modification du fichier hosts n'est nécessaire) :
•     https://www8.starfleet.lan — site de démonstration en PHP 8
•     https://www8.starfleet.lan/login.php — démonstration d'authentification via l'annuaire LDAP
•     https://www7.starfleet.lan — site de démonstration en PHP 7
•     https://php.starfleet.lan — phpMyAdmin
•     https://admin.starfleet.lan — administration système (Cockpit, connexion en root)
•     https://vscore.starfleet.lan — éditeur de code en ligne (bonus)
Le dépôt de fichiers vers le site web se fait quant à lui en FTPS (port 21, données chiffrées TLS), par exemple avec FileZilla, en se connectant à l'adresse IP du serveur (192.168.56.1) avec un compte FTP dédié.
# 5. Utilisation courante
•     Arrêt propre : toujours éteindre les VM via la commande shutdown -h now (serveur) ou le menu système (client) plutôt que de les suspendre ou de forcer l'arrêt, pour éviter toute corruption des bases de données.
•     Sauvegardes : le script /root/backup.sh s'exécute automatiquement chaque nuit à 2h (tâche cron) et archive la configuration système, un export complet de MariaDB et de l'annuaire LDAP dans /root/backups, avec purge automatique des archives de plus de 7 jours.
•     Mises à jour : les dépôts tiers (Nginx, PHP/Sury, MariaDB) sont déjà configurés — un simple apt update && apt upgrade suffit à maintenir les versions à jour.
•     Dépannage rapide : nft list ruleset pour vérifier le pare-feu, systemctl status <service> pour l'état d'un service, journalctl -u <service> -e pour ses derniers journaux.







