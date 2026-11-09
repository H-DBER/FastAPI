# Journal de l'API Olist — FastAPI

**Nom :** Debrabandère Hélène
**Dates :** J1 04/11/2026 · J2 …
**Fichier rendu :** api_olist_DEBRABANDERE.zip

> Compétences : WCS C1.6 · C2.4 · C2.1 · C2.5 (niveau 2 · Adapter) — Simplon C6 · C7 (partiel)
> Remplis les parties **dans l'ordre**. Remplace chaque « … ». Une phrase suffit sauf mention contraire.

**Chiffres de référence (export de lundi)**
- Lignes : 110 197 · Commandes livrées : 96 478
- CA total : 15 424 463,65 R$ · Panier moyen : 159,88 R$
- Novembre 2017 : 1 154 559,53 R$ et 7 289 commandes
- SP : 5 771 713,93 R$, 40 501 commandes, 37,42 % du CA

---

# J1 — L'API sur le fichier CSV

<a id="j-m1"></a>
## M1 — Lancer l'API et la tester

- Version de Python : 3.14.7
- Commande de lancement que j'ai utilisée : curl -X 'GET' \ 'http://127.0.0.1:8000/sante' \ -H 'accept: */*'
- Code et `nb_lignes` renvoyés par `/sante` : 200 : Successful Response,  "nb_lignes": 110197 
- Différence entre `/docs` et `/sante` ouverts dans le navigateur : la mise en page / format 
- Problème rencontré et solution (ou « aucun ») : le terminal dit "INFO:     127.0.0.1:64844 - "GET /favicon.ico HTTP/1.1" 404 Not Found" : c'est une petite icone affichée sur navigateur, non trouvée. 404 probablement affiché lors du chargement de /sante sans mise en forme. Il y en a une pour le portail principal mais pas le détail /sante.

<a id="j-m2"></a>
## M2 — Lire une route complète

### Le trajet d'une requête `GET /indicateurs`, en 4 étapes

1. Le navigateur envoie les informations de `donnees_csv.py` à l'adresse `http://127.0.0.1:8000/docs`
2. FastAPI exécute la fonction GET (@app.get) du fichier `main.py`, qui appelle les données de `donnees_csv.py`
3. Cette fonction lit `donnees_csv.py` et renvoie les données `sante` et `indicateurs` appelées par `main.py`
4. Le modèle `main.py` vérifie le type des données reçues puis FastAPI renvoie une réponse du server contenant les informations demandées, avec le code 200 : Succesful Response.

### L'expérience du contrat

- Quand j'ai mis `panier_moyen` en commentaire dans le modèle, la réponse : WARNING:  StatReload detected changes in 'main.py'. Reloading... : la console a détecté un changement dans le `main.py`, ce qui a provoqué un redémarage du server, pour ommetre l'option `panier_moyen` du visuel sur navigateur.
- Conclusion : `main.py` décide de quelles info de `donnees_csv.py` sortent sur le server.

### Pourquoi deux fichiers (`main.py` et `donnees_csv.py`) ?

- `donnees_csv.py` représente notre source de données, et `main.py` représente notre interface.

### Comparaison avec lundi

- `nb_commandes` : 96478 (lundi : 96 478)
- `ca_total` : 15424463.65 (lundi : 15 424 463,65)

<a id="j-m3"></a>
## M3 — Ventes mensuelles

### Tests

- Sans paramètre : 23 mois, du 2016-09 au 2018-08
- `debut=2017-01` et `fin=2017-12` : 12 mois ; novembre 2017 = 1154559.53 R$ et 7289 commandes
- `debut=2018-02` et `fin=2017-01` : code 400, message : "Error: Bad Request"
- `debut=2017-13` : code 422, message : "Error: Unprocessable Content"

### Questions

1. Différence entre un paramètre de chemin et un paramètre de requête, avec un exemple de cette API : les deux sont compris dans l'adresse exposée dans la rubrique curl, par exemple :
   http://127.0.0.1:8000/ventes/mensuelles?debut=2017-01&fin=2017-12
le paramètre de chemin est : ventes/mensuelles : Où je cherche, 
le paramètre de requête est : ?debut=2017-01&fin=2017-12 : ce que je cherche
   
2. Qui renvoie le code 400 (toi ou FastAPI) ? Et le code 422 ? Pourquoi cette différence ? 
Ce n'est pas clair pour moi, qui renvoie les messages d'erreur. Les deux sont noté "server : uvicorn", donc moi. Néanmoins, j'imagine que:
   400 est renvoyé par moi : j'ai prévu un message d'erreur pour ce cas de figure dans `main.py`
   422 est renvoyé par l'api : ???
   
<a id="j-m4"></a>
## M4 — Top catégories

### Tests

