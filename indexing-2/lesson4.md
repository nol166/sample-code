# Code Summary: Indexing II - Lesson 4 - Wildcard Indexes

The code in this video uses some sample documents that you can insert into your MongoDB instance or Atlas Cluster with the following command in mongosh:

```js
db.getSiblingDB("sample_products").products.insertMany([
  {
    _id: new ObjectId("64a36318574fd20cd8fb9798"),
    sku: 111,
    product_name: "Stero Speakers",
    price: 100,
    stock: 5,
    product_attributes: { color: "black", size: "5x5x5", weight: "5lbs" },
  },
  {
    _id: new ObjectId("64a36318574fd20cd8fb9799"),
    sku: 121,
    product_name: "Bread",
    price: 2,
    stock: 50,
    product_attributes: {
      type: "white",
      calories: 100,
      weight: "24g",
      crust: "soft",
    },
  },
  {
    _id: new ObjectId("64a36318574fd20cd8fb979a"),
    sku: 131,
    product_name: "Milk",
    price: 3,
    stock: 20,
    product_attributes: {
      type: "2%",
      calories: 120,
      weight: "1L",
      brand: "Dairy Farmers",
    },
  },
]);

```

To create a wildcard index on the product attributes field, use the `db.collection.createIndex()` method, passing in the name of the field you want to index appended with a dot, dollar sign and two asterisks:

```js
db.products.createIndex({ "product_attributes.$**" : 1 })
```


To test the query using the explain method, use dot notation to inspect the winning plan and confirm that the wildcard index is being used:

```js
db.products.find({
  "product_attributes.crust": false,
}).explain().queryPlanner.winningPlan
```

To specify which fields to include or exclude in a wildcard index, set the value of a given field in a wildcard projection to 1 to include or 0 to exclude.  To include the `_id`, while excluding the `stock`, and `price` fields, use the following command:

```js
db.products.createIndex(
  { "$**": 1 },
  { wildcardProjection: { _id: 1, stock: 0, prices: 0 } }
)
```

To test the wildcard index with an query on the `sku` field, run the following query to return the `winningPlan` object from the explain output:

```js
db.products.find({ sku: 111 }).explain().queryPlanner.winningPlan
```

To test the wildcard index even further, run another query on one of the product attribute fields, like so:

```js
db.products.find({
  "product_attributes.crust": false
}).explain().queryPlanner.winningPlan
```

To create a compound wildcard index (starting in MongoDB 7.0) on the stock field and all of the product attributes fields, use the following command:

```js
db.products.createIndex({
  stock: 1, "product_attributes.$**" : 1 
})
```

