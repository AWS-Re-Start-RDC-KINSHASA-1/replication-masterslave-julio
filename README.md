# replication-masterslave-julio
Réplication maître-esclave avec MariaDB
Ce didacticiel vous guide tout au long du processus de configuration de la réplication maître-esclave avec MariaDB sur deux instances Amazon EC2 exécutant Amazon Linux 3.

Conditions préalables
Deux instances EC2 avec Amazon Linux 3 avec mariadb sur chaque instance.

Configuration de la réplication maître-esclave MariaDB

Étape 1 : Installer MariaDB sur l'instance principale
sudo yum install mariadb-server
sudo systemctl start mariadb
sudo systemctl enable mariadb

Étape 2 : Installation sécurisée de MariaDB
sudo mysql_secure_installation

Étape 3 : Configurer l'instance Master
Modifier le fichier de configuration MariaDB

sudo nano /etc/my.cnf.d/server.cnf
Ajoutez les lignes suivantes :

server-id = 1
log_bin = /var/log/mariadb/mariadb-bin
Redémarrer MariaDB

sudo systemctl restart mariadb
Créer un utilisateur de réplication

CREATE USER 'repl'@'%' IDENTIFIED BY 'your_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
Obtenir le fichier journal principal et la position

SHOW MASTER STATUS;

Étape 4 : Installer MariaDB sur une instance esclave
Répétez l'étape 1 sur la deuxième instance EC2.

Étape 5 : Configurer l'instance esclave
Modifier le fichier de configuration MariaDB sur linstance esclave

sudo nano /etc/my.cnf.d/server.cnf

Ajoutez les lignes suivantes :

server-id = 2
Redémarrer MariaDB

sudo systemctl restart mariadb
Configurer la réplication esclave

Sur l'instance Esclave :

CHANGE MASTER TO
  MASTER_HOST='master_instance_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='your_password',
  MASTER_LOG_FILE='master_log_file_from_master',
  MASTER_LOG_POS=master_log_position_from_master;
  
Étape 6 : Démarrer la réplication esclave Sur l'instance Esclave :

START SLAVE;
Étape 7 : Vérifier la réplication Sur l'instance Esclave :

SHOW SLAVE STATUS\G
Assurez-vous que Slave_IO_Running et Slave_SQL_Running sont tous deux Oui.

Test de réplication
Créez une base de données sur le Master :

CREATE DATABASE testdb;
Vérifiez que la base de données est répliquée sur l'esclave :

SHOW DATABASES;
Assurez-vous que testdb est répertorié sur l'esclave.
