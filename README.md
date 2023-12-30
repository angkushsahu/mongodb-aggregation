# MONGO AGGREGATION PIPELINE

From: https://youtube.com/playlist?list=PLRAV69dS1uWQ6CZCehxKy0rjkqhQ2Z88t&si=9OL51KHnjwc9anUa

### Table of Contents

1. [Preparation](#preparation)
1. [Match](#match-operator)
1. [Count](#count-operator)
1. [Group](#group-operator)
1. [Unwind](#unwind-operator)
1. [Push](#push-operator)
1. [Lookup](#lookup-operator)

## Preparation

Create a database with any desired name, I am taking `aggregation`. Create 3 collections in it with the following names: `users`, `books` and `authors`.

I advise you to keep the name of the database and the collections same as mine for simplicity while following along.

You can use local mongodb compass or mongodb atlas depending upon your preference.

In the `authors` collections, insert the following data:

```json
[
    {
      "_id": 100,
      "name": "F. Scott Fitzgerald",
      "birth_year": 1896
    },
    {
      "_id": 101,
      "name": "George Orwell",
      "birth_year": 1903
    },
    {
      "_id": 102,
      "name": "Harper Lee",
      "birth_year": 1926
    }
]
```

In the `books` collection, insert the following data:

```json
[
    {
      "_id": 1,
      "title": "The Great Gatsby",
      "author_id": 100,
      "genre": "Classic"
    },
    {
      "_id": 2,
      "title": "Nineteen Eighty-Four",
      "author_id": 101,
      "genre": "Dystopian"
    },
    {
      "_id": 3,
      "title": "To Kill a Mockingbird",
      "author_id": 102,
      "genre": "Classic"
    }
]
```

For the last `users` collection, insert the data provided in the [`users.json`](/users.json) file present in the root of this repository. It is a huge chunk of data so I avoided using it here.

The command for inserting these data in their respective collections are as follows:

```js
db.authors.insertMany( /* Copy paste the data from above */ );
db.books.insertMany( /* Copy paste the data from above */ );
db.users.insertMany( /* Copy paste the data from the users.json file */ );
```

## Match Operator

```js
[
    {
        $match: {
            isActive: true
        }
    }
]
```

## Count Operator

![Count](/images/count.png)

```js
[
    {
        $match: {
            isActive: true
        }
    },
    {
       $count: "activeUsers"
    }
]
```

## Group Operator

![Group](/images/group.png)

```js
[
  {
    $group: {
       _id: "$gender" // _id is mandatory in group pipeline
    }
  }
]
```

The following query groups all the fields in one document

![Group-2](/images/group2.png)

```js
[
  {
    $group: {
      _id: null
    }
  }
]
```

The following query groups all the fields then calculates the average age of all the users in the database

![Average-calculation](/images/group3.png)

```js
[
  {
    $group: {
      _id: null,
      averageAge: {
        "$avg": "$age"
      }
    }
  }
]
```

The following query groups the male and female gender fields separately then calculates the average age of all the users in the database

![Average-calculation](/images/group4.png)

```js
[
  {
    $group: {
      _id: "$gender",
      averageAge: {
        "$avg": "$age"
      }
    }
  }
]
```

The following query calculates the total number of documents from each category

![Total number of documents in each category](/images/group5.png)

```js
[
  {
    $group: {
      _id: "$favoriteFruit",
      count: {
        $sum: 1
      }
    },
  },
]
```

The following query sorts the output of the results fetched in the previous query in ascending order of `count`

![Group-and-sort](/images/group6.png)

```js
[
  {
    $group: {
      _id: "$favoriteFruit",
      count: {
        $sum: 1
      }
    },
  },
  {
    $sort: {
      count: 1 // count: -1 sorts in descending order
    }
  }
]
```

The following query limits the number of results returned in the last query

![Group-and-limit](/images/group7.png)

```js
[
  {
    $group: {
      _id: "$favoriteFruit",
      count: {
        $sum: 1
      }
    },
  },
  {
    $sort: {
      count: 1 // count: -1 sorts in descending order
    }
  },
  {
    $limit: 1
  }
]
```

## Unwind Operator

If the document has an array, then the unwind operator separates each element in the document and creates new documents with each document containing exactly one array element (this may create duplicate entries from the same document)

For example, consider the following document:

```js
users: {
    name: "Angkush Sahu",
    tags: ["coder", "ironman"]
}
```

Here, if we use the unwind operator, it will product the following result

```js
{
    name: "Angkush Sahu",
    tags: "coder"
},
{
    name: "Angkush Sahu",
    tags: "ironman"
}
```

![Unwind](/images/unwind.png)

Let's say we want to find out the average number of tags per user in our database

To achieve this, let's follow the steps below:

1. Unwind the tags operator so that it contains documents containing one tag only.
1. Group these documents as per their _id, this will remove duplication of the same document and as the _id is unique, each user will be listed. Then find the numberOfTags using the $sum operator.
1. Group the output and calculate the average as taught before.

The outputs from each step are listed below

1. Output from the first step
![Unwind-2](/images/unwind2.png)

1. Output from the second step
![Unwind-2](/images/unwind3.png)

1. Output from the third step
![Unwind-2](/images/unwind4.png)

This is not the best solution to the problem stated, this was taught only for the sake of explaining the unwind operator.

A better approach to the problem above is shown below:

![Better-than-unwind](/images/better-unwind-option.png)

```js
[
  {
    $addFields: {
      numberOfTags: {
        $size: {
          $ifNull: ["$tags", []]
        },
      }
    }
  },
  {
    $group: {
      _id: null,
      averageNumberOfTags: {
        $avg: "$numberOfTags"
      }
    }
  }
]
```

## Push Operator

Get all the users having common favoriteFruit

![Push](/images/push.png)

```js
[
  {
    $group: {
      _id: "$favoriteFruit",
      users: {
        $push: "$name"
      }
    }
  }
]
```

## Lookup Operator

Joining two collections

Syntax:
```js
{
    $lookup: {
        from: "name_of_the_other_collection",
        localField: "id_field_name_which_contains_id_from_the_other_collection",
        foreignField: "_id",
        as: "name_of_the_variable_where_you_want_to_store_modified_data"
    }
}

// We are taking _id here, however, it is not necessary that you can create joints using _id field only

// In our case, the lookup table will look like this
// joining books collection with authors collection
{
    $lookup: {
        from: "authors",
        localField: "author_id",
        foreignField: "_id",
        as: "author_details"
    }
}
```

![Lookup](/images/lookup.png)

Since, our data is coming in array format and each of the book documents can contain only one author each, therefore, it will be better if we display it in object format other than array format containing only one element, to perform this, use the following code snippet

![Lookup-with-field-addition](/images/lookup2.png)

```js
[
  {
    $lookup: {
      from: "authors",
      localField: "author_id",
      foreignField: "_id",
      as: "author_details"
    }
  },
  {
    $addFields: {
      author_details: {
        $arrayElemAt: ["$author_details", 0]
      }
    }
  }
]
```
