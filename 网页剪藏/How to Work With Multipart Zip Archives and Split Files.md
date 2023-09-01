
Last updated: November 30, 2022

Written by: [baeldung](https://www.baeldung.com/linux/author/baeldung "Posts by baeldung")

*   [File Compression](https://www.baeldung.com/linux/category/files/compression)

1\. Overview[](#overview)
-------------------------

Handling big files has always been challenging. We need to cope with the limited storage medium capacity as well as restrictions on the web services transfer.

In this tutorial, we’ll learn how to split files, create archives right away in parts, and then restore the original content.

2\. Splitting Binary File[](#splitting-binary-file)
---------------------------------------------------

We can divide a file with [_split_](/linux/split-files). So as an example, let’s split the _ubuntu-22.04-desktop-amd64.iso_ file:

```null
$ split -d -b 500M ubuntu-22.04-desktop-amd64.iso ubuntu_parts/ubuntu_piece_
```

Let’s notice that we use the _b_ option to set the size of parts to 500 MB and the _d_ option to create numerical suffixes. **Furthermore, we point out the folder for results by simply adding it to the prefix _ubuntu\_parts/ubuntu\_pieces__.** Of course, this folder must exist. Now, let’s check the result:

```null
$ ls -sh1 ubuntu_parts
total 3.5G
500M ubuntu_piece_00
500M ubuntu_piece_01
501M ubuntu_piece_02
500M ubuntu_piece_03
500M ubuntu_piece_04
501M ubuntu_piece_05
486M ubuntu_piece_06
```

Then, let’s combine the pieces back into the _iso_ file:

freestar.config.enabled\_slots.push({ placementName: "baeldung\_leaderboard\_mid\_1", slotId: "baeldung\_leaderboard\_mid_1" });

```null
$ cat ubuntu_parts/* > ubuntu_again.iso
```

Let’s emphasize that _cat_ preserves the order of the parts as it processes input files alphabetically. Moreover, _cat_ accepts both text and binary files.

Afterward, we can verify the MD5 checksum with [_md5sum_](https://man7.org/linux/man-pages/man1/md5sum.1.html) to make sure that the file is correctly restored:

```null
$ md5sum ubuntu_again.iso
7621da10af45a031ea9a0d1d7fea9643  ubuntu_again.iso
$ md5sum ubuntu-22.04-desktop-amd64.iso
7621da10af45a031ea9a0d1d7fea9643  ubuntu-22.04-desktop-amd64.iso
```

3\. Using _zip_[](#using-zip)
-----------------------------

Instead of splitting the existing file, we can create the split archive right away. So, let’s use _[zip](/linux/working-with-zip-command-in-linux)_, which since version 3.0 supports this kind of archive. **Then, let’s pass the maximum size of the created file with the _s_ option.** Furthermore, as units, we can use k, m, g, and t for kilobytes, megabytes, gigabytes, and terabytes, respectively. The default unit is megabytes.

So, let’s prepare a backup of the ~/_Picture_ folder in 180-megabyte parts:

```null
$ zip -r -s 180 Pictures_backup.zip ~/Pictures
```

Now let’s examine the result files:

freestar.config.enabled\_slots.push({ placementName: "baeldung\_leaderboard\_mid\_2", slotId: "baeldung\_leaderboard\_mid_2" });

```null
$ ls -sh1 Pictures_backup.*
180M Pictures_backup.z01
180M Pictures_backup.z02
180M Pictures_backup.z03
180M Pictures_backup.z04
180M Pictures_backup.z05
7.4M Pictures_backup.zip
```

So, let’s notice that _zip_ enumerates files on its own, using extensions like _z01_, _z02_, and so on. **Moreover, we shouldn’t change that, as we’d make extracting the archive impossible.**

4\. _unzip_ Multi-File Archive[](#unzip-multi-file-archive)
-----------------------------------------------------------

**To extract the zipped multi-file archive, we need to ‘fix’ it first.** So, let’s run _zip_ with the _FF_ option on the ‘head’ file of the archive. In addition, we need to use the _out_ option:

```null
$ zip -FF Pictures_backup.zip --out Pictures_single.zip
```

Next, we can [_unzip_](/linux/zip-unzip-command-line) the result file _Pictures_single.zip_ here to a temporary directory indicated by the _d_ option:

```null
unzip Pictures_single.zip -d ./temp
```

Finally, let’s notice that if we’ve split an ordinary one-part zip archive with _split_, we don’t need to ‘fix’ it. Then all we should do is join parts with _cat_ and subsequently unzip the result.

5\. Dealing With Missing Parts[](#dealing-with-missing-parts)
-------------------------------------------------------------

**In the case when some parts of the multipart zip archive are missing, we can still fix it with the _FF_ option to _zip_.** Let’s notice that we can use the ‘head’ _Pictures_backup.zip_ file as an argument, even if it’s the missing one. Next, while the command is running, we’re going to come across the menu:

freestar.config.enabled\_slots.push({ placementName: "baeldung\_leaderboard\_mid\_3", slotId: "baeldung\_leaderboard\_mid_3" });

```null
Could not find:
  Pictures_backup.z06

Hit c      (change path to where this split file is)
    s      (skip this split)
    q      (abort archive - quit)
    e      (end this archive - no more splits)
    z      (look for .zip split - the last split)
 or ENTER  (try reading this split again): 
```

We should react accordingly. Usually, when the ‘head’ file is lacking, _zip_ asks for a redundant file. In this example, it’s _Pictures_backup.z06_. Thus, we need to tell when to end. On the other hand, we should skip a missing component file.

Finally, we’ll unzip the created one-file zip file.

6\. Conclusion[](#conclusion)
-----------------------------

In this article, we learned to split a file into parts and join pieces back together. Next, we described how to apply popular compressing utilities, _zip_ and _unzip,_ to handle split archives.

**Let’s notice that other compressing utilities offer the possibility to create a multi-file archive too. So, we can use the _v_ option to [_7z_](/linux/split-files) or the _M_ flag to [_tar_](/linux/tar-command).**

Comments are closed on this article!

freestar.config.enabled\_slots.push({ placementName: "baeldung\_leaderboard\_btf\_2", slotId: "baeldung\_leaderboard\_btf_2" });