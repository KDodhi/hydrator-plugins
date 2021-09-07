PAR Format Thrift
=========================

This implementation of the InputFormat can be used in Source Plugins (i.e. GCS Source) to read Thrift 
PARC (Parsed Anonymized Record Collection) files; the output would consist of multiple StructuredRecord(s) 
each one representing a PAR (data) from the Collection (PAR Collection = PARC).

More Details
------------

A Parsed Anonymized Record Collection (PARC) refers to parsed data tied to anonymous (meaning not PII) identifiers. 
PARC consists of a number of files stored i.e. in a Bucket Data Store,
containing ParsedAnonymizedRecord Thrift objects.

In general an InputFormat describes the input-specification for a Map-Reduce job.

The Map-Reduce framework relies on the InputFormat of the job to:
* Validate the input-specification of the job.
* Split-up the input file(s) into logical InputSplits, each of which is then assigned to an individual Mapper.
* Provide the RecordReader implementation to be used to glean input records from the logical InputSplit for 
  processing by the Mapper.
  
For this PARC-Thrift plugin, the implementation of the points above is left to the class SequenceFileRecordReader 
- https://hadoop.apache.org/docs/r2.9.2/api/index.html?org/apache/hadoop/io/SequenceFile.html as this would take care of
the files' headers and compression.

Code Entry Points
-----------------

The class representing the format-thrift plugin is the `ThriftInputFormatProvider` - this 
is the entry point of the plugin (from the framework).

Build
-----
To build this plugin:

```
   mvn clean package
```

The build will create a .jar and .json file under the ``target`` directory.
These files can be used to deploy your plugins.

## Deployment
You can deploy your plugins using the CDAP CLI or the CDAP UI as you would normally deploy any plugin/artifact.
See section [Deploy as a User Artifact](https://cdap.atlassian.net/wiki/spaces/DOCS/pages/477561299/Plugin+Management#Deploying-as-a-User-Artifact).

### How to use the new format

In order to use the newly deployed `thrift` format, the format-common code need to be updated to include
thrift between the potential values of the `io.cdap.plugin.format.FileFormat` enumeration.

Once the format-common has been updated, any hydrator plugins could be re-packaged using this 
version and deployed as USER artifact.

In summary, i.e. to use the new format in GCS Source plugin:
1. package the format-thrift and deploy it to DOP
1. update the format-common to allow `thrift` format (io.cdap.plugin.format.FileFormat enumeration)
1. re-package the google-cloud plugin, referencing the updated format-common
1. deploy the updated GCS as USER artifact
1. set `.*\.bucketfile` in the Regex Path Filter section of the GCS property in order to only read the `*.bucketfile` 

**If there's any exception during the reading of any PAR record or part file (corrupt file),
the plugin fails and no output is generated.**