# preston-amazon
A sample biodiversity dataset graph created with [Preston](https://github.com/bio-guoda/preston).

# introduction
This repository provides a small subset of biodiversity datasets registered with the [GBIF](https://gbif.org). The purposes is to show an example of (1) how to [track](#tracking-data), (2) [pre-process](#pre-processing-data) and (3) [analyze](#analyzing-data) datasets with [Preston](https://preston.guoda.bio) and [Apache Spark](https://spark.apache.org). 


# tracking data

The ```data``` directory contains original datasets and provanance information created with the help of Preston using the following recipe:

* Download a preston jar from https://github.com/bio-guoda/preston/releases/download/0.0.10/preston.jar .
* In a terminal, run preston to track a graph of biodiversity occurrence datasets related to the Amazon using GBIF's api:

```console
$ java -jar preston.jar track "http://api.gbif.org/v1/dataset/suggest?q=Amazon&amp;type=OCCURRENCE"
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
$ java -jar preston.jar ls -l tsv | grep "application/dwca" | cut -f1 | tail -n1
http://plazi.cs.umb.edu/GgServer/dwca/341C9C4FFFDDFFEF8D1DFFCBCB25FF90.zip
```

* Now, explore the version of a single archive using:

```console
$ java -jar preston.jar ls | grep "http://plazi.cs.umb.edu/GgServer/dwca/341C9C4FFFDDFFEF8D1DFFCBCB25FF90.zip"
<663199f1-3528-4289-8069-d27552f62f10> <http://www.w3.org/ns/prov#hadMember> <http://plazi.cs.umb.edu/GgServer/dwca/341C9C4FFFDDFFEF8D1DFFCBCB25FF90.zip> .
<http://plazi.cs.umb.edu/GgServer/dwca/341C9C4FFFDDFFEF8D1DFFCBCB25FF90.zip> <http://purl.org/dc/elements/1.1/format> "application/dwca" .
<http://plazi.cs.umb.edu/GgServer/dwca/341C9C4FFFDDFFEF8D1DFFCBCB25FF90.zip> <http://purl.org/pav/hasVersion> <hash://sha256/e96d41772596daee7ebf7dd73239e236ae03c81d5ac39f8df4f911fc08776e98> .
```

* Get the content-addressed file and list its content using:

```console
$ java -jar preston.jar get hash://sha256/e96d41772596daee7ebf7dd73239e236ae03c81d5ac39f8df4f911fc08776e98 > dwca.zip
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

# pre-processing data

Now that we have a collection of datasets of known provenance, we use [Apache Spark](https://apache.spark.org) with a [idigbio-spark](https://github.com/bio-guoda/idigbio-spark), a Spark library, to transform the data for further analysis. Our example collection contains Darwin Core Archives. These archives are typically distributed using zip archives. To prepare these Darwin Core archives for analysis, we take following steps pre-processing steps:  

1. unzip the Darwin Core archives and bzip2 the entries

2. collect data schemas from all ```meta.xml``` files into [meta.xml.seq](./data-processed/meta.xml.seq)

3. use data schemas to convert core tables of the datasets to individual ```core.parquet``` files (e.g., [core.parquet](./data-processed/22/0f/220f6dd60ceba458c9b942e205675773d336ab3b0227e3fc04e7c854c85811ad/core.parquet))

4. combine all the individual ```core.parquet``` files into a single ```core.parquet``` (e.g., [core.parquet](./core.parquet)) 

Note that [Apache Parquet](https://parquet.apache.org) and [Hadoop Sequence Files](https://wiki.apache.org/hadoop/SequenceFile) are file formats optimized for distributed/parallel processing. 

See https://github.com/bio-guoda/preston-amazon and https://github.com/bio-guoda/preston-scripts for examples.  

# analyzing data

After pre-processing the data in formats suitable for scalable analysis, we can easily discover meta-data in eml.xml and meta.xml files. Also, the data itself can be queried similar to a database using Spark supported languages like python, R, java, and scala . For the examples below, we'll use pyspark, an interactive Spark python shell . For information how to install / start pyspark, see https://spark.apache.org/docs/latest/#running-the-examples-and-shell . 

## finding distinct scientific names


```python
>>> datasets = spark.read.parquet("file:///some/path/preston-amazon/data-processed/core.parquet") # load aggregate data
>>> datasets.count() # count all rows
3858
>>> datasets.columns # print columns
['http://rs.tdwg.org/dwc/terms/datasetID', 'http://rs.tdwg.org/dwc/terms/specificEpithet', 'http://rs.tdwg.org/dwc/terms/order', 'http://rs.tdwg.org/dwc/terms/taxonID', 'http://rs.tdwg.org/dwc/terms/country', 'http://plazi.org/terms/1.0/basionymYear', 'http://gbif.org/dwc/terms/1.0/canonicalName', 'undefined0', 'http://rs.tdwg.org/dwc/terms/basisOfRecord', 'http://plazi.org/terms/1.0/combinationYear', 'http://plazi.org/terms/1.0/basionymAuthors', 'http://rs.tdwg.org/dwc/terms/scientificName', 'http://rs.tdwg.org/dwc/terms/decimalLatitude', 'http://rs.tdwg.org/dwc/terms/eventDate', 'http://rs.tdwg.org/dwc/terms/waterBody', 'http://rs.tdwg.org/dwc/terms/acceptedNameUsageID', 'http://rs.tdwg.org/dwc/terms/locationID', 'http://rs.tdwg.org/dwc/terms/taxonRank', 'http://rs.tdwg.org/dwc/terms/institutionCode', 'http://rs.tdwg.org/dwc/terms/phylum', 'http://purl.org/dc/terms/references', 'http://rs.tdwg.org/dwc/terms/originalNameUsageID', 'http://rs.tdwg.org/dwc/terms/individualCount', 'http://rs.tdwg.org/dwc/terms/kingdom', 'http://rs.tdwg.org/dwc/terms/year', 'http://rs.tdwg.org/dwc/terms/eventID', 'http://rs.tdwg.org/dwc/terms/identificationQualifier', 'http://rs.tdwg.org/dwc/terms/namePublishedIn', 'http://rs.tdwg.org/dwc/terms/scientificNameAuthorship', 'http://rs.tdwg.org/dwc/terms/taxonomicStatus', 'http://rs.tdwg.org/dwc/terms/decimalLongitude', 'http://rs.tdwg.org/dwc/terms/locality', 'http://rs.tdwg.org/dwc/terms/parentNameUsageID', 'http://rs.tdwg.org/dwc/terms/catalogNumber', 'http://rs.tdwg.org/dwc/terms/collectionCode', 'http://plazi.org/terms/1.0/combinationAuthors', 'http://purl.org/dc/terms/license', 'http://purl.org/dc/terms/bibliographicCitation', 'http://purl.org/dc/terms/accessRights', 'http://rs.tdwg.org/dwc/terms/family', 'http://rs.tdwg.org/dwc/terms/dynamicProperties', 'http://plazi.org/terms/1.0/verbatimScientificName', 'http://rs.tdwg.org/dwc/terms/eventRemarks', 'http://rs.tdwg.org/dwc/terms/class', 'http://rs.tdwg.org/dwc/terms/occurrenceID', 'http://rs.tdwg.org/dwc/terms/nomenclaturalStatus', 'http://rs.tdwg.org/dwc/terms/genus', 'http://purl.org/dc/terms/rightsHolder', 'http://www.w3.org/ns/prov#wasDerivedFrom']
>>> datasets.select('`http://rs.tdwg.org/dwc/terms/scientificName`').show(10) # show top 10 scientific names
+-------------------------------------------+
|http://rs.tdwg.org/dwc/terms/scientificName|
+-------------------------------------------+
|                       Acestrorhynchus g...|
|                       Charax sp. “Madeira”|
|                       Ageneiosus inermi...|
|                       Cyphocharax spilu...|
|                       Moenkhausia dichr...|
|                       Ctenobrycon spilu...|
|                       Hemiodus microlep...|
|                        Loricariichthys sp.|
|                       Trachydoras parag...|
|                       Hoplias malabaric...|
+-------------------------------------------+
only showing top 10 rows
>>> datasets.select('`http://rs.tdwg.org/dwc/terms/scientificName`').count()
3858
>>> datasets.select('`http://rs.tdwg.org/dwc/terms/scientificName`').distinct().count()
708
```

## meta-data

Preston data was pre-processed using a dedicated spark library, [idigbio-spark](https://github.com/bio-guoda/idigbio-spark). This library helps create sequence files of meta.xml and emls files in a Preston archive. The advantage of sequence files is that you can quickly iterate over many items. Large Preston archives consist of hundreds of thousands or millions of small meta-data files. Accessing these files individually can be done through Spark, but is very slow due to IO and memory overhead. The ```data-processed/meta.xml.seq``` files contain tuples/pairs of originating data archive content hash and the content of their meta.xml file. To access this data:

```python
>>> metas = sc.sequenceFile("files:///some/path/preston-amazon/data-processed/meta.xml.seq") # create the meta.xml RDD with (hash, xml string) tuples
>>> metas.map( lambda pair: pair[0] ).distinct().count() # count number of unique archive content hashes
37
>>> print(metas.map( lambda pair: pair[1] ).first()) # print the first meta.xml file
<archive metadata="eml.xml" xmlns="http://rs.tdwg.org/dwc/text/">
  <core rowType="http://rs.tdwg.org/dwc/terms/Taxon" ignoreHeaderLines="1" fieldsEnclosedBy="" linesTerminatedBy="\n" fieldsTerminatedBy="\t" encoding="UTF-8">

    <files>
      <location>taxa.txt</location>
    </files>
    <id index="0"/>
    <field term="http://rs.tdwg.org/dwc/terms/taxonID" index="0"/>
    <field term="http://rs.tdwg.org/dwc/terms/namePublishedIn" index="1"/>
    <field term="http://rs.tdwg.org/dwc/terms/acceptedNameUsageID" index="2"/>
    <field term="http://rs.tdwg.org/dwc/terms/parentNameUsageID" index="3"/>
    <field term="http://rs.tdwg.org/dwc/terms/originalNameUsageID" index="4"/>
    <field term="http://rs.tdwg.org/dwc/terms/kingdom" index="5"/>
    <field term="http://rs.tdwg.org/dwc/terms/phylum" index="6"/>
    <field term="http://rs.tdwg.org/dwc/terms/class" index="7"/>
    <field term="http://rs.tdwg.org/dwc/terms/order" index="8"/>
    <field term="http://rs.tdwg.org/dwc/terms/family" index="9"/>
    <field term="http://rs.tdwg.org/dwc/terms/genus" index="10"/>
    <field term="http://rs.tdwg.org/dwc/terms/taxonRank" index="11"/>
    <field term="http://rs.tdwg.org/dwc/terms/scientificName" index="12"/>
    <field term="http://rs.tdwg.org/dwc/terms/scientificNameAuthorship" index="13"/>
    <field term="http://gbif.org/dwc/terms/1.0/canonicalName" index="14"/>
    <field term="http://plazi.org/terms/1.0/verbatimScientificName" index="15"/>
    <field term="http://plazi.org/terms/1.0/basionymAuthors" index="16"/>
    <field term="http://plazi.org/terms/1.0/basionymYear" index="17"/>
    <field term="http://plazi.org/terms/1.0/combinationAuthors" index="18"/>
    <field term="http://plazi.org/terms/1.0/combinationYear" index="19"/>
    <field term="http://rs.tdwg.org/dwc/terms/taxonomicStatus" index="20"/>
    <field term="http://rs.tdwg.org/dwc/terms/nomenclaturalStatus" index="21"/>
    <field term="http://purl.org/dc/terms/references" index="22"/>
  </core>

  <snip/> <!-- removed a bunch of stuff here -->

</archive>

```

# Funding 

This work is funded in part by grant [NSF OAC 1839201](https://www.nsf.gov/awardsearch/showAward?AWD_ID=1839201&HistoricalAwards=false) from the National Science Foundation.
