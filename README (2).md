# TP2 REST : Service web « Calculatrice » avec JAX-RS et Tomcat

**Compte rendu de : Eya Arifet**
**Matière :** Architecture MicroServices
**Établissement :** Faculté des Sciences de Sfax, Département d'informatique et des Communications
**Année universitaire :** 2026-2027

> Ce fichier contient uniquement les réponses aux questions et les tests demandés, chacun avec sa capture d'écran.
> Les URI sont toutes sous `http://localhost:8080/RestWebApp/api`.

---

## Partie A : Questions de conception

**1. Pourquoi les calculs directs utilisent-ils le verbe GET et non POST ?**
Parce qu'on veut seulement lire un résultat. Rien n'est créé ni modifié sur le serveur. GET sert à consulter, alors que POST sert à créer une ressource.

**2. Dans l'URI `/calcul/somme?a=2&b=3`, comment s'appelle la partie après le point d'interrogation ?**
C'est la chaîne de requête (*query string*). Elle contient les paramètres de requête `a=2` et `b=3`.

**3. Quelle différence y a-t-il entre `/operations` et `/operations/3` ?**
`/operations` désigne toute la collection des opérations. `/operations/3` désigne une seule ressource : l'opération dont l'identifiant est 3.

**4. Quel code de statut HTTP le serveur doit-il renvoyer pour `/operations/99` si elle n'existe pas ?**
Le code **404 (Not Found)**.

---

## Partie C : Premier service REST (la somme)

### Test C.7 : `GET /calcul/somme?a=2&b=3`

**Avec le navigateur** : la page affiche `5`.

![Test navigateur somme](images/test%20navig%205.png)

**Avec curl** : `curl -i "http://localhost:8080/RestWebApp/api/calcul/somme?a=2&b=3"`
Statut `HTTP/1.1 200`, en-tête `Content-Type: text/plain`, corps `5`.

![Test curl somme](images/test%20curl%205.png)

**Avec Postman** : GET, statut `200 OK`, corps `5`.

