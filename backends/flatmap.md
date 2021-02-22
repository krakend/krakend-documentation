---
lastmod: 2020-07-24
date: 2019-04-05
toc: true
linktitle: Array manipulations
title: Array manipulation - flatmap
weight: 70
since: 0.9
source: https://github.com/devopsfaith/flatmap
menu:
  documentation:
    parent: backends
---

The flatmap middleware allows you to **manipulate collections** (or arrays, or lists, you name it). While the [basic manipulation operations](/docs/backends/data-manipulation) allow you to work directly with objects, the collections require a different approach: the **flatmap component**.

When working with lists, KrakenD needs to flatten and expand array structures to objects to operate with them, and vice versa. This process is automatically done by the flatmap component, letting you concentrate only on the type of operation you want to execute.

## When to manipulate arrays
You can manipulate collections at two different stages:

- When the response of a backend is received (inside its `backend` section)
- After having merged all the backend responses (inside the `endpoint` section, starting at KrakenD 1.2)

You can do simultaneous combinations to output the desired result. For instance, declare an endpoint with three backends that apply transformations independently and a final change within the endpoint after merging the three.

## Types of manipulations

There are different types of operations you can do:

*   **Moving**, **embedding** or **extracting** items from one place to another (equivalent concepts to [`mapping`](/docs/backends/data-manipulation/#mapping) and [`allow`](/docs/backends/data-manipulation/#allow))
*   **Deleting** specific items (similar concept to [`deny`](/docs/backends/data-manipulation/#deny))
*   **Appending** items from one list to the other

{{< note title="Important" >}}
Use a basic [data manipulation](/docs/backends/data-manipulation/) operation such as [`target`](/docs/backends/data-manipulation/#target), [`deny`](/docs/backends/data-manipulation/#deny) or [`allow`](/docs/backends/data-manipulation/#allow) whenever it fits as their computational cost is lower.

The flatmap component is **not as a general solution for all objects**, and makes sense only when you need to manipulate collections.
{{< /note >}}

## Flatmap configuration
Depending on the stage you want to do the manipulation, you will need an `extra_config` configuration inside your `endpoint` or `backend` section. For both cases, the namespace is `github.com/devopsfaith/krakend/proxy`.

The component structure with three operations would be as follows:

        "extra_config": {
            "github.com/devopsfaith/krakend/proxy": {
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
- `flatmap_filter` (*list*) The list of operations to **execute sequentially** (top down). Every operation is defined with an object containing two properties:
  - `type` (*string*) One of the recognized operation types
  - `args` (*list*) The arguments passed to the operation.

### Operations and arguments

The types of operations are defined as follows:

*   **Move**: To move or rename a collection to another. It needs two arguments.
    *   `"type": "move"`
    *   `"args": ["target_in_collection", "destination_in_collection"]`
*   **Delete**: To remove all matching patterns within a collection. Needs one or more arguments.
    *   `"type": "del"`
    *   `"args": ["target_in_collection_to_delete", "another_collection_to_delete", "..."]`
*   **Append**: To append a collection after another one, and return only the latter. Needs 2 arguments.
    *   `"type": "append"`
    *   `"args": ["collection_to_append", "returned_collection"]`


The format of the arguments (`args`) to proceed with the operation is very simple. In short, object nesting is represented with **dots**, while the index of an array is represented with a **number**. Or all matching items with **wildcards**. So:

*   The dot operator `.` indicates a new array nesting level
*   The wildcard `*` matches any key (property name, collection key name, or index)
*   A `number` identifies the Nth-1 member of a collection, being `0` its first item.

Operations always apply to ** the last item** in the arguments. For instance, a deletion of `a.b.c` deletes `c` but leaves `a.b` in the response.

### Notation by example

We are going to use an elementary JSON structure as an example of data representation. See below:

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

#### Observations

Notice from this example that...

*   `a` and `b1` contain arrays (`[...]`) with objects inside.
*   `b2`, `c` and `d` are not arrays
*   Since `a` is an array (`"a": []`) we need to use the flatmap component. If it were an object (`"a": {}`) we would use [deny or allow](/docs/backends/data-manipulation/)

#### Representing some values

Now that we are familiar with the structure let's represent same values:

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
| `"a.*.b1.*.c"` | `"a.*.b1.*.d"`         | <i class="fa fa-check ok"></i>        | Rename `c` to `d`                                                      |
| `"a.*.b1.*.c"` | `"a.*.c"`              | <i class="fa fa-times-circle ko"></i> | Missing level                                                          |
| `"a.b1.c"`     | `"c"`                  | <i class="fa fa-times-circle ko"></i> | Missing array after `a`                                                |
| `"a.0.b1.0.c"` | `"c"`                  | <i class="fa fa-check ok"></i>        | Extract only `c` from the first and first items                        |
| `"a.*.b1.c"`   | `"c"`                  | <i class="fa fa-times-circle ko"></i> | Incorrect target, `b1` has an array surrounding `c`                    |
| `"a.*.b1.c"`   | `"a.*.b1.*.d.*.e"`     | <i class="fa fa-times-circle ko"></i> | Incorrect target, `b1` has an array surrounding `c`                    |
| `"a.*.b1.*.c"` | `"a.*.b1.*.c.d.e.f.g"` | <i class="fa fa-check ok"></i>        | Add additional levels                                                  |
| `"a.*.b1.*.c"` | `"a.*.x.*.c"`          | <i class="fa fa-times-circle ko"></i> | Incorrect, renaming to an element `x` that is not in the last position |
| `"a.*.b1.*.c"` | `"a.*.x.*.c.d.e.f.g"`  | <i class="fa fa-times-circle ko"></i> | Incorrect, renaming to an element `x` that is not in the last position |
| `"a.*.b1.*.c"` | `"a.*.b1.*.d.*.e"`     | <i class="fa fa-times-circle ko"></i> | Incorrect, destination path has more wildcards than source path        |

## Configuration example

The following example demonstrates how to modify a collection doing these operations:

        "extra_config": {
            "github.com/devopsfaith/krakend/proxy": {
                "flatmap_filter": [
                    {
                        "type": "append",
                        "args": ["kindergarten", "schools"]
                    },
                    {
                        "type": "move",
                        "args": ["schools.42.students", "alumni"]
                    },
                    {
                        "type": "del",
                        "args": ["schools"]
                    },
                    {
                        "type": "del",
                        "args": ["alumni.*.password"]
                    },
                    {
                        "type": "move",
                        "args": ["alumni.*.PK_ID", "alumni.*.id"]
                    }
                ]
            }
        }

**What did we do here?**

There is a sequence of 4 operations to:

*   Extract all items inside `kindergarten` and append them to the `students` collection.
*   Extract all `students` of the `43rd` school (array starts at 0) and put them under a new property `alumni`
*   Get rid of all the remaining schools
*   Delete all items with a property `password` inside the array
*   Rename all items with a property `PK_ID` to `id`

For more examples, [see the test file](https://github.com/devopsfaith/flatmap/blob/master/tree/tree_example_test.go).


## Mixing flatmap with other manipulation operations

When the flatmap filter is enabled, the operations `group` and `target` keep their functionality, but `allow`, `deny`, and `mapping` are ignored.
