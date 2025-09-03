---
lastmod: 2023-02-07
old_version: true
date: 2019-04-05
toc: true
linktitle: Response manipulation on arrays
title: Array manipulation
description: Integrate the FlatMap backend into KrakenD API Gateway for array data transformation and mapping operations in your API ecosystem
weight: 110
menu:
  community_v2.6:
    parent: "060 Request and Response Manipulation"
meta:
  noop_incompatible: true
  since: v0.9
  source: https://github.com/krakend/flatmap
  namespace:
  - proxy
  scope:
  - backend
  - endpoint
---

The flatmap middleware allows you to **manipulate collections** (or arrays, or lists; you name it) or to **flatten objects** from the response.

While the [basic manipulation operations](/docs/v2.6/backends/data-manipulation/) allow you to work directly with objects, the collections require you to use this **flatmap component**. The flatmap also will enable you to **extract or move nested objects** to have a customized object structure.

{{< note title="Looking for a Query Language manipulation?" type="info" >}}
If you are an Enterprise user, you might want to use [Response manipulation with query language ](/docs/enterprise/endpoints/jmespath/) instead
{{< /note >}}

When working with lists, KrakenD needs to flatten and expand array structures to objects to operate with them and vice versa. This process is automatically done by the flatmap component, letting you concentrate only on the type of operation you want to execute.

## When to manipulate arrays
You can manipulate collections at two different stages:

- When the response of a backend is received (inside its `backend` section)
- After merging all the backend responses (inside the `endpoint` section)

You can do simultaneous combinations to output the desired result. For instance, declare an endpoint with three backends that apply transformations independently and a final change within the endpoint after merging the three.

## When to manipulate objects with flatmap
The flatmap can be used on objects when the [basic data manipulation](/docs/v2.6/backends/data-manipulation/) options fall short. For instance, when you need to extract and rename nested objects to the root, append, and other more sophisticated operations.

## Types of manipulations
There are different types of operations you can do:

