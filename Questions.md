# TP MongoDB : Gestion d'un Parc de Réservation pour Activités Sportives

## **Contexte**

Vous travaillez pour une entreprise de réservation en ligne pour des activités sportives. Cette entreprise gère un large éventail d'activités : escalade, tennis, plongée, randonnées guidées, et bien plus encore. Les données sont stockées dans une base MongoDB, et vous devez utiliser des agrégations pour répondre à diverses analyses demandées par les gestionnaires.

---

## **Données**

Vous disposez de trois collections principales :

### 1. **`customers`**

Informations sur les clients :

```json
[
  {
    "_id": 1,
    "name": "Alice",
    "age": 30,
    "city": "Paris",
    "membership": "gold"
  },
  {
    "_id": 2,
    "name": "Bob",
    "age": 45,
    "city": "Lyon",
    "membership": "silver"
  },
  {
    "_id": 3,
    "name": "Charlie",
    "age": 35,
    "city": "Marseille",
    "membership": "gold"
  }
]
```

### 2. **`activities`**

Détails des activités disponibles :

```json
[
  { "_id": 101, "name": "Tennis", "category": "outdoor", "price": 20 },
  { "_id": 102, "name": "Escalade", "category": "indoor", "price": 30 },
  { "_id": 103, "name": "Plongée", "category": "outdoor", "price": 50 }
]
```

### 3. **`bookings`**

Réservations effectuées :

```json
[
  {
    "_id": 1,
    "customerId": 1,
    "activityId": 101,
    "date": "2023-12-01",
    "participants": 2
  },
  {
    "_id": 2,
    "customerId": 2,
    "activityId": 103,
    "date": "2023-12-02",
    "participants": 1
  },
  {
    "_id": 3,
    "customerId": 3,
    "activityId": 102,
    "date": "2023-12-03",
    "participants": 4
  }
]
```

---

## **Travail à Réaliser**

### **Phase 1 : Import des Données**

1. **Importer les données fournies** dans MongoDB via des scripts ou des fichiers JSON.

   - Créez les collections `customers`, `activities`, et `bookings` avec les documents correspondants.

   `db.insertMany([{...}, {...}, {...}])`

### **Phase 2 : Analyses Simples**

2. **Questions à résoudre :**

   - Combien de réservations ont été effectuées pour chaque activité ?

```js
db.reservations.aggregate([
  {
    $group: {
      _id: "$activityId",
      nombreDeReservations: { $sum: 1 },
    },
  },
  {
    $sort: { nombreDeReservations: -1 },
  },
]);
```

- Quelle est la ville avec le plus grand nombre de clients ?

```js
db.clients.aggregate([
  {
    $group: {
      _id: "$city",
      nombreDeClients: { $sum: 1 },
    },
  },
  {
    $sort: { nombreDeClients: -1 },
  },
  {
    $limit: 1,
  },
]);
```

**Réponse** Rouen

- Affichez les réservations où le prix total (participants × prix de l'activité) est supérieur à 100.

```js
db.reservations.aggregate([
  {
    $lookup: {
      from: "activities",
      localField: "activityId",
      foreignField: "_id",
      as: "activityDetails",
    },
  },
  {
    $unwind: "$activityDetails",
  },
  {
    $addFields: {
      prixTotal: { $multiply: ["$participants", "$activityDetails.price"] },
    },
  },
  {
    $match: {
      prixTotal: { $gt: 100 },
    },
  },
  {
    $project: {
      _id: 1,
      customerId: 1,
      activityId: 1,
      date: 1,
      participants: 1,
      prixTotal: 1,
    },
  },
]);
```

**Objectif :** Utiliser les opérateurs `$group`, `$match`, et `$project`.

### **Phase 3 : Agrégations Plus Complexes**

3. **Proposez des analyses pour aider la gestion :**

   - Quel est le revenu total généré par catégorie d’activité ?

   ```js
   db.bookings.aggregate([
     {
       $lookup: {
         from: "activities",
         localField: "activityId",
         foreignField: "_id",
         as: "activityDetails",
       },
     },
     {
       $unwind: "$activityDetails",
     },
     {
       $group: {
         _id: "$activityDetails.category",
         revenuTotal: {
           $sum: { $multiply: ["$participants", "$activityDetails.price"] },
         },
       },
     },
     {
       $sort: { revenuTotal: -1 },
     },
   ]);
   ```

   - Quelle est la dépense moyenne par client en fonction de leur type d'abonnement (`gold`, `silver`) ?

   - Quels sont les clients ayant réservé plusieurs fois la même activité ?

   **Objectif :** Mettre en pratique `$lookup`, `$unwind`, et `$addFields`.

### **Phase 4 : Traitement des Dates**

4. **Analyses temporelles :**

   - Combien de réservations ont été effectuées par mois en 2023 ?
   - Identifier les jours où les activités étaient totalement réservées (supposons une limite de 10 participants par activité).
   - Déterminer si une activité spécifique (par exemple, "Escalade") est plus populaire en semaine ou le week-end.

   **Objectif :** Travailler avec `$dateFromString`, `$month`, et `$dayOfWeek`.

---

## **Livrables**

1. **Script d'importation :** Fichier JavaScript ou commandes MongoDB pour insérer les documents.
2. **Résultats d'agrégation :** Copiez-collez ou capturez les résultats dans un fichier Markdown.
3. **Explication :** Chaque agrégation doit être expliquée dans le fichier Markdown. Expliquez le choix des opérateurs et leur rôle.

---

## **Bonus (facultatif)**

- Implémentez une **fonction Node.js** qui génère dynamiquement des statistiques (par exemple, revenu total, réservations mensuelles) à partir des données.
- Ajoutez une simulation où un client effectue une réservation, et mettez à jour les collections en conséquence (par exemple, en diminuant une capacité disponible pour une activité).

---

Bonne chance !
