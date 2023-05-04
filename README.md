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

Redis Stack's query and search capability can be used to create an index on vectors. The source code that I am using in this example wraps the Redis commands by using a `VectorDB` class. Let's take a look at some of the methods that I implemented for accessing Redis.

### `create_index`

As the name indicates, this method creates a search index. The default schema has a 'descr' text field, a 'labels' tag field, a numeric field called 'time' and a vector field called 'vec'.

The relevant code within this method is equivalent to:

```
idx_schema = (VectorField("vec", "HNSW", {"TYPE": "FLOAT32", "DIM": dimension, "DISTANCE_METRIC": "COSINE"}),
              TextField("descr"),
              TagField("labels", sortable=True),
              NumericField("time", sortable=True))
idx_def = IndexDefinition(prefix=["{}:".format(item_type)])
con.ft("idx:{}".format(index_name)).create_index(idx_schema, idx_def)
```

### `add`

This method adds a vector with meta data to the database. I use a [Redis hash](https://www.redis.io](https://redis.io/docs/data-types/hashes/) in this case, but you can also store vectors within a JSON with Redis Stack.

```
con.hset("{}:{}".format(item_type, item_id), mapping=data)
```

The data dictionary contains the fields `labels`, `descr`, `time`, and `vec`. The vector is stored as binary within the `vec` field. The library `numpy` is used to convert a more human-readable float vector (a Python list) to its byte string representation:

```
data["vec"] = np.array(vector).astype(np.float32).tobytes()
```

Here is an example of such a data dictionary:

```
data = {'time': 1683233711.983063, 'descr': 'Samuel is into books and comics', 'labels': 'books, comics', 'vec': b'fff?333?\xcd\xccL>'}
```

### `vector_search`

The `vector_search` method performs a vector similarity search. I decide to only return the id and vector score in my implementation. The vector query string takes a few arguments:

```
vector_query = "{}=>[KNN {} @{} $vector]".format(meta_data_query, num_neighbours, vector_field)
```

What I called `meta data query`  refers to a RediSearch query that is, in the first step, not related to the vector. This allows you to pre-filter based on additional meta data fields, such as the description (`desc`), or the `labels`.

You can then query the following way:

```
# The name of the vector score field depends on the name of the vector field
vector_score_field = "__{}_score".format("vec")
redis_query = Query(vector_query).return_fields("_id", vector_score_field).sort_by(vector_score_field, True).dialect(2)

con.ft("idx:{}".format(index_name)).search(redis_query, query_params={"vector": search_vector})
```

Please take a look at the [vector similarity search reference documentation](https://redis.io/docs/stack/search/reference/vectors/) for further details.


