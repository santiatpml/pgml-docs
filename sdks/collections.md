# Collections



Collections are the organizational building blocks of the SDK. They manage all documents and related chunks, embeddings, tsvectors, and pipelines.

## Creating Collections

By default, collections will read and write to the database specified by `DATABASE_URL`.

### **Default `DATABASE_URL`**

{% tabs %}
{% tab title="Python" %}
```python
collection = Collection("test_collection")
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
collection = pgml.newCollection("test_collection")
```
{% endtab %}
{% endtabs %}

### **Custom DATABASE\_URL**

Create a Collection that reads from a different database than that set by the environment variable `DATABASE_URL`.

{% tabs %}
{% tab title="Python" %}
```python
collection = Collection("test_collection", CUSTOM_DATABASE_URL)
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
collection = pgml.newCollection("test_collection", CUSTOM_DATABASE_URL)
```
{% endtab %}
{% endtabs %}

```
```

## Upserting Documents

Documents are dictionaries with two required keys: `id` and `text`. All other keys/value pairs are stored as metadata for the document.

**Upsert documents with metadata**

{% tabs %}
{% tab title="Python" %}
```python
documents = [
    {
        "id": "Document 1",
        "text": "Here are the contents of Document 1",
        "random_key": "this will be metadata for the document"
    },
    {
        "id": "Document 2",
        "text": "Here are the contents of Document 2",
        "random_key": "this will be metadata for the document"
    }
]
collection = Collection("test_collection")
await collection.upsert_documents(documents)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
  const documents = [
            {
              id: "Document One",
              text: "document one contents...",
            },
            {
              id: "Document Two",
              text: "document two contents...",
            },
    ];
    await collection.upsert_documents(documents);
```
{% endtab %}
{% endtabs %}

```
```

## Searching Collections

The Python SDK is specifically designed to provide powerful, flexible vector search.

Pipelines are required to perform search. See the [Pipelines Section](https://github.com/postgresml/postgresml/tree/master/pgml-sdks/rust/pgml/python#pipelines) for more information about using Pipelines.

### **Basic vector search**

{% tabs %}
{% tab title="Python" %}
```python
collection = Collection("test_collection")
pipeline = Pipeline("test_pipeline")
results = await collection.query().vector_recall("Why is PostgresML the best?", pipeline).fetch_all()
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
collection = pgml.newCollection("test_collection")
pipeline = pgml.newPipeline("test_pipeline")
results = await collection.query().vector_recall("Why is PostgresML the best?", pipeline).fetch_all()
```
{% endtab %}
{% endtabs %}

### **Vector search with custom limit**

{% tabs %}
{% tab title="Python" %}
```python
collection = Collection("test_collection")
pipeline = Pipeline("test_pipeline")
results = await collection.query().vector_recall("Why is PostgresML the best?", pipeline).limit(10).fetch_all()
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
collection = pgml.newCollection("test_collection")
pipeline = pgml.newPipeline("test_pipeline")
results = await collection.query().vector_recall("Why is PostgresML the best?", pipeline).limit(10).fetch_all()
```
{% endtab %}
{% endtabs %}

### **Metadata Filtering**

We provide powerful and flexible arbitrarly nested metadata filtering based off of [MongoDB Comparison Operators](https://www.mongodb.com/docs/manual/reference/operator/query-comparison/). We support each operator mentioned except the `$nin`.

**Vector search with $eq metadata filtering**

{% tabs %}
{% tab title="Python" %}
```python
collection = Collection("test_collection")
pipeline = Pipeline("test_pipeline")
results = (
    await collection.query()
    .vector_recall("Here is some query", pipeline)
    .limit(10)
    .filter({
        "metadata": {
            "uuid": {
                "$eq": 1
            }    
        }
    })
    .fetch_all()
)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
collection = pgml.newCollection("test_collection")
pipeline = pgml.newPipeline("test_pipeline")
results = await collection.query()
    .vector_recall("Here is some query", pipeline)
    .limit(10)
    .filter({
        "metadata": {
            "uuid": {
                "$eq": 1
            }    
        }
    })
    .fetch_all()
```
{% endtab %}
{% endtabs %}

The above query would filter out all documents that do not contain a key `uuid` equal to `1`.

**Vector search with $gte metadata filtering**

{% tabs %}
{% tab title="Python" %}
```python
collection = Collection("test_collection")
pipeline = Pipeline("test_pipeline")
results = (
    await collection.query()
    .vector_recall("Here is some query", pipeline)
    .limit(10)
    .filter({
        "metadata": {
            "index": {
                "$gte": 3
            }    
        }
    })
    .fetch_all()
)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
collection = pgml.newCollection("test_collection")
pipeline = pgml.newPipeline("test_pipeline")
results = await collection.query()
    .vector_recall("Here is some query", pipeline)
    .limit(10)
    .filter({
        "metadata": {
            "index": {
                "$gte": 3
            }    
        }
    })
    .fetch_all()
)
```
{% endtab %}
{% endtabs %}

The above query would filter out all documents that do not contain a key `index` with a value greater than `3`.

**Vector search with $or and $and metadata filtering**

{% tabs %}
{% tab title="Python" %}
```python
collection = Collection("test_collection")
pipeline = Pipeline("test_pipeline")
results = (
    await collection.query()
    .vector_recall("Here is some query", pipeline)
    .limit(10)
    .filter({
        "metadata": {
            "$or": [
                {
                    "$and": [
                        {
                            "$eq": {
                                "uuid": 1
                            }    
                        },
                        {
                            "$lt": {
                                "index": 100 
                            }
                        }
                    ] 
                },
                {
                   "special": {
                        "$ne": True
                    } 
                }
            ]    
        }
    })
    .fetch_all()
)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
collection = pgml.newCollection("test_collection")
pipeline = pgml.newPipeline("test_pipeline")
results = await collection.query()
    .vector_recall("Here is some query", pipeline)
    .limit(10)
    .filter({
        "metadata": {
            "$or": [
                {
                    "$and": [
                        {
                            "$eq": {
                                "uuid": 1
                            }    
                        },
                        {
                            "$lt": {
                                "index": 100 
                            }
                        }
                    ] 
                },
                {
                   "special": {
                        "$ne": True
                    } 
                }
            ]    
        }
    })
    .fetch_all()
```
{% endtab %}
{% endtabs %}

The above query would filter out all documents that do not have a key `special` with a value `True` or (have a key `uuid` equal to 1 and a key `index` less than 100).

### **Full Text Filtering**

If full text search is enabled for the associated Pipeline, documents can be first filtered by full text search and then recalled by embedding similarity.

{% tabs %}
{% tab title="Python" %}
```python
collection = Collection("test_collection")
pipeline = Pipeline("test_pipeline")
results = (
    await collection.query()
    .vector_recall("Here is some query", pipeline)
    .limit(10)
    .filter({
        "full_text": {
            "configuration": "english",
            "text": "Match Me"
        }
    })
    .fetch_all()
)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
collection = pgml.newCollection("test_collection")
pipeline = pgml.newPipeline("test_pipeline")
results = await collection.query()
    .vector_recall("Here is some query", pipeline)
    .limit(10)
    .filter({
        "full_text": {
            "configuration": "english",
            "text": "Match Me"
        }
    })
    .fetch_all()
```
{% endtab %}
{% endtabs %}

The above query would first filter out all documents that do not match the full text search criteria, and then perform vector recall on the remaining documents.