- `move`: To **move**, **rename**, **embed** or **extract** items from one place to another (equivalent concepts to  and [`allow`](/docs/v2.6/backends/data-manipulation/#allow))
- `del`: To **delete** specific items
- `append`: To **append** items from one list to the other

{{< note title="Basic manipulation is faster" type="warning">}}
Prefer basic [data manipulation](/docs/v2.6/backends/data-manipulation/) operations such as [`mapping`](/docs/v2.6/backends/data-manipulation/#mapping), [`target`](/docs/v2.6/backends/data-manipulation/#target), [`deny`](/docs/v2.6/backends/data-manipulation/#deny) or [`allow`](/docs/v2.6/backends/data-manipulation/#allow) over flatmap whenever you work with objects as their computational cost is lower. Reserve the flatmap component for collections or operations that basic manipulation can't do (such as flattening objects)
{{< /note >}}

## Flatmap configuration
Depending on the stage you want to do the manipulation, you will need an `extra_config` configuration inside your `endpoint` or `backend` section. For both cases, the namespace is `proxy`.

{{< note title="Flatmap at the endpoint level requires +1 backend" type="info" >}}
The flatmap does not load at the `endpoint` level unless there is more than one backend configured, as its purpose is to manipulate responses after the merge operation. Therefore, use it in the `backend` if you only have one.
{{< /note >}}


The component structure with three operations would be as follows:
```json
{
    "extra_config": {
        "proxy": {
            "flatmap_filter": [
                {
                    "type": "move",
                    "args": ["target_in_collection", "destination_in_collection"]
                },
                {
                    "type": "del",
                    "args": ["target_in_collection"]
                },
                {
                    "type": "append",
                    "args": ["collection_to_append", "returned_collection"]
                }
            ]
        }
    }
}
```
Then the `flatmap_filter` is an **array** with the **list of operations to execute sequentially** (top-down). Each flatmap step takes the output of its previous execution, and every operation is defined with an object containing two properties:

{{< schema version="v2.6" data="proxy/flatmap.json" property="items" >}}

### Operations and arguments

The types of operations are defined as follows:

- **Move**: To move or rename a collection to another. It needs two arguments.
    - `"type": "move"`
    - `"args": ["target_in_collection", "destination_in_collection"]`
- **Delete**: To remove all matching patterns within a collection. It needs one or more arguments.
    - `"type": "del"`
    - `"args": ["target_in_collection_to_delete", "another_collection_to_delete", "..."]`
- **Append**: To append a collection after another and return only the latter. It needs two arguments.
    - `"type": "append"`
    - `"args": ["collection_to_append", "returned_collection"]`


The format of the arguments (`args`) to proceed with the operation is very simple. In short, **object nesting** is represented with **dots**, while the index of an array is represented with a **number**. Or all matching items with **wildcards**. So:

*   The dot operator `.` indicates a new array nesting level
*   The wildcard `*` matches any key (property name, collection key name, or index)
*   A `number` identifies the Nth-1 member of a collection, being `0` its first item.

Operations always apply to ** the last item** in the arguments. So, for instance, the deletion of `a.b.c` deletes `c` but leaves `a.b` in the response.

### Notation by example

We will use an elementary JSON structure as an example of data representation. See below:

```json
{
    "a": [
        {
            "b1": [
                {
                    "c": 1,
                    "d": "foo"
                },
                {
                    "c": 2,
                    "d": "bar"
                }
            ],
            "b2": true
        },
        {
            "b1": [
                {
                    "c": 3,
                    "d": "vaz"
                }
            ]
        }
    ]
}
```

#### Observations

Notice from this example that...

- `a` and `b1` contain arrays (`[...]`) with objects inside.
- `b2`, `c` and `d` are not arrays
- Since `a` is an array (`"a": []`), we need to use the flatmap component. If it were an object (`"a": {}`), we would use [deny or allow](/docs/v2.6/backends/data-manipulation/)

#### Representing some values

Now that we are familiar with the structure let's represent some values:

| Notation     | Value                                                                                                               |
| ------------ | ------------------------------------------------------------------------------------------------------------------- |
| `a`          | The content of _a_: `[{"b1": [{"c": 1,"d": "foo"},{"c": 2,"d": "bar"}],"b2": true}, {"b1": [{"c": 3,"d": "vaz"}]}]` |
| `a.1`        | Second object of _a_ key: `{"b1": [ { "c": 3, "d": "vaz" } ]}` (first objects starts at 0)                          |
| `a.0.b1.0.d` | `foo`                                                                                                               |
| `a.1.b1.0.d` | `vaz`                                                                                                               |
| `a.*.b1.*.d` | 3 matches of `d` in this path: `foo`, `bar`, `vaz`                                                                  |
| `a.*.*.*.d`  | 3 matches of `d` in this path: `foo`, `bar`, `vaz`                                                                  |

#### Practical examples regarding operations

Some individual operations **on the example structure above**:

| Target         | Destination            | Correct?                              | Comments                                                               |
| -------------- | ---------------------- | ------------------------------------- | ---------------------------------------------------------------------- |
| `"a.*.b1.*.c"` | `"a.*.b1.*.d"`         | Yes        | Rename `c` to `d`                                                      |
| `"a.*.b1.*.c"` | `"a.*.c"`              | No         | Missing level                                                          |
| `"a.b1.c"`     | `"c"`                  | No         | Missing array after `a`                                                |
| `"a.0.b1.0.c"` | `"c"`                  | Yes        | Extract only `c` from the first and first items                        |
| `"a.*.b1.c"`   | `"c"`                  | No         | Incorrect target, `b1` has an array surrounding `c`                    |
| `"a.*.b1.c"`   | `"a.*.b1.*.d.*.e"`     | No         | Incorrect target, `b1` has an array surrounding `c`                    |
| `"a.*.b1.*.c"` | `"a.*.b1.*.c.d.e.f.g"` | Yes        | Add additional levels                                                  |
| `"a.*.b1.*.c"` | `"a.*.x.*.c"`          | No         | Incorrect, renaming to an element `x` that is not in the last position |
| `"a.*.b1.*.c"` | `"a.*.x.*.c.d.e.f.g"`  | No         | Incorrect, renaming to an element `x` that is not in the last position |
| `"a.*.b1.*.c"` | `"a.*.b1.*.d.*.e"`     | No         | Incorrect, destination path has more wildcards than source path        |

## Configuration examples

The following examples demonstrate how to modify a collection or objects using flatmap.

### Example: Extract objects to another level
We have a backend that provides the following response as Input for flatmap:

**Input**:
```json
{
    "shipping_id": "f15f8c62-8c63-46de-a7f6-a08f131848c5",
    "zone": {
        "state": "NY",
        "zip": "10001"
    }
}
```
And we want to **extract fields** from `zone`, rename them, and **place them in the root**, like this:

**Output**:

```json
{
    "shipping_id": "f15f8c62-8c63-46de-a7f6-a08f131848c5",
    "shipping_state": "NY",
    "shipping_zip": "10001"
}
```

**Configuration**:

```json
{
  "backend": [{
      "url_pattern": "/shipping",
      "extra_config": {
          "proxy": {
              "flatmap_filter": [
                  { "type": "move", "args": ["zone.state","shipping_state"] },
                  { "type": "move", "args": ["zone.zip","shipping_zip"] },
                  { "type": "del","args": ["zone"] }
              ]
          }
      }
  }]
}
```

As you can see, we did three operations:

1. Move the state to the root with a new name
2. Move the zip to the root with a new name
3. Delete `zone` as it became a null object after emptying it.

### Example: Moving around data in arrays
In this example we have a couple of arrays that we want to manipulate.

**Input**:
```json
{
    "kindergarten": [
        { "name": "TEST Kinder" },
        { "name": "Lil' Elephants" },
        { "name": "Bright Rainbows" }
    ],
    "schools": [
        { "title": "Brookside Elementary" },
        { "title": "Oak Tree School" }
    ]
}
```
And we want an output where both arrays are merged, using consistent naming. And we also want to get rid of the first TEST element. As follows:

**Output**:

```json
{
    "schools":[
        {"name":"Lil' Elephants"},
        {"name":"Bright Rainbows"},
        {"name":"Brookside Elementary"},
        {"name":"Oak Tree School"}
    ]
}
```
Then we need the following **configuration**:

```json
{
    "endpoint": "/education-clean",
    "backend": [{
        "url_pattern": "/education",
        "extra_config": {
            "proxy": {
                "flatmap_filter": [
                    {
                        "type": "del",
                        "args": ["kindergarten.0"]
                    },
                    {
                        "type": "move",
                        "args": ["schools.*.title", "schools.*.name"]
                    },
                    {
                        "type": "append",
                        "args": ["kindergarten", "schools"]
                    }
                ]
            }
        }
    }]
}
```

**What did we do here?**

There is a sequence of three operations:

- Delete the first element (index `0`) of `kindergarten`.
- Rename all `title` attributes of `schools` to `name`.
- Append all the `kindergarten` content to `schools`.

For more examples, [see this test file](https://github.com/krakend/flatmap/blob/master/tree/tree_example_test.go).


## Mixing flatmap with other manipulation operations

When the flatmap filter is enabled, the operations `group` and `target` keep their functionality, but `allow`, `deny`, and `mapping` are ignored.
