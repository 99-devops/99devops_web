---
title: "What is Base56 and how is it different from base58 and base62 ?"
date: 2021-01-12T08:31:31+11:00
draft: false
description: "What is base56 and how is it different from base62"
tags: [linux, security]
categories: [devops]
---

You might know what base62 is, just for recap base62 is a encoding mechanism which uses 62 characters.

These characters consists of capital A-Z, small a-z and number 0-9 which would look like this

abcdefghijklmnopqrstuvwxyzABCDEFGHIJLKMNOPQRSTUVWXYZ0123456789

Now, take time and look at all those letters above and see which letter are confusing. 

I think you are now aware of what the problem is with this implementation. 

There are some confusing characters like '1' and 'l', '0' and 'o'

So, lets remove those characters.

23456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnpqrstuvwxyz

now, you are left with these characters. Now encoding using these characters is known as base56 implementation. Base56 is a variant of [Base58](https://en.wikipedia.org/wiki/Binary-to-text_encoding) which was invented by Satoshi Nakamoto.