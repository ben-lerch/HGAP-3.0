# HGAP: Hierarchical Genome Assembly Process



Table of contents
=================

  * [Overview](#overview)
  * [Manual](#manual)
    * [Running with SMRT Link](#running-with-smrt-link)
    * [Running on the Command Line with pbsmrtpipe](#running-on-the-command-line-with-pbsmrtpipe)
  * [Advanced Analysis Options](#advanced-analysis-options)
    * [SMRT Link/pbsmrtpipe HGAP Options](#smrt-linkpbsmrtpipe-resequencing-options)
  * [Files](#files)
  * [Algorithm Modules](#algorithm-modules)
  * [Glossary](#glossary)

#Overview

![hgap](https://cloud.githubusercontent.com/assets/12494820/12428471/9248383a-be99-11e5-9473-ac5b5bb628af.png)



Analyses are performed in two basic stages: FALCON and Polishing. 

* __FALCON__
  * FALCON takes as input a set of subreads and produces a draft assembly as output. 
* __Polishing__
  * Polishing takes as input a set of subreads and the draft assembly produced by FALCON and produces a polished assembly. Polishing is in fact the same pipeline as Resequencing. It proceeds by aligning the subreads to the draft assembly with PBAlign, and then building a consensus sequence (the polished assembly) using variantCaller. The details of the Resequencing pipeline have been described [here](https://github.com/ben-lerch/Resequencing-3.0/blob/master/README.md).  

#Manual

##Running with SMRT Link

To run HGAP using SMRT Link, follow the usual steps for analysing data on SMRT Link. TODO: Link to document explaining SMRT Link.

##Running on the Command-Line with pbsmrtpipe
###Install pbsmrtpipe
pbsmrtpipe is a part of `smrtanalysis-3.0` package and will be installed
if `smrtanalysis-3.0` has been installed on your system. Or you can [download   pbsmrtpipe](https://github.com/PacificBiosciences/pbsmrtpipe) and [install](http://pbsmrtpipe.readthedocs.org/en/master/).
    
You can verify that pbsmrtpipe is running OK by:

    pbsmrtpipe --help

### Create a dataset
Now create an XML file from your subreads.

```
dataset create --type SubreadSet --generateIndices my.subreadset.xml subreads1.bam subreads2.bam ...
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

## Advanced Analysis Options

### SMRT Link/pbsmrtpipe Resequencing Options

You may modify advanced analysis parameters for Resequencing as described below via SMRTLink.

|   Module   |           Parameter (pbsmrtpipe_name)          |     Default      |  Explanation      |
| ---------- | ---------------------------------------------- | ----------------- | ---------------- |
|Alignment   | Min. accuracy (min_accuracy) | 70  | Minimum required alignment accuracy (percent) |
|Alignment   | Concordant alignment (concordant) | False | Maps subreads of a ZMW to the same genomic region |
|Alignment   | Min. length | 50 | Minimum required alignment length. |
|Alignment   | Use CCS | | Map the ccsSequence to the genome first, the align subreads to the interval that the CCS subreads mapped to. useccs: only maps subreads that span the length of the template. useccsall: maps all subreads. useccsdenovo: maps ccs only. |
|Alignment   | Hit policy (hit_policy) | randomBest  | Specify a policy for how to treat multiple hit random : selects a random hit. all : selects ll hits. allbest : selects all the best score hits. randombest: selects a random hit from all best score hits. leftmost : selects a hit which has the best score and the smallest mapping coordinate in any reference. Default value is randombest. |
|Alignment   | Algorithm options (algorithm_options) | -minMatch 12 -bestn 10 -minPctSimilarity 70.0  | List of space-separated arguments passed to blasr |
|Consensus   | Minimum confidence (min_confidence) | 40  | The minimum confidence for a variant call to be output to variants.gff |
|Consensus   | Diploid mode (diploid) | True  | Enable detection of heterozygous variants |
|Consensus   | Algorithm (algorithm) | quiver  | Algorithm name |
|Consensus   | Minimum coverage (min_coverage) | 5  | The minimum site coverage that must be acheived for variant calls and consensus to be calculated for a site |
|SMRT Pipe   | Minimum subread length | 0 | Minimum length of subreads to write to FASTA/FASTQ |  
|Assembly    | FALCON cfg overrides () |  | This is intended to allow support engineers to override the cfg which we will generate from other options. It is a semicolon-separated list of key=val pairs. Newlines are allowed but ignored. For more details on the available options see the [falcon manual](https://github.com/PacificBiosciences/FALCON/wiki/Manual). |
|Assembly    | Cores Max Ignored (HGAP_CoresMax_str) | 40  | IGNORE- Not currently used |
|Reports     |  Number of regions (absent) | 1000  | Desired number of genome regions in the summary statistics (used for guidance, not strict) |
|Reports     | Region size (absent) | 0  | If supplied, use a fixed genomic region size |
|Reports     | Force the number of regions (absent) | False  | If supplied, then try to use this number (max value=40000) of regions per reference, otherwise the coverage summary report will optimize the number of regions in the case of many references. Not compatible with a fixed region size. |

#Algorithm Modules

##FALCON

FALCON proceeds with the following steps to generate a draft genome assembly from a set of sequencing reads:

* Raw sub-reads overlapping for error correction
* Pre-assembly and error correction
* Overlapping detection of the error corrected reads
* Overlap filtering
* Constructing graph from overlaps
* Constructing contig from graph

####Raw sub-reads overlapping for error correction

FALCON employs a modified version of Gene Meyer's DAligner to compute overlaps between every subread with every other subread in the dataset. First, basic filtering is applied to the subreads. Then the subreads are converted to a Dazzler database and the database is split. Then the core DAligner algorithm is used to compute overlaps. 

####Pre-assembly and error correction

##Polishing
The polishing algorithm module is actually the same as Resequencing. Polishing proceeds by aligning raw subreads to the draft contigs. Then the alignments are used to refine the draft contigs, correcting miscalls to generate a polished assembly. Details of the Resequencing algorithms are available [here](https://github.com/ben-lerch/Resequencing-3.0/blob/master/README.md#algorithm-modules).
