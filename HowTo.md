# HelloDojo Marketing

Ce fichier permet de documenter (et logguer) les étapes que vous avez suivies
pour réaliser les demandes du [README.md](README.md).

## Mise en place
Voici les étapes que j'ai suivies pour installer XXX et importer les tables.
e.g. `CREATE SCHEMA `kata-sql` DEFAULT CHARACTER SET utf8mb4 ;`
J'ai choisi d'utiliser XXX comme client de base de donnée.
J'ai généré le schéma avec XXX:

![Mon MLD](schema.jpg "Mon MLD généré avec XXX")


## Informations à récolter

### Générales
1. La table `people` contient `NUMBER` personnes, ma requête est:  
   `SELECT COUNT(*) FROM people`
1. La table `people` contient `NUMBER` doublons, ma requête est:  
   `SELECT COUNT(*) AS nbr_doublon, firstname, lastname FROM people GROUP BY firstname, lastname HAVING COUNT(*) > 1`
1. Cette requête permet de trouver les infos de la personne dont le nom de
   famille est "Warren" ?
   `SELECT firstname, lastname, email, birthdate, idnumber FROM people WHERE people.lastname = "Warren" GROUP BY firstname, lastname`
1. La table `people` est triée par nom de famille, ma requête est:  
   `SELECT   firstname, lastname, email, birthdate, idnumber
    FROM     people
    ORDER BY lastname ASC`
1. Les 5 premières entrées de la table `people` sont:  
   `SELECT * FROM people LIMIT 5`
1. Je trouve toutes les personnes dont le nom ou le prénom contient `ojo`, ma  
  requête est:  
  `SELECT * FROM people WHERE firstname OR lastname LIKE "%ojo%"`
1. Les 5 personnes plus agées sont obtenus avec cette requête:  
  `SELECT * FROM people ORDER BY birthdate ASC LIMIT 5`
1. Les 5 personnes plus jeunes sont obtenus avec cette requête:  
  `SELECT * FROM people ORDER BY birthdate DESC LIMIT 5`
1. La requête suivante permet de trouver l'age en année de chaque personne:  
  `SELECT *, TIMESTAMPDIFF(YEAR, birthdate, NOW()) AS age FROM people`
1. La moyenne d'age est `26.1732`, ma requête est:  
  `SELECT AVG(TIMESTAMPDIFF(YEAR, birthdate, NOW())) FROM people`
1. Le plus long prénom est `Clementine`, ma requête est:  
  `SELECT firstname FROM people WHERE LENGTH(firstname) = (SELECT max(LENGTH(firstname)) FROM people)`
1. Le plus long nom de famille est `Christensen`, ma requête est:  
  `SELECT lastname FROM people WHERE LENGTH(lastname) = (SELECT max(LENGTH(lastname)) FROM people)`
1. Les plus longues paires nom + prénom sont `Cheyenne Pennington, Wallace Christensen, Benedict Daugherty`, ma requête est:  
  `SELECT firstname, lastname FROM people WHERE (LENGTH(lastname) + LENGTH(firstname)) = (SELECT MAX(LENGTH(firstname) + LENGTH(lastname)) FROM people)`
  Double vérification :
  `SELECT *, (LENGTH(lastname) + LENGTH(firstname)) AS total FROM people ORDER BY total DESC`

### Invitations
    `SELECT *, TIMESTAMPDIFF(YEAR, birthdate, NOW()) AS age FROM people
    WHERE TIMESTAMPDIFF(YEAR, birthdate, NOW())>18 AND TIMESTAMPDIFF(YEAR, birthdate, NOW())<60 AND (email REGEXP '[a-zA-Z0-9_\\-\\.\\+]+@([a-zA-Z0-9_\\-]+\\.)+([a-zA-Z0-9_\\-]+)')`
1. Pour lister tous le membres de plus de 18 ans:  
  a. et de moins de 60 ans:  
  a. qui ont une addresse email valide:  
  a. Avec une colonne `age` dans le résultat de la requête:  
  `SELECT *, YEAR(NOW())-YEAR(birthdate) AS age FROM people
    HAVING age>18 AND age<60 AND (email REGEXP '[a-zA-Z0-9_\\-\\.\\+]+@([a-zA-Z0-9_\\-]+\\.)+([a-zA-Z0-9_\\-]+)')`
1. Pour générer une liste contenant `Prénom Nom <email@provider.com>;`:  
    `SELECT CONCAT(firstname, lastname) AS blend
    FROM people
    WHERE YEAR(NOW())-YEAR(people.birthdate)>18 AND YEAR(NOW())-YEAR(people.birthdate)<60 AND (email REGEXP '[a-zA-Z0-9_\\-\\.\\+]+@([a-zA-Z0-9_\\-]+\\.)+([a-zA-Z0-9_\\-]+)')`
