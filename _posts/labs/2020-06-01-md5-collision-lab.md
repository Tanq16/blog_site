---
title: MD5 Collision Attack Lab - SeedLabs
date: 2020-06-01 12:00:00 +0500
categories: [Lab Practice Notes,Seed Labs]
tags: [md5-collision,cryptography,lab,seed-labs]
image:
  path: /assets/img/covers/seedlabs-cover.jpg
  alt: Seed Labs Artwork
---

A hash function is said to be secure if it is a one way hash function and is collision resistant. The one way property ensures that given a hash value `h`, it is computationally infeasible to find an input `m` such that `hash(m) = h`. The collision resistance property ensures that it is computationally infeasible to find two inputs `m` and `n` such that `hash(m) = hash(n)`. The experiment is about actually launching collision attacks on `MD5` hash function. A tool called `Fast MD5 Collision Generator` is used.

## Task 1 : Generating two different files with the same MD5 hash

To do this, we create two different files with same beginning part or prefix. Then we use the above mentioned tool which allows us to create an arbitrary file, the contents of which will be used as prefix to generate two files `out1.bin` and `out2.bin` which will have the same MD5 hash. Command is -

```bash
md5collgen -p prefix.txt -o out1.bin out2.bin
```

The tool generates parts P and Q for given prefix text such that `hash(prefix + P) = hash(prefix + Q)`. To check if the output files are different and the hash sums are same, we can use commands -

```bash
diff out1.bin out2.bin
md5sum out1.bin
md5sum out2.bin
```

We can use a hex editor (example GHex) to read and modify the binary files. We can use python to pass values inside the prefix file. Let examples be of the form -

```bash
echo $(python -c 'print("\x41"*55)') > prefix.txt
```

This will add 55 `A`’s to the prefix file. The examples will have 55 replaced with other numbers for different cases.

#### Prefix file < 64 bytes

If the length of the prefix file is not a multiple of 64B, we run the `md5collgen` tool to produce the two output files. The files are verified to be different by using `diff` command. The files are opened using `GHex`. On looking into the hex editor, we see that all bytes the prefix is short of a multiple of 64 are padded with regular expression - `(0A)(00)*`.

#### Prefix file = 64 bytes

If the length of the prefix file is exactly 64 bytes, the tool still pads the prefix but this time pads the next 64 bytes i.e., from byte offset 40 to 7F. The same is true when the length is a multiple of 64 bytes. Therefore, for length = 64*k, the tool adds a pad of 64 bytes after it.

#### Difference between data

The two files generated are very slightly different. Taking the example -

```bash
echo $(python -c 'print("\x41"*60)') > prefix.txt
```

This adds 4 bytes of padding and then data that may or may not differ between 2 files. For the above case, the values that were actually different were located at byte offsets 93 and BB.

## Task 2 : Understanding MD5’s property

At high level, MD5 divides its data into blocks of 64 bytes and then computes the hash iteratively on these blocks. The core of MD5 is a compression function which produces a 128 bit IHV or intermediate hash value. The input for the first iteration i.e., IHV0 is fixed. Based on the working of the MD5 algorithm, we can derive a property which is - Given two inputs `M` and `N`, if `MD5(M) = MD5(N)`, then for any input `T`, `MD5(M || T) = MD5(N || T)`. Therefore, adding a particular suffix to any two distinct messages having the same MD5 hash, gives two new longer messages for by concatenation of the original and the suffix messages, both of which also have the same MD5 hash. To demonstrate this, we use the `cat` command in `bash`, to concatenate the contents of files -

```bash
cat > prefix
asdfghjkl

md5collgen -p prefix -o file1 file2
MD5 collision generator v1.5by Marc Stevens (http://www.win.tue.nl/hashclash/)
Using output filenames: 'file1' and 'file2'
Using prefixfile: 'prefix'
Using initial value: 3c314196e1dd87fadfe827be4e35094c
Generating first block: ..........
Generating second block: S00......
Running time: 18.89 s
diff file1 file2
Binary files file1 and file2 differ
cat > suffix
qwerty
l
total 16
-rw-rw-r-- 1 seed seed 192 Mar 16 11:50 file1
-rw-rw-r-- 1 seed seed 192 Mar 16 11:50 file2
-rw-rw-r-- 1 seed seed  10 Mar 16 11:49 prefix
-rw-rw-r-- 1 seed seed   7 Mar 16 11:51 suffix
md5sum file1
a99e149410559bb7556234176df8cb2c  file1
md5sum file2
a99e149410559bb7556234176df8cb2c  file2
cat file1 suffix > modfile1
cat file2 suffix > modfile2
md5sum modfile1
c3f590a71d69ad69b0fed60867f05529  modfile1
md5sum modfile2
c3f590a71d69ad69b0fed60867f05529  modfile2
```

## Task 3 : Generating two executable files with the same MD5 hash

Given a code in C, create two different versions of this code such that the difference in them lies in the array contents, but the hash values of their executables are the same. The code is -

```cpp
#include <stdio.h>
unsigned char xyz[200] = {}; // populated with self values.
// let's see how taio reacts to very long lines when they are supposed to be printed on a pdf. This seems to be wrapping text easily, would it be that good>>??? Time to find out.
int main()
{
  int i;
  for (i=0; i<200; i++)
  {
    printf("%x", xyz[i]);
  }
  printf("\n");
  return 0;
}
```

Initially, we fill the array contents with just `A`’s. This makes it easy to spot the location of the array in the executable after compilation. To print 200 `A`’s, we can use python -

```python
print('\''+'\',\''.join(x for x in ['A']*200)+'\'')
```

