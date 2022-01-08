---
layout: page
title: TIL
permalink: /til
---

This page is for one line snippets of knowledge that I have learned, typically throughout the current day.


---

There is a parenthesis missing in the original Lisp paper, in the `evcon` function.

---

In GCC C/C++, the only case when `sizeof` is evaluated at run time instead of the usual compile time, is when its argument is a Variable-Length Array.

---

You can compute the diameter of a tree by finding the deepest leaf, then finding the deepest leaf from there, although I do not know of any proof.

---

In python, it is sometimes faster to do `deepcopy = lambda x: pickle.loads(pickle.dumps(x))` than the original `deepcopy`.

---

In ISO C pointer arithmetic, `&*x` cancels out never truly referencing `*x`, making this safe even when `x = NULL`.

---

In ISO C pointer arithmetic, addition is commutative, making `"string"[2] ≡ *("string" + 2) ≡ *(2 + "string") ≡ 2["string"]`.

---

A permutation is sorted in $O(n)$ by swapping values in-place, faster than the general comparison-based sorting optimum of $O(n \log n)$

---
<!-- - C signed/unsigned comparison -->
<!-- https://stackoverflow.com/questions/5416414/signed-unsigned-comparisons -->

GCC casts to unsigned when doing signed-unsigned comparisons, making `-1 > 2U` equivalent to `(unsigned) -1 > 2U`, which evaluates to `1` (`true`).
