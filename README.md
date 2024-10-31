# Example: Serverless Search Portal with Transfer Integration

This repository is an example of the [@globus/template-search-portal](https://github.com/globus/template-search-portal)

You can create your own portal with similar functionality by following the [**Creating Your Own Research Search Portal**](https://github.com/globus/template-search-portal?tab=readme-ov-file#creating-your-own-static-research-search-portal) section in the template repository and then referencing the sections below.

## Background

Globus Search is often used to provide metadata to datasets available via Globus Transfer. To support this use case, our serverless search portal includes the ability to enable Transfer-related features for users browsing your index.

## Pre-Requisites

- A [Globus Search Index](https://docs.globus.org/api/search/) you administer.
- A Globus Collection that hosts data relevant to the documents in your Globus Search Index.
- A Search Portal, created following: [**Creating Your Own Research Search Portal**](https://github.com/globus/template-search-portal?tab=readme-ov-file#creating-your-own-static-research-search-portal) in our template repository, **with Authentication enabled.**


## Globus Transfer Integration

The simplest way to enable Transfer functionality in your Search Portal is to [add authentication to your portal](https://github.com/globus/template-search-portal?tab=readme-ov-file#private-globus-search-indexes-authentication)[^1] and include Transfer properties in your search entries.

[^1]: Even if your search data is public, authentication is required to use Globus Transfer.

### Adding Transfer Properties

When a `globus.transfer` object is present on your `GMetaResult.entries.content[n]`, and you've configured authentication, Transfer functionality will be automatically enabled[^2].

[^2]: You can opt-out of the Transfer functionality by setting `data.attributes.features.transfer` to `false` in your `static.json` file.


```jsonc
"globus": {
  "transfer": {
    /**
     * The UUID of the Globus Collection where the data resides.
     * @type string
     */
    "collection": "a6f165fa-aee2-4fe5-95f3-97429c28bf82",
    /**
     * The path to the data the entry represents on the Collection.
     * @type string
     */
    "path": "/portal/catalog/dataset_atl",
    /**
     * The type of inode the path represents.
     * @default "file" If no value is provided, the path is assumed to be a file.
     * @type "directory" | "file"
     */
    "type": "directory"
  }
}
```

<details>

  <summary>View a more comprehensive example of a <code>GMetaEntry</code></summary>

```json
  {
    "subject": "5352507d-1293-4827-8ba9-c2d4a0eb78d1",
    "visible_to": [
      "public"
    ],
    "content": {
      "name": "Atlanta International Airport Climate Data (ATL)",
      "id": "5352507d-1293-4827-8ba9-c2d4a0eb78d1",
      "path": "/portal/catalog/dataset_atl",
      "region": "south",
      "tags": [
        "airport"
      ],
      "globus": {
        "transfer": {
          "collection": "a6f165fa-aee2-4fe5-95f3-97429c28bf82",
          "path": "/portal/catalog/dataset_atl",
          "type": "directory"
        }
      }
    }
  }
```

</details>

### Advanced Integrations

#### Using Dynamic Transfer Property References

**🧪 This feature is experimental, feel free to provide feedback if you encounter issues.**

As an alternative to adding the `globus.transfer` property to entries [JSONata](https://jsonata.org/) can be used to allow dynamic references to existing properties.

1. Enable JSONata support in your portal by setting `data.attributes.features.jsonata` to `true`

```jsonc
  "data": {
    "attributes": {
      "features": {
        "jsonata": true
      }
      // ...
    }
  }
```

2. You'll customize the `Result` component to include the `globus.transfer` configuration.

  - A `property` member can be used to signal JSONata should be used (against the `GMetaResult`) for sourcing.
  - For static values, a `string` can be used. 

```jsonc
  "data": {
    "attributes": {
      "components": {
        "Result": {
          "globus": {
            "transfer": {
              // The `collection` property will be sourced from the `subject` on the `GMetaResult`.
              "collection": {
                "property": "$split($split(subject, 'globus://')[1], '/')[0]"
              },
              // The `path` will be sourced from the `subject`, similar to the `collection`.
              "path": {
                "property": "$replace(subject, 'globus://' & $split($split(subject, 'globus://')[1], '/')[0], '')"
              },
              // Setting the `type` to a static value.
              "type": "directory"
            }
          }
        }
      }
      // ...
    }
  }
```






