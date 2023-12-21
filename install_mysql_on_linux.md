# INSTALLATION DE MYSQL SUR LINUX (UBUNTU)

Par défaut, le user root de mysql n'a pas de mot de passe et se connecte au CLI mysql avec la commande :
```bash
$ sudo mysql
```

Cependant, lors de la configuration du mode 'safe' avec 'mysql_secure_installation', il est préférable d'ajouter un mot de passe à root avant de lancer l'utilitaire

```bash
$sudo mysql
```

```sql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '<mon_mot_de_passe>';
mysql> exit
```

```bash
$ sudo mysql_secure_installation
# on peut alors répondre Yes à toutes les questions pour renforcer la sécurité
```

Une fois fait, on peut rétablir l'authentification **sudo** :

```bash
$ mysql -u root -p	# taper le ot de passe précédemment configuré
```

```sql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
```

On peut alors, à nouveau, se connecter avec **sudo** :

```bash
$ sudo mysql
```

```sql
mysql> ...
```
=====================

Par souci de sécurité, il est préférable de créer un utilisateur spécifique pour chaque BDD :

```bash
$ sudo mysql
```

```sql
mysql> CREATE USER 'mika'@'localhost' IDENTIFIED BY '<mon_mot_de_passe>';
mysql> GRANT ALL PRIVILEGES ON webtrader.* TO 'mika'@'localhost' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
mysql> exit
```

Dès lors, pour se connecter avec cet utilisateur, on utilisera la commande :

```bash
$ mysql -u mika -p	# saisir le mot de passe
```

```sql
mysql> ...
```

=====================

Pour tester l'état de la connexion :

```bash
$ systemctl status mysql.service
```

Si la connexion n'est pas active :

```bash
$ sudo systemctl start mysql
```
