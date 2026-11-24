# TP1 – SOAP et WSDL
## Partie A – Code-first 

##Matricules : 24012 - 24163 


### Question 1 : Quelle annotation fait de `EmpruntService` un service web ? Quelle annotation fait de `LivreIndisponibleException` une erreur déclarée dans le contrat ?

- L'annotation `@WebService` transforme la classe `EmpruntService` en service web SOAP. Elle permet à JAX-WS de publier les méthodes de cette classe comme des opérations web et de générer automatiquement le WSDL.
- L'annotation `@WebFault` déclare que `LivreIndisponibleException` représente une erreur SOAP déclarée dans le contrat du service.

### Question 2 : Pourquoi les classes de données doivent-elles avoir un constructeur vide ?

Les classes de données doivent avoir un constructeur vide pour permettre à JAX-WS de créer automatiquement leurs objets lors de la désérialisation des messages XML reçus.

Ce constructeur facilite la conversion entre les données XML et les objets Java.

---

# Étape 2 – Lire le WSDL généré

### Question 1 : Quelle est l'URL du service ? Quelles sont les deux opérations et leurs messages d'entrée et de sortie ?

L'URL du service est :

`http://localhost:8080/bib/EmpruntService`

Le WSDL est accessible à l'adresse :

`http://localhost:8080/bib/EmpruntService?wsdl`

Le service contient deux opérations :

**1. emprunterLivre**
- Entrée : `idAdherent` et `isbn`.
- Sortie : `emprunt`, qui contient le numéro de l'emprunt et la date de retour.
- Cette opération peut également produire une erreur `LivreIndisponible`.

**2. rendreLivre**
- Entrée : `numeroEmprunt`.
- Sortie : `penalite`, de type décimal (`BigDecimal`).

### Question 2 : Quel style et quel use le binding déclare-t-il ?

Le binding généré par JAX-WS utilise normalement le style `document` et le mode `literal`.

Cela signifie que les messages SOAP sont structurés sous forme de documents XML et que les données suivent directement les types définis dans le schéma XSD.

**À vérifier dans le WSDL :** `style="document"` et `use="literal"`.

### Question 3 : Est-ce du SOAP 1.1 ou 1.2 ?

Le service publié par défaut avec JAX-WS utilise SOAP 1.1, sauf configuration explicite d'un binding SOAP 1.2.

On peut le reconnaître grâce à l'espace de noms :

`http://schemas.xmlsoap.org/soap/envelope/`

### Question 4 : Quel type XSD a été généré pour `isbn` et pour `dateRetour` ? La règle de format de l'ISBN apparaît-elle ?

- `isbn` est de type `xsd:string`.
- `dateRetour` est également de type `xsd:string`, car les dates sont représentées sous forme de chaînes de caractères dans les classes Java.

La règle de validation de l'ISBN définie avec `Pattern.compile()` dans le code Java n'apparaît pas automatiquement dans le XSD.

Le XSD indique donc seulement que l'ISBN est une chaîne de caractères, sans préciser son format exact.

### Question 5 : Dans la requête SoapUI, les paramètres `idAdherent` et `isbn` portent-ils un préfixe d'espace de noms ?

Les paramètres appartiennent à l'espace de noms du service :

`http://supnum.mr/bib`

SoapUI peut les afficher avec un préfixe, par exemple `bib:idAdherent` et `bib:isbn`, ou utiliser un espace de noms par défaut.

Le préfixe `bib` est simplement un raccourci XML qui désigne cet espace de noms.

### Question 6 : Dans l'onglet Raw, où voyez-vous HTTP, XML, SOAP ? Quel rôle joue chacun de ces standards, et quel rôle joue le WSDL ?

- **HTTP :** assure le transport de la requête et de la réponse entre le client et le serveur.
- **XML :** représente les données sous forme structurée et lisible par les machines.
- **SOAP :** définit le format des messages échangés, notamment l'Envelope, le Header et le Body.
- **WSDL :** décrit le contrat du service web : ses opérations, ses paramètres, ses réponses, ses types de données et son adresse.

---

# Étape 3 – Envoyer des messages SOAP

### Question 1 : Pourquoi la requête utilise-t-elle POST, même pour lire une information ?

SOAP utilise généralement la méthode HTTP `POST` pour envoyer les messages au serveur.

La requête SOAP contient une enveloppe XML dans le corps de la requête HTTP. Même lorsqu'une opération consiste à lire une information, le message SOAP doit être transmis au serveur pour être traité.

### Question 2 : Le bloc `jeton` du Header a-t-il empêché le traitement ? Pourquoi ?

Non, le bloc `jeton` n'a pas empêché le traitement dans cette configuration.

Le service ne contient pas de code qui vérifie ce jeton et il ne déclare pas non plus ce Header comme obligatoire dans son contrat.

Par conséquent, le serveur peut l'ignorer lorsqu'il ne porte pas l'attribut `mustUnderstand="1"`.

### Question 3 : Que répond le service au message SOAP 1.2 ? Expliquez avec le binding lu à l'étape 2.

Le service utilise un binding SOAP 1.1 par défaut, alors que la requête envoyée utilise l'espace de noms SOAP 1.2.

