# Serialization

A compact binary serialization format. The goal is zero extra classes — existing structs serialize directly without wrapper types.
  
## Flat Structs

Fields are written sequentially. Variable-length fields (e.g. strings) are preceded by their size.

```
struct {
    int    id;
    string name;
}
```

```
bytes = [id, name.size, name.data...]
```

## Graph / Cyclic Structures

Nodes with references serialize by storing offsets (addresses) into the byte array instead of embedding child data inline. This handles cycles naturally.

```
struct Node {
    int    id;
    Node[] nodes;
}
```

**Example graph:**
```
1 → 2
2 → 3
3 → 4, 2
4 → 1
```

**Serialized layout:**
```
bytes = [
    node1.id, node1.nodes.size, node2.offset,
    node2.id, node2.nodes.size, node3.offset,
    node3.id, node3.nodes.size, node4.offset, node2.offset,
    node4.id, node4.nodes.size, node1.offset
]
```

Each reference is an offset (index) into the byte array pointing to the target node.

## Inheritance

When a field type is a parent class (e.g. `Media`), a `classId` is prepended to identify the concrete subtype at deserialization time:
```
struct Media { }
struct Image : Media { string path;}
struct Video : Media { string path;}
struct Document : Media { string path;}

struct Area {
// here is ref or not but anyway class id should present
    Media media;
    int x;
    int y;
    int width;
    int height;
}
```

```
bytes = [media.classid, media.path.size, media.path.data..., x, y, width, height]
```

## Other

Custom serialization or a plugin can be used to support edge cases not covered by the default format.

## Meta lang
for each base type:
- int -> i4
- float -> f4
- pointer -> p< T >
- object -> T
- array -> a< T >

struct node looks like: 

```
node {
    i4 id;
    a<node> nodes;
}
```