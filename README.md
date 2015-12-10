# HGAP: Hierarchical Genome Assembly Process


Table of contents
=================

  * [Overview](#overview)
  * [Manual](#manual)
    * [Running with SMRTLink](#running-with-smrtlink)
    * [Running on the Command Line with pbsmrtpipe](#running-on-the-command-line-with-pbsmrtpipe)
  * [Advanced Analysis Options](#advanced-analysis-options)
    * [SMRTLink/pbsmrtpipe HGAP Options](#smrtlinkpbsmrtpipe-HGAP-options)
  * [Files](#files)
  * [Algorithm Modules](#algorithm-modules)
  * [Glossary](#glossary)

#Overview

#Manual

##Running with SMRTLink

##Running on the Command-Line with pbsmrtpipe
###Install pbsmrtpipe
pbsmrtpipe is a part of `smrtanalysis-3.0` package and will be installed
if `smrtanalysis-3.0` has been installed on your system. Or you can [download   pbsmrtpipe](https://github.com/PacificBiosciences/pbsmrtpipe) and [install](http://pbsmrtpipe.readthedocs.org/en/master/).
    
You can verify that pbsmrtpipe is running OK by:

    pbsmrtpipe --help

### Create a dataset
Now create an XML file from your subreads.

```
dataset create --type SubreadSet my.subreadset.xml subreads1.bam subreads2.bam ...
```
This will create a file called `my.subreadset.xml`. 


### Create and edit HGAP Analysis options and global options for `pbsmrtpipe`.
Create a global options XML file which contains SGE related, job chunking and
job distribution options that you may modify by:

```
 pbsmrtpipe show-workflow-options -o global_options.xml
```

Create an HGAP options XML file which contains HGAP related options that 
you may modify by:
```
 pbsmrtpipe show-template-details pbsmrtpipe.pipelines.polished_falcon_lean -o hgap_options.xml
```

The entries in the options XML files have the format:

```
 <option id="pbtranscript.task_options.min_seq_len">
            <value>300</value>
        </option>
```
And you can modify options using your favorite text editor, such as vim.

### Run the HGAP application from pbsmrtpipe
Once you have set your options, you are ready to run the HGAP software application via pbsmrtpipe:

```
pbsmrtpipe pbsmrtpipe.pipelines.polished_falcon_lean -e eid_subread:my.subreadset.xml --preset-xml=hgap_options.xml --preset-xml=global_options.xml
```
