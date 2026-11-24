# TP1 – SOAP et WSDL
## Partie B : Contract-First

**Binôme :** 24012 / 24163

---

# Étape 5 – Générer le code depuis le WSDL

### Question 1. Dans ce WSDL, retrouvez l'opération, son message de fault et l'action SOAP à envoyer.

**Réponse :**

- **Opération :** `emprunterLivre`
- **Message de fault :** `LivreIndisponibleFault`, qui pointe vers l'élément `LivreIndisponible`.
- **Action SOAP :** `http://supnum.mr/bib/emprunterLivre` (attribut `soapAction` de `<soap:operation>`).

### Question 2. Quelle est la signature de `emprunterLivre` dans `EmpruntPortType` ?

**Réponse :**

```java
EmpruntType emprunterLivre(String idAdherent, String isbn)
    throws LivreIndisponibleFault;
```

### Question 3. Quel type Java a été généré pour `dateRetour` ? Comparez avec la partie A.

**Réponse :**

`dateRetour` est généré en `XMLGregorianCalendar`, car le WSDL le déclare de type `xs:date`. En Partie A, il était représenté par un simple `String`.

### Question 4. Quel est le nom de la classe d'exception générée, et d'où vient ce nom dans le WSDL ?

**Réponse :**

La classe d'exception générée est `LivreIndisponibleFault`. Son nom provient de `<wsdl:message name="LivreIndisponibleFault">`.

`LivreIndisponible` (sans le suffixe `Fault`) est la classe de données transportée par cette exception.

---

# Étape 6 – Implémenter et comparer

### Question 1. Le WSDL publié est-il celui que vous avez fourni ? Contient-il le motif de l'ISBN et le type `xs:date` ?

**Réponse :**

Oui, le WSDL publié est celui que j'ai fourni. Le service republie le fichier du projet grâce à l'attribut `wsdlLocation`.

Il contient :

- Le motif de validation de l'ISBN (`xs:pattern`) :
  ```xml
  97[89]-\d-\d{2,5}-\d{2,6}-\d
  ```
  défini dans `IsbnType`.
- Le type `xs:date` pour `dateRetour`.

Le schéma est directement intégré dans le WSDL (pas de `?xsd=1`), contrairement à la Partie A où le WSDL était généré avec `xs:string` et sans motif de validation.

### Question 2. Avez-vous dû modifier `01-emprunter.xml` pour qu'il fonctionne ici ? Pourquoi ?

**Réponse :**

Oui, j'ai dû modifier le fichier `01-emprunter.xml`.

En Partie A, les paramètres `idAdherent` et `isbn` étaient non qualifiés. Ici, l'attribut `elementFormDefault="qualified"` impose l'utilisation de l'espace de noms `http://supnum.mr/bib`. Il faut donc écrire :

```xml
<bib:idAdherent>
<bib:isbn>
```

Sans cette modification, le service ne reconnaît pas les paramètres, `isbn` vaut `null` et la requête échoue avec la Fault suivante :

```text
S:Server : Cannot invoke "String.equals(Object)" because "isbn" is null
```

Après qualification des paramètres, le service répond `200 OK` avec le numéro d'emprunt `E-2026-0812`.

### Question 3. Sans `@SchemaValidation`, l'ISBN `978-2-10` est-il refusé ? Et avec ? Quelle Fault obtenez-vous ?

**Réponse :**

- **Sans `@SchemaValidation` :** l'ISBN `978-2-10` n'est pas refusé. Le service répond `200 OK` avec un numéro d'emprunt, car le motif est uniquement défini dans le WSDL et n'est pas vérifié par le code Java.
- **Avec `@SchemaValidation` :** l'ISBN est refusé. Une Fault `S:Client` (HTTP 500) est retournée avec l'erreur de validation du schéma.

Erreur obtenue :

```text
org.xml.sax.SAXParseException;
cvc-pattern-valid: Value '978-2-10' is not facet-valid
with respect to pattern
'97[89]-\d-\d{2,5}-\d{2,6}-\d'
for type 'IsbnType'.
```

Le serveur valide alors le message SOAP contre le schéma avant d'exécuter le code Java.

### Question 4. Retirez `wsdlLocation` de l'annotation et redémarrez : qu'est-ce qui change dans le WSDL publié ?

**Réponse :**

Sans `wsdlLocation`, le service ne publie plus le fichier WSDL fourni. Il génère automatiquement le WSDL à partir des classes Java, comme dans l'approche **Code-First**.

Les changements observés sont :

- Le schéma est séparé du WSDL : un `xsd:import` pointe vers `?xsd=1`.
- Le nom du binding devient `EmpruntPortBinding` au lieu de `EmpruntSoapBinding`.
- Le fault devient `LivreIndisponibleFault` au lieu de `LivreIndisponible`.
- Les messages deviennent `emprunterLivre` et `emprunterLivreResponse` au lieu de `emprunterLivreRequest`.
- Des attributs `wsam:Action` apparaissent.

Le contrat n'est donc plus celui que nous avons écrit. L'ISBN `978-2-10` est de nouveau accepté (`200 OK`), malgré la présence de `@SchemaValidation`, car le schéma généré ne contient plus le motif de validation.

---

# Conclusion – Choisir et rendre

### Question 1. Citez deux différences que vous avez observées entre le WSDL de la Partie A et celui de la Partie B.

**Réponse :**

Les deux principales différences observées sont :

1. **Le schéma :** en Partie A, il est séparé du WSDL (`?xsd=1`), tandis qu'en Partie B, il est intégré directement au WSDL.
2. **Les types de données :** en Partie A, `isbn` et `dateRetour` sont de type `xs:string`, sans motif de validation. En Partie B, `isbn` utilise `IsbnType` avec un motif de validation, et `dateRetour` est de type `xs:date`.

D'autres différences existent également : les paramètres sont non qualifiés en Partie A et qualifiés en Partie B, le `soapAction` est vide en Partie A et défini en Partie B, et les noms du binding et du fault diffèrent.

### Question 2. Si on renomme le champ `dateRetour` dans le code, quel projet voit son contrat changer ? Quel risque pour les clients ?

**Réponse :**

C'est le projet **Code-First** dont le contrat change, car son WSDL est généré automatiquement à partir du code Java.

En **Contract-First**, le WSDL est un fichier de référence indépendant du code. Il ne change pas tant qu'on ne le modifie pas explicitement.

Le risque pour les clients est une incompatibilité : un client généré avec l'ancien WSDL continuera d'attendre le champ `dateRetour`. Si ce champ est renommé, le client peut recevoir des données manquantes ou rencontrer des erreurs, sans que cette modification ait été prévue dans le contrat.

### Question 3. Quelle approche choisiriez-vous pour le service Paiement, appelé par Scolarité (.NET), Bibliothèque (PHP) et le portail (Node.js) ? Et pour un prototype interne de deux jours ?

**Réponse :**

- **Service Paiement : Contract-First.** Plusieurs équipes et langages utilisent le service. Le contrat doit donc être défini, validé et versionné avant le développement, afin de garantir sa stabilité et sa compatibilité entre les différents clients.
- **Prototype interne de deux jours : Code-First.** Cette approche est plus rapide lorsqu'il y a peu de consommateurs et que le développement doit être réalisé rapidement. Il faudra cependant figer le WSDL avant la mise en production.

---
