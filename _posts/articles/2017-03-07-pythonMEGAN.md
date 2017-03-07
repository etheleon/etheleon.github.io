---
layout: post
title: "Metagenomics for the not so beginner"
excerpt: "Want to use MEGAN’s LCA algorithm in the server use the blast2lca tool."
categories: articles
tags: [bioinformatics, software, metagenomics]
author: wesley_goi
comments: true
share: true
modified: 2017-03-07
mathjax: true
image:
  feature: metabolicGraph.png
  thumb:  meta_thumb.jpg
  credit: Wesley GOI
  creditlink: http://etheleon.github.io
---

# A Python Wrapper for MEGAN blast2lca

From beginners to intermediate beginners in the field of Metagenomics.

> "Metagenomics (also referred to as environmental and community genomics) is the genomic analysis of microorganisms by direct extraction and cloning of DNA from an assemblage of microorganisms."

One tool you’ll quickly come across is [![MEGANimg](http://megan.informatik.uni-tuebingen.de/uploads/default/original/1X/c3b77ecaaa6f3b8f4c71d45f070a3a6b9952605b.png)](http://ab.inf.uni-tuebingen.de/software/megan/) from Daniel Huson’s Lab in Tubingen University.

If you take a look inside the tools directory of the installation, you’ll find a java executable called `blast2lca`. Below’s the printout from the command line tool.

```
SYNOPSIS
        Blast2LCA [options]
DESCRIPTION
        Applies the LCA alignment to reads and produce a taxonomic classification
OPTIONS
 Input  
        -i, --input [string]                 Input BLAST file. Mandatory option.
        -f, --format [string]                BLAST format. Default value: Unknown. Legal values: Unknown, DAA, BlastText, BlastXML, BlastTab, RapSearch2Aln, IlluminaReporter, RDPAssignmentDetails, RDPStandalone, Mothur, SAM, References_as_FastA
        -m, --mode [string]                  BLAST mode. Default value: Unknown. Legal values: Unknown, BlastN, BlastP, BlastX, Classifier
 Output  
        -o, --output [string]                Taxonomy output file. Default value: foo-taxonomy.txt.
        -ko, --keggOutput [string]           KEGG output file. Default value: foo-kegg.txt.
 Functional classification:
        -k, --kegg                           Map reads to KEGG KOs?. Default value: false.
 Output options:
        -sr, --showRanks                     Show taxonomic ranks. Default value: true.
        -oro, --officialRanksOnly            Report only taxa that have an official rank. Default value: true.
        -tid, --showTaxIds                   Report taxon ids rather than taxon names. Default value: false.
 Parameters  
        -ms, --minScore [number]             Min score. Default value: 50.0.
        -me, --maxExpected [number]          Max expected. Default value: 0.01.
        -top, --topPercent [number]          Top percent. Default value: 10.0.
        -mid, --minPercentIdentity [number]   Min percent identity. Default value: 0.0.
        -kr, --maxKeggPerRead [number]       Maximum number of KEGG assignments to report for a read. Default value: 4.
        +ktp, --applyTopPercentKegg          Apply top percent filter in KEGG KO analysis. Default value: true.
 Classification support:
        -tn, --parseTaxonNames               Parse taxon names. Default value: true.
        -g2t, --gi2taxa [string]             GI-to-Taxonomy mapping file. 
        -a2t, --acc2taxa [string]            Accession-to-Taxonomy mapping file. 
        -s2t, --syn2taxa [string]            Synonyms-to-Taxonomy mapping file. 
        -g2kegg, --gi2kegg [string]          GI-to-KEGG mapping file. 
        -a2kegg, --acc2kegg [string]         Accession-to-KEGG mapping file. Default value: ref2kegg.map.
        -s2kegg, --syn2kegg [string]         Synonyms-to-KEGG mapping file. 
 Other:
        -fwa, --firstWordIsAccession         First word in reference header is accession number. Default value: true.
        -atags, --accessionTags [string(s)]   List of accession tags. Default value(s): gb| ref|.
        -v, --verbose                        Echo commandline options and be verbose. Default value: false.
        -h, --help                           Show program usage and quit.
AUTHOR(s)
        Daniel H. Huson.
VERSION
        MEGAN Ultimate Edition (version 6.3.7, built 26 Apr 2016).
Copyright (C) 2016 Daniel H. Huson. This program comes with ABSOLUTELY NO WARRANTY..
```

It's been kindly included into the Community Edition of MEGAN (MEGAN CE) and its extremely valueable as a tool for one to basically access the core algorithms within MEGAN like the Lowest Common Ancestor algorithm and KEGG/COG/eggNOG functional assignment.

MEGAN's been around for awhile and in its 6th iteration it includes quite a few additions to deal with increasingly large datasets. However, it is still meant to be run on a desktop computer and if you need to run any huge jobs on multiple large sequencing projects, you'll be hardpressed.
Which is the precise reason for this tool being written in the first place.

Although, with the recent [MEGAN6 Ultimate Edition](https://computomics.com/index.php/megan.html), you'll find new server friendly capabilities but this will be outside of the blogpost.

## The wrapper

In this blogpost, I'll be sharing with you a [python2 wrapper](https://github.com/etheleon/megan), I've written around `blast2lca`.

So say for example you've got a huge number of samples which you would like to run some analysis and you're using a weak Macbook 12, but _alas_ you have access to a powerful univeristy headless server.
One option is to install MEGAN UE on your server and using the desktop client to connect with it but if you are not
in a position to pay for the license (which now includes KEGG) or have a older version of KEGG lying around somewhere in the filesystem, what should you do?

Use the `blast2lca` tool of course.

In the root directory of the github repo, you'll find a `fullPipeline` python script

```
usage: fullPipeline [-h] [--blast2lcaJAR JAR] [--gi2kegg GI2KEGG]
                    [--gi2taxid GI2TAXID] [--debug] [--verbose]
                    root sampledir sample DAA file taxonomy kegg

Command line wrapper of blast2lca and merging annotations

positional arguments:
  root                 the root directory
  sampledir            relative path from root directory to sample directory
  sample               sample name, could be same name as sample directory.
                       for combined ko and taxon
  DAA file             meganized DAA filename
  taxonomy             blast2lca taxonomy output filename - has to be in
                       taxIDs d__2
  kegg                 blast2lca ko output filename

optional arguments:
  -h, --help           show this help message and exit
  --blast2lcaJAR JAR   blast2lca jar file
  --gi2kegg GI2KEGG    gi2kegg mapping file
  --gi2taxid GI2TAXID  gi2taxid mapping file
  --debug              debug mode
  --verbose            to switch on verbose mode
```

It requires your samples to be stored in the following fashion

```
/projectDir
└── sampleDir/
     └── sample.daa
```

after running a command like this:

```
fullPipeline.py $PWD/projectDir sampleDirName SampleName inputSampleDAAfile blast2lca-tax-Output blast2lca-ko-Output
```


After which you'll get the outputs from blast2lca and with a merged file `sample-combined.txt`

```
/projectDir
└── sampleDirName/
     ├── blast2lca-tax-Output
     ├── blast2lca-ko-Output
     ├── sampleName-combined.txt
     └── inputSampleDAAfile.daa
```

In the `combined.txt` file, you'll find a table:

```
rank    ncbi-taxid  KEGG-ko #reads
phylum  67820   K00000  4
phylum  1224    K06937  1
phylum  1224    K00656  6
phylum  1224    K04564  2
phylum  1224    K06934  1
phylum  1224    K12524  24
phylum  1224    K00558  7
phylum  1224    K02674  1
phylum  1224    K06694  1
phylum  1224    K01785  12
...

...
species 1262910 K00033  1
species 1262911 K00000  525
species 35760   K00000  14
species 7462    K15421  1
species 7462    K00000  1
species 1262918 K00000  365
species 1262919 K02429  5
species 1262919 K07133  1
species 1262919 K03800  1
species 1262919 K00000  529
```

`0` in the `ncbi-taxid` and `K00000` in the KEGG-ko columns stands for unclassified.

Hope this helps!

## Reference

1. Handelsman, J (2004). Metagenomics: application of genomics to uncultured microorganisms. Microbiol. Mol. Biol. Rev., 68, 4:669-85.
2. Huson, D. H., Tappu, R., Bazinet, A. L., Xie, C., Cummings, M. P., Nieselt, K., & Williams, R. (2017). Fast and simple protein-alignment-guided assembly of orthologous gene families from microbiome sequencing reads. Microbiome, 5(1), 11. http://doi.org/10.1186/s40168-017-0233-2