1. Avec cette requête:  
     `SELECT COUNT(*)
    FROM people
    WHERE YEAR(NOW())-YEAR(people.birthdate)>18 AND YEAR(NOW())-YEAR(people.birthdate)<60 AND (email REGEXP '[a-zA-Z0-9_\\-\\.\\+]+@([a-zA-Z0-9_\\-]+\\.)+(ch)')`
   je peux estimer que `NUMBER` personnes habitent en Suisse.

### Countries
1. La requête qui permet d'obtenir la liste d'options
   sous la forme: `<option value="XXX">XXX</option>` est:  
   `SELECT CONCAT("<option value=", name_fr, ">", name_fr, "</option>") FROM countries`
1. Pour avoir la liste d'options en plusieurs langues, je procède de la manière suivante:
`SELECT CONCAT("<option value=", name_fr, ">", name_fr, "</option>") FROM countries`
`---------------------------`

### Jointure
1. Avec cette requête:  
    `SELECT COUNT(*) FROM people
    JOIN countries_people
    ON countries_people.idperson=people.id
    JOIN countries
    ON countries.id = countries_people.idcountry
    WHERE name_fr="Suisse"`
   je sais que `NUMBER` personnes habitent en Suisse.
1. Avec cette requête:  
    `SELECT COUNT(*) FROM people
    JOIN countries_people
    ON countries_people.idperson=people.id
    JOIN countries
    ON countries.id = countries_people.idcountry
    WHERE name_fr!="Suisse"`
   je sais que `NUMBER` personnes n'habitent pas en Suisse.
1. Avec cette requête:  
    `SELECT CONCAT(firstname, " ", lastname) FROM people
    JOIN countries_people
    ON countries_people.idperson=people.id
    JOIN countries
    ON countries.id = countries_people.idcountry
    WHERE name_fr REGEXP '(France|Allemagne|Italie|Autriche|Lischenchtein)'`
   je liste (nom & prénom) des membres habitants de France, Allemagne, Italie, Autriche
   et Lischenchtein.
1. Cette requête:  
    `SELECT COUNT(*), countries.name_fr FROM people
    JOIN countries_people
    ON countries_people.idperson=people.id
    JOIN countries
    ON countries.id = countries_people.idcountry
    GROUP  BY countries.name_fr`
   permet de compter combien il y a de personnes par pays.
1. Cette requête:  
     `SELECT name_fr FROM countries WHERE countries.id NOT IN ( SELECT countries_people.idcountry FROM countries JOIN countries_people ON countries_people.idcountry = countries.id )`
   liste les pays qui ne possèdent pas de personnes.
1. En exécutant cette requête:  
     `SELECT firstname FROM people WHERE (SELECT COUNT(*) FROM countries_people WHERE countries_people.idperson = people.id) > 1`
   je sais que `DAI` et `MINERVA` sont lié(e)s à plusieurs pays.
1. En exécutant cette requête:  
    `SELECT firstname FROM people
    WHERE people.id = (SELECT countries_people.idperson FROM countries_people
    WHERE countries_people.idcountry NOT IN (SELECT countries.id FROM countries))`
   je sais que `NAME` est lié à un pays qui n'existe pas dans la base.
1. De la manière suivante:
    `SELECT (COUNT(*) / (SELECT COUNT(*) FROM people) * 100) AS poucentage, countries.name_fr FROM people
    JOIN countries_people
    ON countries_people.idperson=people.id
    JOIN countries
    ON countries.id = countries_people.idcountry
    GROUP  BY countries.name_fr`
   nous pouvons afficher le pourcentages de personnes par pays.


### Procédures

1. Cette requête permet d'extraire `tld` de l'adresse email et de le lier à la
   table `countries`:  
    `SELECT people.firstname, people.email, countries.tld FROM people
    LEFT JOIN countries_people
    ON countries_people.idperson=people.id
    LEFT JOIN countries
    ON countries.id = countries_people.idcountry
    GROUP  BY countries.tld`
1. Pour ajouter une chaine si la jointure ne retourne rien, j'ai procédé de la
   manière suivante:  
   `SELECT people.firstname, people.email, countries.tld FROM people
   LEFT JOIN countries_people
   ON countries_people.idperson=people.id
   LEFT JOIN countries
   ON countries.id = countries_people.idcountry
   GROUP  BY countries.tld`
1. Avec `STRING`, nous pouvons partager le mécanisme qui extrait le `tld`.

### Vue
1. J'ai créé une vue bien pratique contenant toutes les informations utiles à
   un humain. Ma requête est:  
   `SELECT people.firstname, people.lastname, people.birthdate, people.email, people.idnumber, countries.iso2, countries.iso3, countries.name_en, countries.name_fr, countries.tld
FROM people
   LEFT JOIN countries_people
   ON countries_people.idperson=people.id
   LEFT JOIN countries
   ON countries.id = countries_people.idcountry`

### Finances
1. J'ai créé une table pour les finances. Ma requête est:  
   `CREATE TABLE finances ( id INT, avoir INT )`
1. J'ai modifié la vue en y ajoutant les finances. Ma requête est:  
   `UPDATE XXX`
