# Redis as a Vector Database for Recommendations

This repository contains example code for using Redis Stack's vector similarity search as a recommendation engine.

The idea is quite simple:

1. The interests of a user are expressed as a vector. Each component of the vector is associated to one category of interest.
2. If we know the interests of a specific user, we can search for the K-N(earest)N(eighbours) to find other users that share the same interests.
4. We can then inspect the behaviour of these users (e.g., the purchase history) to recommend our user specific products.

The following table shows the vector `[0.9, 0.7, 0.2]`:

|books|comics|computers|
|---|---|---|
|0.9|0.7|0.2|


Let's furthermore assume that a user is only interested in a specific category if the interest value is larger than the threshold of `0.4`, which means that our user is interested in 'books' and 'comics', but he is not really into 'computers'.

## Redis as a Vector Database

Redis Stack's query and search capability can be used to create an index on vectors. The source code that I am using in this example wraps the Redis commands by using a `VectorDB` class. Let's take a look at some of the methods that I implemented for accessing Redis:

* `create_index`: Creates a search index. The default schema has a 'descr' text field, a 'labels' tag field, a numeric field called 'time' and a vector field called 'vec'.

The relevant code within this method is equivalent to:

```
idx_schema = (VectorField("vec", "HNSW", {"TYPE": "FLOAT32", "DIM": dimension, "DISTANCE_METRIC": "COSINE"}),
              TextField("descr"),
              TagField("labels", sortable=True),
              NumericField("time", sortable=True))
idx_def = IndexDefinition(prefix=["{}:".format(item_type)])
con.ft("idx:{}".format(index_name)).create_index(idx_schema, idx_def)
```

* 


