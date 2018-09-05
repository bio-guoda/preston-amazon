# preston-amazon
a biodiversity dataset graph created with [Preston](https://github.com/bio-guoda/preston).

# steps to reproduce

This repository was created with the following recipe:

* Download a preston jar from https://github.com/bio-guoda/preston/releases/download/0.0.3/preston.jar .
* In a terminal, run preston to update the graph of biodiversity datasets.

```console
$ preston update "http://api.gbif.org/v1/dataset/suggest?q=Amazon&amp;type=OCCURRENCE"
<https://preston.guoda.org> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/prov#SoftwareAgent> .
<https://preston.guoda.org> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/prov#Agent> .
<https://preston.guoda.org> <http://purl.org/dc/terms/description> "Preston is a software program that finds, archives and provides access to biodiversity datasets."@en .
<83e11589-3579-47d8-ad5b-126e640112cd> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/prov#Activity> .
<83e11589-3579-47d8-ad5b-126e640112cd> <http://purl.org/dc/terms/description> "A crawl event that discovers biodiversity archives."@en .
<83e11589-3579-47d8-ad5b-126e640112cd> <http://www.w3.org/ns/prov#startedAtTime> "2018-09-05T09:42:39.614Z"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
...
```

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
