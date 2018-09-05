# preston-amazon
a biodiversity dataset graph created with [Preston](https://github.com/bio-guoda/preston).

# steps to reproduce

This repository was created with the following recipe:

* Download a preston jar from https://github.com/bio-guoda/preston/releases/download/0.0.3/preston.jar .
* In a terminal, run ```preston update "http://api.gbif.org/v1/dataset/suggest?q=Amazon&amp;type=OCCURRENCE"```
* You have now installed some occurrence datasets related to the Amazon using the GBIF dataset registry
* Locate a dwca in the biodiversity dataset graph using:

```console 
$ preston ls -l tsv | grep "application/dcwa" | cut -f1 | tail -n1```.
http://plazi.cs.umb.edu/GgServer/dwca/341C9C4FFFDDFFEF8D1DFFCBCB25FF90.zip
```

* Now, explore the version of a single archive using:

```console
preston history http://plazi.cs.umb.edu/GgServer/dwca/341C9C4FFFDDFFEF8D1DFFCBCB25FF90.zip``` , which results in:
<http://plazi.cs.umb.edu/GgServer/dwca/341C9C4FFFDDFFEF8D1DFFCBCB25FF90.zip> <http://purl.org/pav/hasVersion> <hash://sha256/e96d41772596daee7ebf7dd73239e236ae03c81d5ac39f8df4f911fc08776e98> .
```

* Get the content-addressed file and list its content using:

```console
$ preston get hash://sha256/e96d41772596daee7ebf7dd73239e236ae03c81d5ac39f8df4f911fc08776e98 > dwca.zip ```
$ unzip -l dwca.zip
Archive:  dwca.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
    11694  2016-01-03 13:36   meta.xml
     5085  2016-01-03 13:36   eml.xml
     3720  2017-06-20 02:41   taxa.txt
      284  2017-06-20 02:41   occurrences.txt
    19069  2017-06-20 02:41   description.txt
       54  2017-06-20 02:41   distribution.txt
    53610  2017-06-20 02:41   media.txt
    10738  2017-06-20 02:41   references.txt
       33  2017-06-20 02:41   vernaculars.txt
---------                     -------
   104287                     9 files
```

* for more information, see https://github.com/bio-guoda/preston .
