# vectors

A vector store for [Folio](https://github.com/samyyy2423/folio): vectors (embeddings) kept with an id, their
text and what you say of them, searched for the ones most like a query, and saved to a file and loaded again.
The store a RAG system keeps its chunks in, or anything found by meaning. In Folio alone, so it works
everywhere Folio does.

```text
folio add vectors
```

```
use the package "vectors" as vectors

set notes to vectors's store()
vectors's insert(notes, (list of 0.1, 0.8, 0.3), "bees fly by day", dictionary where "page" is 2)
vectors's put(notes, "intro", (list of 0.9, 0.1, 0.0), "folios fly at night", nothing)

for each found in vectors's search(notes, (list of 0.2, 0.7, 0.3), 5)
    say found's score " " found's text " " found's meta
end

vectors's save(notes, "notes.vectors")
set again to vectors's load("notes.vectors")
```

The vectors usually come from a model: `ai.folio`'s `embed` gives an array with a row for each text, which
`insert_many` takes as it is.

## What it has

| | |
|---|---|
| `store()` | an empty store, ranked by cosine similarity |
| `store_by(metric)` | ranked by `"cosine"`, `"dot"` (the dot product) or `"l2"` (the nearest by distance; each found has its `distance`, and its `score` is the distance taken from 0) |
| `insert(store, vector, text, meta)` | a vector (a list of numbers, or an array) with its text and meta (a dictionary, or `nothing`), under the next number: its id |
| `insert_many(store, vectors, texts, metas)` | many at once (an array with a row each, or a list of vectors); texts and metas are lists or `nothing`: their ids |
| `put(store, id, vector, text, meta)` | under an id of your own (a number or text), new or in place of the one with that id |
| `delete(store, id)` | taken out: whether it was there |
| `get(store, id)` | a record of `id`, `text`, `meta` and `vector`, or `nothing` |
| `size(store)`, `ids(store)` | how many, and their ids in the order they came |
| `search(store, query, k)` | the k most like the query, the best first: records of `id`, `score`, `text` and `meta` |
| `search_where(store, query, k, filter)` | of only those the filter lets through: a dictionary of what their meta must have, or a function given each found record that says yes or no |
| `search_many(store, queries, k)` | many queries at once, all scored by one matrix multiplication: a list of answers |
| `save(store, file)`, `load(file)` | to a file and back (`--allow-write`, `--allow-read`) |

Every vector in a store has the same length, set by the first. For cosine, vectors are kept scaled to a length
of 1, so `get` gives them that way. Equal scores come back in the order the vectors came.

## How fast

Each search multiplies every vector by the query at once (`arrays.folio`'s matmul, in machine code), and keeps
the best k as it goes through the scores. With 20,000 vectors of 384 numbers (as all-MiniLM-L6-v2 makes):

| | |
|---|---|
| insert all 20,000 | 0.27 s |
| a search | 10 ms (the first after a change, 0.1 s, makes the matrix again) |
| save, load | 0.29 s, 0.25 s; the file is 31 MB |

and the best it finds is the one a plain loop over every vector finds. A file keeps each number as a 32-bit
float, as embeddings are made, so scores after `load` are the same to about 7 digits.

It searches every vector, so it finds the best ones exactly; past a few hundred thousand vectors, memory is the
limit.

## Tests

```text
folio test --allow-read --allow-write vectors.test.folio
```

## License

Apache License 2.0, as Folio is: see [LICENSE](LICENSE).
