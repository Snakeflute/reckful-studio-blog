---
title: LeetCode 0194 - Transpose File
date: 2018-06-11 14:00:07
categories: LeetCode
---
# Transpose File

<!--more-->

## Desicription

Given a text file `file.txt`, transpose its content.

You may assume that each row has the same number of columns and each field is separated by the ' ' character.

**Example**:

If `file.txt` has the following content:

```
name age
alice 21
ryan 30
```

Output the following:

```
name alice ryan
age 21 30
```

## Solution

```sh
# Read from the file file.txt and print its transposed content to stdout.
awk ' {
    for(i = 1; i <= NF; i++) {
        if(NR == 1) {
            s[i] = $i;
        }
        else {
            s[i] = s[i] " " $i;
        }
    }
}
END {
    for(i = 1; s[i] != ""; i++) {
        print s[i];
    }
}' file.txt
```