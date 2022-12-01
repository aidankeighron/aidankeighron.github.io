---
layout: post
title: File Calculator
date: 2022-11-30 12:00:00
categories: [java]
---

# File Calculator
I wanted to create a non conventional calculator so I decided to create one that takes up zero space, but how does one create a zero byte calculator.

Folders and empty files "technically" take up zero space. I know that they take up space on your filesystem's metadata files but when you look at its properties it says zero bytes. Lets add some restrictions so we are not generating millions of folders.

```json
File System
├── 0
├── 1
├── 2
│   ├── 0
│   └── 1
│       └── 3
├── 3

```

Using a directory structure like this we can 