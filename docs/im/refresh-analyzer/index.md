---
layout: default
title: Refresh Search Analyzer
nav_order: 40
parent: Index management
has_children: false
redirect_from: /docs/ism/refresh-analyzer/
has_toc: false
---

# Refresh search analyzer

With ISM installed, you can refresh search analyzers in real time with the following API:

```json
POST /_opensearch/_refresh_search_analyzers/<index or alias or wildcard>
```
For example, if you change the synonym list in your analyzer, the change takes effect without you needing to close and reopen the index.

To work, the token filter must have an `updateable` flag of `true`:

```json
{
  "analyzer": {
    "my_synonyms": {
      "tokenizer": "whitespace",
      "filter": [
        "synonym"
      ]
    }
  },
  "filter": {
    "synonym": {
      "type": "synonym_graph",
      "synonyms_path": "synonyms.txt",
      "updateable": true
    }
  }
}
```
