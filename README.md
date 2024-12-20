# Analyses Simples\*\*

## Combien de réservations ont été effectuées pour chaque activité ?

```js
db.bookings.aggregate([
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

**Résultat :**

![Capture](/assets/Capture%20d'écran%202024-12-20%20135851.png)

---

## Quelle est la ville avec le plus grand nombre de clients ?

```js
db.customers.aggregate([
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

**Résultat :**

![Capture](/assets/Capture%20d'écran%202024-12-20%20140051.png)

---

## Affichez les réservations où le prix total (participants × prix de l'activité) est supérieur à 100.

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

**Résultat :**

![Capture](/assets/Capture%20d'écran%202024-12-20%20140221.png)

---

# Agrégations Plus Complexes

## Quel est le revenu total généré par catégorie d’activité ?

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

**Résultat :**

![Capture](/assets/Capture%20d'écran%202024-12-20%20140542.png)

---

### Quelle est la dépense moyenne par client en fonction de leur type d'abonnement (`gold`, `silver`) ?

```js
db.bookings.aggregate([
  {
    $lookup: {
      from: "clients",
      localField: "customerId",
      foreignField: "_id",
      as: "clientDetails",
    },
  },
  {
    $unwind: "$clientDetails",
  },
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
      _id: {
        customerId: "$customerId",
        membership: "$clientDetails.membership",
      },
      totalDepense: {
        $sum: { $multiply: ["$participants", "$activityDetails.price"] },
      },
    },
  },
  {
    $group: {
      _id: "$_id.membership",
      depenseMoyenne: { $avg: "$totalDepense" },
    },
  },
  {
    $sort: { depenseMoyenne: -1 },
  },
]);
```

![Capture](/assets/Capture%20d'écran%202024-12-20%20141031.png)

---

### Quels sont les clients ayant réservé plusieurs fois la même activité ?

```js
db.bookings.aggregate([
  {
    $group: {
      _id: {
        customerId: "$customerId",
        activityId: "$activityId",
      },
      count: { $sum: 1 },
    },
  },
  {
    $match: {
      count: { $gt: 1 },
    },
  },
  {
    $lookup: {
      from: "customers",
      localField: "_id.customerId",
      foreignField: "_id",
      as: "clientDetails",
    },
  },
  {
    $unwind: "$clientDetails",
  },
  {
    $lookup: {
      from: "activities",
      localField: "_id.activityId",
      foreignField: "_id",
      as: "activityDetails",
    },
  },
  {
    $unwind: "$activityDetails",
  },
  {
    $project: {
      _id: 0,
      customerId: "$_id.customerId",
      customerName: "$clientDetails.name",
      activityId: "$_id.activityId",
      activityName: "$activityDetails.name",
      count: 1,
    },
  },
]);
```

**Résultat**

![Capture](/assets/Capture%20d'écran%202024-12-20%20141844.png)

---

# Analyses temporelles :

### Combien de réservations ont été effectuées par mois en 2023 ?

```js
db.bookings.aggregate([
  {
    $match: {
      date: { $gte: "2023-01-01", $lt: "2024-01-01" },
    },
  },
  {
    $project: {
      date: { $dateFromString: { dateString: "$date" } },
      month: { $month: { $dateFromString: { dateString: "$date" } } },
    },
  },
  {
    $group: {
      _id: "$month",
      totalBookings: { $sum: 1 },
    },
  },
  {
    $sort: { _id: 1 },
  },
]);
```

![Capture](/assets/Capture%20d'écran%202024-12-20%20144316.png)

---

### Identifier les jours où les activités étaient totalement réservées (supposons une limite de 10 participants par activité).

```js
db.bookings.aggregate([
  {
    $group: {
      _id: {
        date: "$date",
        activityId: "$activityId",
      },
      totalParticipants: { $sum: "$customers" },
    },
  },
  {
    $match: {
      totalParticipants: { $lte: 10 },
    },
  },
  {
    $lookup: {
      from: "activities",
      localField: "_id.activityId",
      foreignField: "_id",
      as: "activityDetails",
    },
  },
  {
    $unwind: "$activityDetails",
  },
  {
    $project: {
      _id: 0,
      date: "$_id.date",
      activityId: "$_id.activityId",
      activityName: "$activityDetails.name",
    },
  },
]);
```

![Capture](/assets/Capture%20d'écran%202024-12-20%20145558.png)

---

### Déterminer si une activité spécifique (par exemple, "Escalade") est plus populaire en semaine ou le week-end.

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
    $match: {
      "activityDetails.name": "Escalade",
    },
  },
  {
    $project: {
      date: 1,
      participants: 1,
      dayOfWeek: { $dayOfWeek: { $dateFromString: { dateString: "$date" } } },
    },
  },
  {
    $group: {
      _id: {
        weekType: {
          $cond: [{ $in: ["$dayOfWeek", [1, 7]] }, "weekend", "weekday"],
        },
      },
      totalParticipants: { $sum: "$participants" },
    },
  },
  {
    $sort: { _id: 1 },
  },
]);
```

![Capture](/assets/Capture%20d'écran%202024-12-20%20151108.png)
