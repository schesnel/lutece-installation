# Contenue de ce repo

*  `pom.xml` : le fichier qui va vous permettre d'assembler une application Lutece
*  `site-forms-demo-1.0.0-SNAPSHOT.zip` : l'application déjà assemblée et prête à être deployée sur Tomcat

# L'assemblage d'une application

* L'assemblage de l'application vous permet de choisir quelle librairies et plugins/modules vous voulez inclure dans l'application.
* Pour assembler l'application, vous n'avez besoin que du `pom.xml`. Ce fichier peut varier selon ce que vous voulez avoir comme librairies et plugins.
* Le `pom.xml` dans ce repo contient une vintaine de plugin les plus utilisés
* pour assembler l'application, `maven` devrait être installée préalablement

Pour assembler une application il suffit de rouler cette command dans le même dossier que `pom.xml`:

`mvn lutece:site-assembly`

Le resultat c'est un dossier `target` qui contient l'application sous 2 formes :
* site-forms-demo-1.0.0-20210425.105408.war : version compressée de l'application (ne sera pas utilisée)
* site-forms-demo-1.0.0-SNAPSHOT : version non compréssée. C'est cette version qu'on va utiliser lors du déploiment puisque ça nous permet de changer quelques parametres de base de données

# Ajouter un plugin

Disons on veut ajouter un plugin pour le helpdesk, voici les étapes :

1. Ouvrez `pom.xml` et aller à la fin de la section des plugins (ligne 226)
2. Coller ce bout de code pour la dépendance  (j'ai pris la version 3.1.4 qui est la denière. [La list complete des versions de ce plugin](https://dev.lutece.paris.fr/nexus/content/groups/maven_repository/fr/paris/lutece/plugins/plugin-helpdesk/)) 
<pre>
  &lt;dependency&gt;
       &lt;groupId&gt;fr.paris.lutece.plugins&lt;/groupId&gt;
       &lt;artifactId&gt;plugin-helpdesk&lt;/artifactId&gt;
       &lt;version&gt;[3.1.4]&lt;/version&gt;
       &lt;type&gt;lutece-plugin&lt;/type&gt;
   &lt;/dependency&gt;
</pre>

3. Assembler l'application avec `mvn lutece:site-assembly`
4. L'application sera générée dans le dossier `target`

# Rouler l'application

##### Quelques considérations
- Rouler l'application consiste à préparer la base de donnée et à deployer l'application sur un serveur Tomcat.
- On va partir du fichier zip qui est dans ce repo. Je fais le déployement sur un Ubuntu 18.04.4 mais ça serait pareil pour pas mal d'autres version Linux

##### Les applications suivantes devrait être installées préalablement
- Tomcat : le serveur sur lequel l'application va rouler
- Ant : permet de rouler un script qui va ajouter les tables nécessaires dans la base de données
- MySQL : la base de donnée

Les étapes du déploiment:

1. `cd {dossier tomcat}/bin && ./shutdown.sh` (Assurez-vous que Tomcat est arrêté)
1. `unzip site-forms-demo-1.0.0-SNAPSHOT.zip -d site-forms-demo-1.0.0-SNAPSHOT` (Dézipper le fichier qui est dans ce repo et bien utiliser le dossier qui est dans target assemblé)
2. `sed -i 's/root/admin/' site-forms-demo-1.0.0-SNAPSHOT/WEB-INF/conf/db.properties` (Changer l'utilisateurde root à admin)
3. `cp -r site-forms-demo-1.0.0-SNAPSHOT {dossier tomcat}/webapp/lutece` (copier l'application vers le dossier webapp de tomcat)
4. `mysql -uroot -e "CREATE USER 'admin'@'%' IDENTIFIED BY 'motdepasse'; GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%';CREATE DATABASE lutece;"` 
(créer l'utilisateur admin et la bd lutece. Ici on prend pour acquis que votre utilisateur `root` de MySQL n'a pas de mot de passe)
5. `cd site-forms-demo-1.0.0-SNAPSHOT/WEB-INF/sql && ant` (rouler le script ant qui va initialiser les tables)
6. `cd {dossier tomcat}/bin && ./startup.sh` (Rouler Tomcat. Ça peut prendre quelques minutes)
7. Garder un oeil sur les logs dns `{dossier tomcat}/logs/catalina.out` s'il y a des erreurs
8. Le site est accessible sous `<votre serveur>:8080/lutece` et la section admin est accessible sur : `<votre serveur>:8181/lutece/jsp/admin/AdminLogin.jsp`.
  L'accès par défaut : admin/adminadmin
  
  















## 