Il existe donc une incompatibilité entre la version SOAP attendue par le service et celle utilisée dans la requête.

Le serveur peut répondre par une erreur SOAP de type `VersionMismatch`, avec un code HTTP d'erreur.

### Question 4 : Listez les cinq erreurs du dernier message et leur correction. Certaines ne déclenchent aucune erreur côté serveur : lesquelles, et pourquoi ?

Les cinq erreurs à corriger sont :

1. **Mauvaise casse de l'élément Envelope :** `soap:envelope` doit être remplacé par `soap:Envelope`.
2. **Mauvais espace de noms du service :** `http://supnum.mr/bibliotheque` doit être remplacé par `http://supnum.mr/bib`.
3. **Mauvais nom du paramètre :** `numero` doit être remplacé par `numeroEmprunt`.
4. **Mauvais ordre des éléments :** le Header doit être placé avant le Body.
5. **Mauvaise casse de la balise de fermeture :** la balise fermante doit correspondre exactement à la balise ouvrante `soap:Envelope`.

Certaines erreurs peuvent être rejetées directement par le moteur SOAP, notamment la mauvaise casse de l'enveloppe et le mauvais ordre du Header et du Body.

En revanche, une erreur de valeur ou de paramètre peut parfois ne pas être détectée immédiatement par le moteur SOAP si le message reste XML valide. Elle peut ensuite provoquer une erreur lors de l'appel de l'opération.

---

# Étape 4 – Provoquer et construire des Faults

### Question 1 : Pour les cas 1 et 2, quels `faultcode` et `faultstring` recevez-vous, et quel code HTTP ?

**Cas 1 : ISBN mal formé (`978-2-10`)**

Le service détecte que le format de l'ISBN est incorrect et déclenche une `SOAPFaultException`.

Le code du Fault est normalement `soap:Client`, car l'erreur provient des données envoyées par le client.

Le message d'erreur est :

`ISBN mal formé : 978-2-10`

Le code HTTP attendu est `500`.

**Cas 2 : Livre déjà emprunté (`978-2-10-074444-4`)**

Le service détecte que le livre est déjà emprunté et déclenche une `LivreIndisponibleException`.

Le message d'erreur est :

`Livre indisponible`

Le détail du Fault contient les informations sur le livre, notamment son ISBN et sa date de retour prévue.

Le code HTTP attendu est `500`.

**Remarque :** recopier les valeurs exactes de `faultcode`, `faultstring` et du code HTTP affichées dans SoapUI, car le code du Fault du deuxième cas dépend de la réponse réellement produite par le runtime JAX-WS.

### Question 2 : Dans le cas 2, que contient `<detail>` ? Où sa structure est-elle décrite dans le WSDL ?

Le bloc `<detail>` contient les informations structurées concernant l'erreur `LivreIndisponible`.

Il contient notamment :
- `isbn` : l'identifiant ISBN du livre indisponible.
- `retourPrevu` : la date prévue pour le retour du livre.

La structure de ces informations est décrite dans le WSDL, à l'intérieur du schéma XSD généré pour l'exception `LivreIndisponible`.

### Question 3 : Quel code JAX-WS a-t-il choisi pour le livre indisponible ? Est-ce une erreur du client ou du serveur ? Qu'auriez-vous choisi ?

Il faut relever le `faultcode` réellement reçu dans SoapUI pour l'exception `LivreIndisponibleException`.

Sur le plan fonctionnel, le livre est déjà emprunté. Il s'agit donc d'une situation liée à la demande du client et non d'une panne technique du serveur.

J'aurais choisi `Client` (ou `Sender` en SOAP 1.2), car le client demande un livre qui n'est pas disponible.

### Question 4 : Que se passe-t-il avec `mustUnderstand="1"` ? Et avec `"0"` ?

L'attribut `mustUnderstand` indique si le destinataire doit obligatoirement comprendre et traiter un élément du Header SOAP.

- Avec `mustUnderstand="1"`, le serveur doit comprendre et traiter le Header. Si l'élément est inconnu ou non pris en charge, il peut renvoyer une Fault `MustUnderstand`.
- Avec `mustUnderstand="0"`, le serveur n'est pas obligé de traiter cet élément. Il peut donc l'ignorer.

Dans notre service, le jeton n'est pas traité par une logique d'authentification.

### Question 5 : Quelle différence voyez-vous entre votre Fault 1.2 et les Faults 1.1 reçues ?

La structure des Faults SOAP 1.1 et SOAP 1.2 est différente.

**SOAP 1.1 :**
- Utilise `<faultcode>`.
- Utilise `<faultstring>` pour décrire l'erreur.
- Utilise `<detail>` pour les informations supplémentaires.

**SOAP 1.2 :**
- Utilise `<Code>` et `<Value>` pour identifier le type d'erreur.
- Utilise `<Reason>` et `<Text>` pour expliquer l'erreur.
- Utilise `<Detail>` pour les informations supplémentaires.

SOAP 1.2 utilise également des codes comme `Sender` et `Receiver`, alors que SOAP 1.1 utilise notamment `Client` et `Server`.

---