![Test Postman somme](images/te'st%20postman%205.png)

### Question C.1 : Remplacer GET par POST dans Postman

![POST sur /calcul/somme](images/remplacement%20de%20get%20par%20post%20,%205.png)

J'obtiens le statut **405 (Method Not Allowed)**.
La ressource `/calcul/somme` existe bien, mais la méthode `somme` est annotée seulement avec `@GET`. Elle ne répond donc pas au verbe POST, et Tomcat refuse la requête.

---

## Partie D : Compléter la calculatrice

### Test D.1 : Soustraction et produit

**`GET /calcul/soustraction?a=10&b=4`** : résultat `6`.

Dans le navigateur :

![Soustraction navigateur](images/test%20navigateur%20soustraction%206.png)

Dans Postman (200 OK) :

![Soustraction Postman](images/test%20postman%20sous%206.png)

**`GET /calcul/produit?a=6&b=7`** : résultat `42`.

Dans le navigateur :

![Produit navigateur](images/test%20navigateur%20produit%2042.png)

Dans Postman (200 OK) :

![Produit Postman](images/test%20postman%20produit%2042.png)

### Test D.2 : Division et gestion d'erreur

**`GET /calcul/division?a=7&b=2`** : statut `200`, corps `3.5`.

![Division 7 par 2](images/test%20postman%20div%203.5.png)

**`GET /calcul/division?a=7&b=0`** : statut `400 Bad Request`, corps `Erreur : division par 0 impossible`.

![Division par zéro](images/test%20postman%20div%20impossible%20par%200.png)

### Observations D.3

**1. `GET /calcul/somme?a=2` (le paramètre `b` est absent)** : statut `200`, corps `2`.

![Paramètre b absent](images/test%20b%20abscent.png)

**2. `GET /calcul/somme?a=abc&b=3` (`a` n'est pas un entier)** : statut `404 Not Found`.

![Paramètre non entier](images/test%20parametre%20non%20entier.png)

**Question D.1. Quelle valeur JAX-RS donne-t-il à un paramètre entier absent ?**
Il lui donne la valeur par défaut du type `int`, c'est-à-dire **0**. C'est pour ça que `somme?a=2` renvoie 2 + 0 = 2.

**Question D.2. Quel statut obtient-on quand le paramètre n'est pas un entier ? La méthode `somme` a-t-elle été exécutée ?**
J'obtiens **404 (Not Found)**. La méthode `somme` n'a **pas** été exécutée : Jersey n'arrive pas à convertir `abc` en entier, donc il s'arrête avant d'appeler la méthode.

### Test D.4 : `@PathParam`, `GET /calcul/produit/6/7`

Statut `200 OK`, résultat `42`.

![Produit avec PathParam](images/test%20postman%2042%20de%20d.4.png)

**Question D.3. Différence entre `@QueryParam` et `@PathParam` ? Laquelle pour lire l'identifiant dans `/operations/{id}` ?**
`@QueryParam` lit une valeur dans la chaîne de requête, après le `?` (exemple : `?a=2`). `@PathParam` lit une valeur directement dans le chemin de l'URI, à l'endroit marqué par `{...}` (exemple : `/produit/6/7`). Pour lire l'identifiant dans `/operations/{id}`, on utilise **`@PathParam`**.

---

## Partie E : Échanger des données en JSON

### Test E.2 : `GET /calcul/somme/detail?a=2&b=3`

Commande : `curl -i "http://localhost:8080/RestWebApp/api/calcul/somme/detail?a=2&b=3"`
Résultat : statut `200`, en-tête `Content-Type: application/json`, corps `{"a":2,"b":3,"operation":"somme","resultat":5.0}`.

![curl somme detail](images/curl%20accept%20somme%20=5%20json.png)

**Question E.1. Qui a effectué la conversion en JSON ? D'où viennent les noms `a`, `b`, `operation` et `resultat` ?**
C'est **Jersey** qui a converti l'objet Java en JSON, grâce à la dépendance `jersey-media-json-binding`. Les noms viennent des **propriétés de la classe `Resultat`**, c'est-à-dire de ses accesseurs (`getA()` donne `a`, `getOperation()` donne `operation`, etc.).

### Test E.3 : L'en-tête `Accept`

**Requête avec `Accept: application/json`** : elle donne la même réponse que la capture E.2 ci-dessus (statut 200, JSON).

**Requête avec `Accept: application/xml`** :
`curl -i -H "Accept: application/xml" "http://localhost:8080/RestWebApp/api/calcul/somme/detail?a=2&b=3"`
Résultat : statut **406 (Not Acceptable)**.

![Accept XML, statut 406](images/curl%20accept%20somme%20=5%20xml.png)

**Question E.2. Pourquoi la seconde requête est-elle refusée ?**
Parce que la méthode ne sait produire que du `application/json` (annotation `@Produces(MediaType.APPLICATION_JSON)`). Le client demande du XML, et le serveur n'a aucun format en commun à lui proposer. Il répond donc 406.

---

## Partie F : La ressource « operations »

### Tests F.3 : Création et lecture

| N° | Requête | Corps envoyé | Statut obtenu |
|----|---------|--------------|---------------|
| 1 | `POST /operations` | `{"operateur":"somme","a":8,"b":2}` | 201 |
| 2 | `POST /operations` | `{"operateur":"division","a":7,"b":2}` | 201 |
| 3 | `GET /operations` | aucun | 200 |
| 4 | `GET /operations/2` | aucun | 200 |
| 5 | `GET /operations/99` | aucun | 404 |
| 6 | `POST /operations` | `{"operateur":"division","a":7,"b":0}` | 400 |
| 7 | `POST /operations` | `{"operateur":"racine","a":9,"b":0}` | 400 |

**Test 1 : `POST /operations` avec `{"operateur":"somme","a":8,"b":2}`**
Avec curl : statut `201`, en-tête `Location: http://localhost:8080/RestWebApp/api/operations/1`, corps `{"a":8,"b":2,"id":1,"operateur":"somme","resultat":10.0}`.

![Test 1 curl](images/curl%20json%20somme%2010.png)

Avec Postman : statut `201 Created`, même opération créée (ici avec l'identifiant 3, car j'avais déjà fait d'autres créations avant).

![Test 1 Postman](images/postman%20post%20201%20somme.png)

**Test 2 : `POST /operations` avec `{"operateur":"division","a":7,"b":2}`**
Statut `201 Created`, résultat `3.5`. L'identifiant est 5 et non 2, car le compteur continue de s'incrémenter à chaque création faite pendant mes essais.

![Test 2](images/postman%20post%20201%20div.png)

**Test 3 : `GET /operations`**
Statut `200 OK`, tableau JSON avec toutes les opérations enregistrées.

![Test 3](images/postman%20get%20200%20tab%20json.png)

**Test 4 : `GET /operations/2`**
Statut `200 OK`, l'opération d'identifiant 2 (produit de 6 et 7, résultat 42).

![Test 4](images/get%20%20200%20identifion%202.png)

**Test 5 : `GET /operations/99`**
Statut `404 Not Found`. Postman affiche la page d'erreur standard de Tomcat.

![Test 5](images/get%20operation%2099,%20404.png)

**Test 6 : `POST /operations` avec `{"operateur":"division","a":7,"b":0}`**
Statut `400 Bad Request`, corps `{"erreur":"Calcul impossible"}`.

![Test 6](images/get%20operations%20400%20div.png)

**Test 7 : `POST /operations` avec `{"operateur":"racine","a":9,"b":0}`**
Statut `400 Bad Request`, corps `{"erreur":"Calcul impossible"}` (opérateur inconnu).

![Test 7](images/get%20operations%20400%20racine.png)

### Test F.4 : Modifier une opération (PUT)

Requête : `PUT /operations/1` avec le corps `{"operateur":"produit","a":8,"b":2}`.
Résultat attendu : statut `200`, résultat `16.0`, puis `GET /operations/1` pour vérifier.

> **Capture à ajouter ici** (requête PUT, statut et corps obtenus).

### Tests F.5 : Supprimer une opération (DELETE)

**1. `DELETE /operations/1`** : statut `204 No Content`.

![DELETE 204](images/detele%201.png)

**2. `GET /operations/1`** : statut `404 Not Found` (l'opération n'existe plus).

![GET après suppression](images/get%20after%20delete.png)

**3. `DELETE /operations/1` une deuxième fois** : statut `404 Not Found`.

![DELETE 404](images/delete%20n2.png)

---

## Questions de synthèse

**1. Tableau de l'étape A.1 avec les codes de statut HTTP**

| Verbe | URI | Rôle | Codes de statut possibles |
|-------|-----|------|---------------------------|
| GET | `/calcul/somme?a=2&b=3` | Retourne a + b | 200 ; 404 si un paramètre n'est pas un entier |
| GET | `/calcul/soustraction?a=2&b=3` | Retourne a − b | 200 ; 404 si un paramètre n'est pas un entier |
| GET | `/calcul/produit?a=2&b=3` | Retourne a × b | 200 ; 404 si un paramètre n'est pas un entier |
| GET | `/calcul/division?a=6&b=3` | Retourne a ÷ b | 200 ; 400 si b = 0 ; 404 si un paramètre n'est pas un entier |
| POST | `/operations` | Crée une opération | 201 ; 400 si le calcul est impossible (415 si le corps n'est pas du JSON) |
| GET | `/operations` | Retourne toutes les opérations | 200 |
| GET | `/operations/{id}` | Retourne l'opération id | 200 ; 404 si l'id n'existe pas |
| PUT | `/operations/{id}` | Modifie l'opération id | 200 ; 400 si le calcul est impossible ; 404 si l'id n'existe pas |
| DELETE | `/operations/{id}` | Supprime l'opération id | 204 ; 404 si l'id n'existe pas |

**2. Annotations JAX-RS utilisées et leur rôle**

- `@ApplicationPath("/api")` : active JAX-RS et définit le préfixe commun de toutes les URI.
- `@Path` : indique l'URI d'une classe ou d'une méthode.
- `@GET`, `@POST`, `@PUT`, `@DELETE` : indiquent le verbe HTTP auquel la méthode répond.
- `@Produces` : indique le format de la réponse (texte brut ou JSON).
- `@Consumes` : indique le format attendu dans le corps de la requête (JSON).
- `@QueryParam` : lit un paramètre dans la chaîne de requête (après le `?`).
- `@PathParam` : lit un paramètre placé dans le chemin de l'URI.
- `@Context` : permet de récupérer des informations sur la requête, comme `UriInfo` pour construire l'URI de la nouvelle ressource.

**3. Différence entre JAX-RS et Jersey**
JAX-RS est une **spécification** : elle définit l'API et les annotations pour écrire des services REST en Java. Jersey est une **implémentation** de cette spécification : c'est le code qui fait réellement fonctionner les annotations.

**4. Rôle de Tomcat. Pourquoi un fichier `.war` et non un `.jar` exécutable ?**
Tomcat est un serveur d'applications web (conteneur de servlets). Il reçoit les requêtes HTTP, héberge l'application et l'exécute. Le projet produit un `.war` parce que c'est une **application web** qui doit être déployée dans un conteneur déjà installé, ici Tomcat. Un `.jar` exécutable, lui, démarre tout seul avec un `main` et n'a pas besoin de conteneur.

**5. Pourquoi garder la liste des opérations ne contredit pas le principe stateless ?**
Le principe stateless interdit au serveur de mémoriser **l'état du client** entre deux requêtes (une session, par exemple). Ici, la liste des opérations n'est pas l'état d'un client : ce sont des **ressources** de l'application, partagées par tous les clients. Chaque requête reste complète en elle-même : elle contient tout ce qu'il faut pour être comprise (le verbe, l'URI avec l'id, le corps), sans dépendre des requêtes précédentes.

**6. Même requête envoyée deux fois : POST ou PUT ?**
- Deux fois `POST /operations` : le serveur crée **deux opérations différentes** (par exemple id 1 et id 2). Chaque répétition change l'état du serveur. POST n'est pas idempotent.
- Deux fois `PUT /operations/1` : l'opération 1 est remplacée par les mêmes valeurs, donc le résultat final est **identique** après la première ou la deuxième requête. PUT est idempotent.
