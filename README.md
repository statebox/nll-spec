# NLL

A compact encoding of list of list of numbers, using (signed) varints.

### Specification

It is just an unsigned varint followed by a list of varints (possibly signed).

| HEAD | ITEM | ITEM | ... | ITEM |
|------|------|------|-----|------|
| h    | X1   | X2   | ... |X**n**|

First value in the stream is the header, an unsigned varint **h** and encodes two properties:

- **signed**: Whether the following numbers use signed or unsigned encoding,
- **n**: Total number of items in the rest of the stream.

###### Header

The header value *h* is either **even** or **odd**.

|            | Even  | Odd         |
|------------|-------|-------------|
| **signed** | no    | yes         |
| **n**      | h / 2 | (h - 1) / 2 |

###### Delimiter

The number zero is used as a delimiter, indicating the start of a new
list.

In order to still be able to encode a list containing zeros, we use a
trivial mapping to the integers excluding zero.

| x    | -3 | -2 | -1 | 0 | 1 | 2 | 3 |        |
|------|----|----|----|---|---|---|---|--------|
| F(x) | -3 | -2 | -1 | 1 | 2 | 3 | 4 | encode |
| G(x) | -3 | -2 | -1 | 0 | 0 | 1 | 2 | decode |

F leaves negative numbers untouched and shifts the positive numbers one
up, and G does the inverse, so for all x, G(F(x)) = x.

```
F(x) := (x <  0) ? x : x + 1
G(x) := (x <= 0) ? x : x - 1
```

## Example

As an example, encoding

```js
[
  [],
  [1,2],
  [3],
  [0]
]
```

gives us

| HEAD | ITEM | ITEM | ITEM | ITEM | ITEM | ITEM | ITEM | ITEM |
|------|------|------|------|------|------|------|------|------|
| 16   | 0    | 0    | 2    | 3    | 0    | 4    | 0    | 1    |

Another example, this time with signed integers

```js
[
  [-1,0,1]
]
```

gives us

| HEAD | ITEM | ITEM | ITEM | ITEM |
|------|------|------|------|------|
| 9    | 0    | -1   | 1    | 2    |

some other examples:

```
[]      00
[[]]    02 00
[[0]]   04 00 01
[[1]]   04 00 02
[[-1]]  05 00 01
```