- Sans paramètre : 10 catégories ; la première est "health_beauty" avec 9.16 %
- Somme des 10 parts : 62.39 %
- `n=3` : 25.3%
- `n=0` : code Ne s'affiche pas · `n=51` : 422 
### Question

- Que reçoit Rafael s'il n'envoie pas `n` ? Où est-ce décidé dans le code ? 
Sans préciser 'n', le code envoie les 10 premières catégories.

<a id="j-m5"></a>
## M5 — Chiffres d'un État

### Mon modèle `Etat` (copie le code)

```python
@app.get("/etats/{code_etat}", response_model=..., tags=["Indicateurs"])
def etat(code_etat: str = Path(pattern=r"^[A-Za-z]{2}$", description="Code de l'État sur 2 lettres, ex. SP")):
    """Chiffres d'un État brésilien."""
    code = code_etat.upper()          # « sp » et « SP » donnent le même résultat
    resultat = donnees.etat(...)
    if resultat is None:
        raise HTTPException(status_code=..., detail=f"Aucune vente pour l'État « {code} ».")
    return resultat
```
# mon code : 

class Etat(BaseModel):
    etat: str
    ca: float
    nb_commandes: int
    part_ca: float


@app.get("/etats/{code_etat}", response_model=list[Etat], tags=["Indicateurs"])
def etat(code_etat: str = Path(pattern=r"^[A-Za-z]{2}$", description="Code de l'État sur 2 lettres, ex. SP")):
    """Chiffres d'un État brésilien."""
    code = code_etat.upper()          # « sp » et « SP » donnent le même résultat
    resultat = donnees.etat(code)
    if resultat is None:
        raise HTTPException(status_code=404, detail=f"Aucune vente pour l'État « {code} ».")
    return [resultat]

### Tests

- `/etats/SP` : CA 5771713.93 , commandes 40501 , part 37.42 %
- `/etats/sp` : identique (oui / non) : oui
- `/etats/XX` : code 404 , message Error: Not Found
- `/etats/SPX` : code 422

### Question

- Pourquoi un État sans vente renvoie 404 et pas une réponse vide avec le code 200 ?
  Parce qu'on cherche un bloc de réponse, avec un format particulier. Si ce format exact n'est pas trouvé, alors il est absent, et s'il est absent, il ne peut pas être trouvé.

<a id="j-m6"></a>
## M6 — Recette complète dans /docs

Pour chaque test : le code obtenu, puis une valeur lue dans la réponse.

1. `GET /sante` (attendu 200) : 200, "nb_lignes": 110197
2. `GET /indicateurs` (attendu 200) : 200, "ca_total": 15424463.65
3. `GET /ventes/mensuelles` sans paramètre (attendu 200, 23 mois) : 200, 23 mois, "mois": "2016-10", "ca": 46490.66, "nb_commandes": 265
4. `GET /ventes/mensuelles?debut=2018-01` (attendu 200) : 200, nombre de mois 8,  "mois": "2018-03", "ca": 1120726.7, "nb_commandes": 7003
5. `GET /ventes/mensuelles?debut=2018-02&fin=2017-01` (attendu 400) : 400 : "detail": "« debut » doit être antérieur ou égal à « fin »."
6. `GET /ventes/mensuelles?fin=nov` (attendu 422) : 422, "msg": "String should match pattern '^\\d{4}-(0[1-9]|1[0-2])$'",
7. `GET /categories/top` (attendu 200, 10 catégories) : 200, "categorie": "computers_accessories", "ca": 1032723.77, "part_ca": 6.7
8. `GET /categories/top?n=50` (attendu 200) : 200, dernière catégorie de la liste : "categorie": "books_technical", "ca": 22901.07,
 "part_ca": 0.15
9. `GET /categories/top?n=0` (attendu 422) : 422 
10. `GET /etats/RJ` (attendu 200) : 200, part du CA part_ca": 13.33
11. `GET /etats/XX` (attendu 404) : 404, "detail": "Aucune vente pour l'État « XX »."
12. `GET /etats/SPX` (attendu 422) : 422

### Bilan du J1

1. Ce qui fonctionne : tout !
2. Ce qui me reste à comprendre ou à terminer : je ne saisi pas encore entièrement le code de chemin qu'il a fallu écrire en mission 5. Je n'ai pas appelé "code" initialement, je ne savais pas quelle valeur mettre à cet endroit.

---

# J2 — L'API sur PostgreSQL

<a id="j-m7"></a>
## M7 — Charger les ventes dans PostgreSQL

- Base utilisée : olist
- Résultat du contrôle final du script : 110197 lignes, 96478 commandes, 15424463.65 de CA
- Droit donné à `lecteur_olist` sur la table, et pourquoi seulement celui-là : en lecture seule (SELECT), pour assurer qu'il ne modifie pas les valeurs qu'on lui transmets.

