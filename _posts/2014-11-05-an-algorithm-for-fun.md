---
layout: post
title: An Algorithm for Fun
---

Today we three were very boring. So boring that we casually made out a question. We have n characters (just suppose n <= 26 and they are 'a', 'b', ... 'a' + n - 1). What is the kth plalindrome formed by that n characters unduplicatedly? (It is OK only if one char is duplicated because of palindrome, that is to say, each charater can appear at most twice).

<!--more-->

## The story

We had this question just for fun, but we three could not stop thinking about that and we did get an algorithm of construciton.

We will finally have a sequence of plalindromes. And when we know one plalindrome in that sequence, we can construct the next one.

## Case 0

At the beginning we suppose that the plalindrome can have duplicated characters and its length can only be m. Then the question may be quite easy. We just need to care about only the first half of the plalindrome, and the question turns in to a simple permutation.

So we give up such case.

## Case 1

If the plalindrome cannot be longer than m (m is an interger). Also we only care the first half.

Suppose n = 5, m = 4, the first half of the plalindrome sequence is:

```
a, ab, abc, abcd, abce, abd, abde, ...
```

In fact, transfer from one to the next one, there are only two cases:

> - If the m characters are not used up, then pick out the smallest and append it to the tail
> - If m characters are used up, then do some swapping just as simple permutation generation, and cut out the tail of the string.

There are also some other details of the algorithm. Each element of the above sequence can generate two result plalindromes, so he real final result is:

```
a(a, aa), ab(aba, abba), abc(abcba, abccba), abcd(abcdcba, abcddcba), abce(abcecba, abceecba), abd(abdba, abddba), ...
```

I doubt whether the time complexitiy is something like O(kn).

Or we make it faster using DP?

## Case 2

If the plalindrome can be longer than m, which means no limitation is set on the length.

Suppose n = 5, the first half of the plalindrome sequence is:

```
a, ab, abc, abcd, abcde, abce, abced, ...
```

Maybe similar to the Case 1


## Am I right?

objectkuan AT gmail DOT com
