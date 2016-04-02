+++
date = "2016-03-20T03:42:37+08:00"
draft = true
title = "how to sum all numbers in file in bash"

+++

### I have a file, with one number by line, how to get the sum

```bash
paste -s -d+  myfile_with_numbers | bc 
```

and if you have a csv, delimited by `,` and for which the number is on
the 2nd field:

```
cut -d,  my_file.csv -f 2 | paste -s -d+  | bc 
```