<a id="j-m8"></a>
## M8 — Mettre les secrets dans `.env`

- Résultat de la commande de test : lecteur_olist
- Rôle de `.env` : stocker les secrets et figurer dans la liste des dossiers à ignorer de gitignore
- Rôle de `.env.exemple` : nous fournir un exemple de présentation de env.
- Rôle de la ligne `.env` dans `.gitignore` : signale qu'il faut ignorer les documents .env
- J'ai vérifié qu'aucun `.py` ne contient le mot de passe (oui / non) : oui

<a id="j-m9"></a>
## M9 — Réécrire les fonctions en SQL

### Résultat de `verifier_sources.py`

- Bilan affiché : 8 / 8
- Fonction qui m'a demandé le plus de corrections, et pourquoi : 
  top50 : `ORDER BY part_ca DESC` au lien de `ORDER BY ca DESC`

### Ma requête de `top_categories()` (copie-la)

```sql
        SELECT categorie,
               round(sum(montant), 2) AS ca,
               round(100 * sum(montant) / (SELECT sum(montant) FROM analytique.ventes), 2) AS part_ca
        FROM analytique.ventes
        GROUP BY categorie
        ORDER BY ca DESC
        LIMIT %(n)s
```

### Sécurité

- Pourquoi `%(code)s` plutôt qu'une f-string ? Donne l'exemple d'une valeur dangereuse. 
  Il s'agit d'une sécurité pour éviter d'injecter plus de code que le `%(code)s` voulu. En envoyant par exemple '1' = '1', la base de données va ignorer le filtre et tout retourner. Cela contourne la sécurité de la BDD. un usager malveillant pourrait aussi inclur d'autres choses dans la f-string et injecter du code pour supprimer toute la BDD par exemple. 
- Pourquoi faire calculer PostgreSQL plutôt que Python ? par principe d'économie. Postgre ne retourne que quelques lignes à python, au lieu de l'intégralité de la BDD. 

<a id="j-m10"></a>
## M10 — Basculer l'API sur PostgreSQL

- `/sante` après la bascule : source postgresql , lignes 110197
- Pourquoi il faut relancer l'API après avoir modifié `.env` : parce que les paramètres de connexion ont changé, il faut les mettre à jour
- Test de panne (`PGPORT=5999`) : code 503 , message `Error: Service Unavailable`
- Ce qu'aurait vu Rafael sans le gestionnaire d'erreur `SourceIndisponible` : un message d'erreur sans aucune information, probablement
- Ce que Rafael a dû changer dans son code pour la bascule : il faut qu'il modifie le PGPORT

<a id="j-m11"></a>
## M11 — Écrire le client de Rafael

- Bilan affiché par `client.py` : 10 / 10
- Nombre de lignes de `ca_mensuel_2017.csv`, en-tête compris : 13
- Différence entre `.status_code` et `.json()` :  `.status_code` donne le code brut, `.json()` : transforme la réponse en dictionnaire ou en liste python
- Utilité du paramètre `timeout` : donne une borne de fin temporelle à la requête, pour éviter que ça mouline dans le vide indéfiniment.

<a id="j-m12"></a>
## M12 — README et rendu

- Le test de remise à zéro a fonctionné en suivant seulement le README (oui / non) : oui
- Ce que j'ai dû corriger dans le README après ce test : RAS
- Contenu de mon zip, fichier par fichier : .env.exemple, .gitignore, client.py, donnees_csv.py, donnees_pg.py, erreurs.py, main.py, requirements.txt, verifier_sources.py, ca_mensuel_2017.csv

### Mes choix techniques (3 à 5 lignes)

- Source de données retenue et pourquoi : Postgre, ne laisse pas l'utilisateur modifier les données, requêtes + rapides et économes
- Compte de connexion et pourquoi : SELECT (lecture simple)
- Gestion des erreurs : En cas d'échec, l'API renvoie un code d'erreur documenté dans ce README
- Ce que j'ajouterais pour mettre l'API en production : sécuriser l'API 

### Message à Mariana (3 lignes, sans jargon)

- Ce que Rafael peut faire maintenant : lancer l'api sur son navigateur et consulter les chiffres dont il a besoin
- D'où viennent les chiffres : du server de l'entreprise
- Le chiffre qui prouve que c'est juste : Quel chiffre? à quoi renvoie "c'est" ? / les chiffres viennent de l'entreprise : confirmé par service/sante :  "source": "postgresql"
      - le chiffre qui prouvent que la base de donnée est juste : une comparaison entre ca_mensuel_2017.csv et une requête sur l'API par exemple : le CSV dit pour janvier 2017 : 2017-01,127482.37,750 ; l'API renvoie exactemement les mêmes valeurs : "mois": "2017-01", "ca": 127482.37, "nb_commandes": 750