Now to compile and open the executable, we use gcc and then GHex respectively. Spotting the location of a continuous block of `A`’s, the byte offset is 1040 (4160). Now the whole executable can be divided into 3 parts - byte offset 0 to x, x to y and y to end. The part y to end will be treated as a suffix. The part 0 to x is treated as prefix. The part x to y is the one where the change is required, or variant, such that - `MD5(prefix || variant1 || suffix) = MD5(prefix || variant2 || suffix)`. We keep the prefix little over the byte offset of the first `A` as well as a multiple of 64. Therefore, choosing byte offset 4224, everything of the first 4224 bytes is the prefix.

```bash
head -c 4224 a.out > prefix
```

Using this prefix file for md5collgen, get two files having the same hash called file1 and file 2.

```
md5collgen -p prefix -o file1 file2
```

This gives the files with ending byte offset as 10FF. Therefore, we need to keep the bytes after 10FF from the original binary as the suffix.

```
tail -c +4352 a.out > suffix
```

Now to create the two binaries, concatenate the suffix to the two individual files.

```
cat file1 suffix > code1$ cat file2 suffix > code2
```

These code files can be made executables, which upon execution print different data (one printed 1 byte more than the other in this case). However, the md5sum of both the codes are the same. One way to make sure that the data output by the code files are different is to compute a hash on them. If the hashes are different then the files indeed printed different data.

```
echo $(./code1) | md5sumededed2819bd22f8732296e63229ca40  -
echo $(./code2) | md5sumb12e08dfdbd9217653c20c39eb290aba  -
```

This proves that the experiment was successful in creating two different binaries from a single binary, both producing different output but having the same md5 hash.

## Task 4 : Making two programs behave differently

This task requires us to exploit the hash collision vulnerability. The main idea is to create a program that executes malicious code even after it has been verified as well as checked by hashing it. Therefore, the code needs to have two parts - a malicious and a good part. If the factor that decides whether the good code is executed or the bad one is something that can be exploited, then the attacker can use that to write a code that can pass all verification checks and still manage to run the malicious code. The idea behind the factor is to keep two arrays. If the contents are the same, execute good code, otherwise execute malicious code. Therefore, write a program as follows.

```cpp
#include<stdio.h>

unsigned char b[200] = {<populate>};
unsigned char a[200] = {<populate>};

int main()
{
	int flag = 1;
	int i;
	for(i=0;i<200;i++)
	{
		//printf("%c  ", a[i]);
		if(a[i] != b[i])
		{
			flag = 0;
			break;
		}
	}
	if(flag)
		printf("good code!\n");
	else
		printf("bad code!\n");
	return 0;
}
```

The arrays are populated with `A`’s, using python -

```python
print('\''+'\',\''.join(x for x in ['A']*200)+'\'')
```

Compile to get an executable, `a.out`. All work will be done on this executable. Set the prefix using head command - this includes all the bytes before the start of the first array. Then generate two files using this prefix, which yields `out1` and `out2` having all except the last 8 elements of the first array. Add all bytes after the 4352nd one in `a.out` to `suffixtest`.

```
head -c 4224 a.out > prefix
md5collgen -p prefix -o out1 out2
tail -c +4353 a.out > suffixtest
```

To complete the first array in the files `out1` and `out2`, add the first 8 bytes of `suffixtest` to both giving files `out1arrcomplete` and `out2arrcomplete`. Now the `suffix` file is created which contains all bytes after the 8th byte in `suffixtest`.

```
head -c 8 suffixtest > arrcomplete
cat out1 arrcomplete > out1arrcomplete
cat out2 arrcomplete > out2arrcomplete
tail -c +9 suffixtest > suffix
```

Now, to add the bytes between the ending of the first array and the beginning of the second array, make a file called `tillnext`. Store the bytes beginning with the second array in `suffix` to `suffixtest`. Add these bytes to `out1arrcomplete` and `out2arrcomplete` to give `file1tillnext` and `file2tillnext`.

```
tail -c +25 suffix > suffixtest
head -c 24 suffix > tillnext
cat out1arrcomplete tillnext > file1tillnext
cat out2arrcomplete tillnext > file2tillnext
```

Now the two result files are the two separate part executables which have the contents up to beginning of the second array. To make the attack successful, one file needs to print “good code” while the other “bad code”. To do this the contents of the second array needs to be equal to one of the generated arrays. So, put the bytes after the second array in `suffixtest` to `suffix`. Then copy the first array from `out1arrcomplete` to `comparray`. The file `comparray` can be appended to `file1tillnext` and `file2tillnext` along with `suffix` to give the final executables `firstexec` and `secondexec`.

```
tail -c +201 suffixtest > suffix
tail -c +4161 out1arrcomplete > comparray
cat file1tillnext comparray suffix > firstexec
cat file2tillnext comparray suffix > secondexec
```

Make these two final files executable and calculate the md5 hash sum. Executing them both gives desired results.

```
md5sum firstexec
e68a168be99f12c1bc782b7da5603f62  firstexec
md5sum secondexec
e68a168be99f12c1bc782b7da5603f62  secondexec

chmod +x firstexec
chmod +x secondexec

./firstexec
good code!

./secondexec
bad code!
```

This is how the md5 collision vulnerability can be exploited.

---

> The above is a documentation of a lab experiment by the name MD5 Collision Attack Lab from publicly available seed labs by Syracuse University. [Seed Labs](http://www.cis.syr.edu/~wedu/seed/labs.html) Copyright &copy; Wenliang Du, Syracuse University. I do not own any software mentioned in the above document.
