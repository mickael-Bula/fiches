## Connexions aux BDD postgres

Une bonne pratique est de toujours créé un user dédié à une BDD : si la sécurité de celui-ci est compromise, les autres BDD ne le seront pas.

Pour ce faire, il est possible de créer un role(user) avec les privilèges nécessaires à l'aide des requêtes suivantes :

```sql
create database database_name;
GRANT ALL PRIVILEGES ON DATABASE nom_de_la_base TO nom_du_role;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO nom_du_role;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO nom_du_role;
```

Pour lui ajouter un mot de passe (ce qui ne se fait par défaut dans phpstorm) :

```sql
ALTER USER nom_du_role WITH PASSWORD 'votre_mot_de_passe';
```
