RF2 to json conversion utility
==============================

Conversion of SNOMED CT RF2 files to a JSON format, including pre-computed indexes for common search strategies

Building
--------

Build the project using Maven, it will generate an executable jar with embedded dependencies (i.e. "target/rf2-to-json-conversion-1.0-SNAPSHOT-jar-with-dependencies.jar").

Conversion configuration
------------------------

Create a config.xml for the conversion execution, it requires to define the location of the source RF2 Snapshot files.

```
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <processInMemory>true</processInMemory>
    <defaultTermLangCode>en</defaultTermLangCode>
    <defaultTermDescriptionType>900000000000003001</defaultTermDescriptionType>
    <defaultTermLanguageRefset>900000000000509007</defaultTermLanguageRefset>
    <normalizeTextIndex>true</normalizeTextIndex>
    <createCompleteConceptsFile>true</createCompleteConceptsFile>
    <editionName>International Edition</editionName>
    <databaseName>en-edition</databaseName>
    <effectiveTime>20140131</effectiveTime>
    <expirationTime>20150201</expirationTime>
    <outputFolder>/Users/alo/Downloads/Releases/en-json</outputFolder>
    <foldersBaselineLoad>
        <folder>/Users/alo/Downloads/Releases/SnomedCT_Release_INT_20140131/RF2Release/Snapshot</folder>
    </foldersBaselineLoad>
    <modulesToIgnoreBaselineLoad>
    </modulesToIgnoreBaselineLoad>
    <foldersExtensionLoad>
    </foldersExtensionLoad>
    <modulesToIgnoreExtensionLoad>
    </modulesToIgnoreExtensionLoad>
</config>
```
Variables:
  * *processInMemory*: true for processing with memory maps, really fast, but it requires 8gb of RAM for simple proyects (International Edition + 1 extension). Set to false for using disk based maps, slower but runs with less memory.
  * *defaultTermLangCode*: Language code for default term selection.
  * *defaultTermDescriptionType*: description type SCTID for the default term selection, some prefer to use the FSN as default term, other prefer to use a Synonym (the preferred synonym will be picked).
  * *defaultTermLanguageRefset*: language reference set used to identify the preferred acceptability.
  * *normalizeTextIndex*: setting this to True will normalize the text index with diacritics removal.
  * *createCompleteConceptsFile*: Always set to false, true provides an additional level of denormalization not implemented in any APIS today and increases significantly the size of the resulting model.
  * *editionName*: name for this edition.
  * *databaseName*: short name for the edition, used for the ongodb database.
  * *effectiveTime*: release date/effective time for the finl package, if it combines the International Edition and an extension use the later date, usually the extension one.
  * *expirationTime*: date when a warning needs to appear in the browser to announce that data may be deprecated.
  * *outputFolder*: folder where the resulting .json files will be stored.
  * *foldersBaselineLoad*: list of folders with baseline edition snapshots, usually the international edition.
  * *modulesToIgnoreBaselineLoad*: list of Modules SCTIDs in the baseline files to ignore during conversion.
  * *foldersExtensionLoad*: list of folders with extension edition files.
  * *modulesToIgnoreExtensionLoad*: list of Modules SCTIDs in the extension files to ignore during conversion.
 
If the base edition and the extension are available in a single RF2 snapshot package, this folder can be used as a single baseline folder. All folders will be recursively expored and RF2 files will be indentified base on file names and headers (firs row of the file).

You can find sample config files in the ```config``` folder.

Executing conversion
--------------------

Run the executable jar file:

```
java -Xmx8g -jar rf2-to-json-conversion-1.0-SNAPSHOT-jar-with-dependencies.jar config.xml
```

The results will be a set of .json files in the output folder.

Importing data
--------------

The snapshot API requires a MongoDB database to store the denormalized representation contained in the .json files. MongoDb 2.6.x is required to leverage its latest adavances in full text search.

This project includes a script file for importing the resulting .json files into MongoDB, and creating the necessary indexes. This sentence needs to be executed from the folder that contains the .json files. The script will use the `mongoimport` command provided by mongoDB, it is expected to be in the path, as well as `mongo` runtime.

```./import.sh en-edition 20140731```

The first parameter is the edition short name, and the second is the effective time.
The edition and effective time will be used to configure the Rest API that runs with this data.

