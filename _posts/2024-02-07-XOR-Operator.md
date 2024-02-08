---
title: XOR Operator
date: 2024-02-05 11:11:00 -0700
categories: [algorithm]
tags: 
Pin:
---

what is the XOR in python?

In Python, **XOR is a bitwise operator** that is also known as **Exclusive OR**.

The symbol for XOR in Python is '^' and in mathematics, its symbol is '⊕'.

**Syntax**

```python
xor_num = num1 ^ num2
```

The properties of XOR operation are as follows:

- **Reflexivity**: A number XORed with itself results in 0. That is, $ x \oplus x = 0 $.
- **Identity**: Any number XORed with 0 results in the number itself. That is, $ x \oplus 0 = x $.
- **Commutativity**: The order of operands does not affect the result of the XOR operation. That is,$ x \oplus y = y \oplus x $.
- **Associativity**: The XOR operation can be grouped in any order without affecting the result. That is, $ x \oplus (y \oplus z) = (x \oplus y) \oplus z $.

| A (Input) | B (Input) | $ A \oplus B $ (Output) |
| --------- | --------- | -------------------------- |
| 0         | 0         | 0                          |
| 0         | 1         | 1                          |
| 1         | 0         | 1                          |
| 1         | 1         | 0                          |

**Truth table for XOR (boolean)**

| A     | B     | $ A \oplus B $ |
| :---- | :---- | :---------------- |
| False | False | False             |
| True  | False | True              |
| False | True  | True              |
| True  | True  | False             |