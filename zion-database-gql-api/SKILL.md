---
description: Zion.app database operations via auto-generated GraphQL schema. Use when: (1) Querying or mutating database records, (2) Working with relationships (1:1, 1:N, N:N), (3) Filtering, sorting, paginating data, (4) Understanding how tables map to GraphQL types, (5) Using aggregate functions, (6) Handling media assets (images/files)
alwaysApply: false
---

# Data Model Overview

The GraphQL schema is automatically generated directly from the user's data model. To illustrate the general rules for generating this schema, the following explanations will primarily use the post table from the example metadata as a reference point.

Here is the metadata for a simple blogging application. We will use this `post` table and its related tables as the primary example throughout this guide.

- **Columns:**

  - id: bigint (primary key)
  - created_at: timestamptz
  - title: text
  - content: text
  - author_account: bigint (foreign key to account.id, reference to the author of the post)
- **Constraints: **

  - post_id_key: unique constraint on (id)
  - post_pkey: primary key on (id)
- **Relationships (from the post's perspective):**

  - author (One-to-Many from account to post): The author_account column links each post to one account record
  - post_tags (One-to-Many from post to post_tag): A post can be associated with multiple tag records through the post_tag join table.
  - meta (One-to-One from post to post_meta): Each post has a corresponding meta record (post_meta) containing additional information.

**TableMetadata:**

```json
[
  {
    "name": "post",
    "columnMetadata": [
      {"name": "id", "type": "BIGINT"},
      {"name": "created_at","type": "TIMESTAMPTZ"},
      {"name": "updated_at","type": "TIMESTAMPTZ"},
      {"name": "title","type": "TEXT"},
      {"name": "content","type": "TEXT"},
      {"name": "author_account","type": "BIGINT", "displayName": "Author ID"},
      {"name": "cover_image", "type": "IMAGE"}
    ],
    "constraintMetadata": [
      {"name": "post_id_key","compositeUniqueColumns": ["id"]},
      {"name": "post_pkey","primaryKeyColumns": ["id"]}
    ]
  },
  {
    "name": "tag",
    "displayName": "Tags on Posts",
    "columnMetadata": [
      {"name": "id","type": "BIGINT"},
      {"name": "created_at","type": "TIMESTAMPTZ"},
      {"name": "updated_at","type": "TIMESTAMPTZ"},
      {"name": "name","type": "TEXT"}
    ],
    "constraintMetadata": [
      {"name": "tag_id_key","compositeUniqueColumns": ["id"]},
      {"name": "tag_pkey","primaryKeyColumns": ["id"]}
    ]
  },
  {
    "name": "post_tag",
    "columnMetadata": [
      {"name": "id","type": "BIGINT"},
      {"name": "created_at","type": "TIMESTAMPTZ"},
      {"name": "updated_at","type": "TIMESTAMPTZ"},
      {"name": "post_post","type": "BIGINT"},
      {"name": "tag_tag","type": "BIGINT"}
    ],
    "constraintMetadata": [
      {"name": "post_tag_id_key","compositeUniqueColumns": ["id"]},
      {"name": "post_tag_pkey","primaryKeyColumns": ["id"]}
    ]
  },
  {
    "name": "post_meta",
    "displayName": "Post Metadata",
    "columnMetadata": [
      {"name": "id","type": "BIGINT"},
      {"name": "created_at","type": "TIMESTAMPTZ"},
      {"name": "updated_at","type": "TIMESTAMPTZ"},
      {"name": "seo_title","type": "TEXT"},
      {"name": "word_count","type": "BIGINT"},
      {"name": "post_post","type": "BIGINT"}
    ],
    "constraintMetadata": [
      {"name": "post_meta_id_key","compositeUniqueColumns": ["id"]},
      {"name": "post_meta_post_post_key","compositeUniqueColumns": ["post_post"]},
      {"name": "post_meta_pkey","primaryKeyColumns": ["id"]}
    ]
  },
  {
      "name": "account",
      "columnMetadata": [
        {"name": "id", "type": "BIGINT"},
        {"name": "name", "type": "TEXT"}
      ],
      "constraintMetadata": [
        {"name": "account_pkey", "primaryKeyColumns": ["id"]},
        {"name": "account_id_key","compositeUniqueColumns": ["id"]}
      ]
  }
]
```

**RelationMetadata:**

```
[
  {
    "targetTable": "post_tag",
    "type": "ONE_TO_MANY",
    "sourceTable": "post",
    "sourceColumn": "id",
    "nameInSource": "post_tags",
    "nameInTarget": "post",
    "targetColumn": "post_post"
  },
  {
    "targetTable": "post_meta",
    "type": "ONE_TO_ONE",
    "sourceTable": "post",
    "sourceColumn": "id",
    "nameInSource": "meta",
    "nameInTarget": "post",
    "targetColumn": "post_post"
  },
  {
    "targetTable": "post_tag",
    "type": "ONE_TO_MANY",
    "sourceTable": "tag",
    "sourceColumn": "id",
    "nameInSource": "post_tags",
    "nameInTarget": "tag",
    "targetColumn": "tag_tag"
  },
  {
    "targetTable": "post",
    "type": "ONE_TO_MANY",
    "sourceTable": "account",
    "sourceColumn": "id",
    "nameInSource": "posts",
    "nameInTarget": "author",
    "targetColumn": "author_account"
  }
]
```

## Detailed Reference

For comprehensive details on GraphQL schema generation, operations, and advanced features, see [detailed-reference.md](references/detailed-reference.md). This includes:

- **Tables**: Root operations (query, mutation, subscription)
- **Columns**: Primitive types, composite types (media assets), system-managed columns
- **Relationships**: One-to-One (1:1), One-to-Many (1:N), Many-to-Many (N:N) patterns
- **Constraints**: Primary keys, unique constraints, foreign keys
- **Query Operations**: Filtering, sorting, pagination, deduplication, aggregation
- **Mutation Operations**: Insert, update, delete (bulk and single-record)
- **Advanced Features**: Subscriptions, nested operations, relationship traversal

Read `references/detailed-reference.md` when you need:
- Detailed GraphQL operation syntax
- Complex filtering and sorting patterns
- Relationship query examples
- Mutation input structures
- Aggregate function usage

