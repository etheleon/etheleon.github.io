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

# blast2lca++ a Python Wrapper for MEGAN blast2lca

Download now from: [https://github.com/etheleon/megan](https://github.com/etheleon/megan)

> "Metagenomics (also referred to as environmental and community genomics) is the genomic analysis of microorganisms by direct extraction and cloning of DNA from an assemblage of microorganisms."

From beginners to intermediate beginners in the field of Metagenomics.
One tool you’ll quickly come across is MEGAN from Daniel Huson’s Lab in Tubingen University.

[![MEGANimg](http://megan.informatik.uni-tuebingen.de/uploads/default/original/1X/c3b77ecaaa6f3b8f4c71d45f070a3a6b9952605b.png)](http://ab.inf.uni-tuebingen.de/software/megan/)

If you take a look inside the tools directory of the community edition (MEGAN CE) installation, you’ll find a bash executable called `blast2lca` [MEGAN6 Community Edition github repository](https://github.com/danielhuson/megan-ce/blob/master/tools/blast2lca) which taps into the java classes.

`blast2lca` is extremely valuable as a tool for one to basically access the core algorithms within MEGAN:
1. Lowest Common Ancestor algorithm and 
2. Functional (KEGG/COG/eggNOG) functional assignment.

MEGAN's been around for awhile with its [1st release](http://www.genome.org/cgi/reprint/gr.5969107v1.pdf) way back in (2007) and newest iteration [MEGAN6](http://www-ab.informatik.uni-tuebingen.de/software/megan6/) it includes quite a few additions to deal with increasingly large datasets. However, I would say it’s still meant for desktop users and if you need to run any huge jobs on multiple large sequencing projects, you'll be hardpressed.
Although, with the recent [MEGAN6 Ultimate Edition](https://computomics.com/index.php/megan.html), you'll find new server friendly capabilities but still difficult if one wants to incorporate MEGAN into the customised pipeline.
Discussions about MEGAN server will be outside of the blogpost, message me if you want to know more or directly with the authors.

## blast2lca++

In this blogpost, I'll be sharing with you a python wrapper [https://github.com/etheleon/megan](https://github.com/etheleon/megan), I've written around `blast2lca`. (At the time of writing we tested this with MEGAN6 community edition 6.6.0 from Dec 2016).

Say for example you've got a huge number of samples which you would like to run some analysis and you're using a weak Macbook 12, but _alas_ you have access to a powerful univeristy headless server.

![lotsOfWork](http://weknowmemes.com/wp-content/uploads/2013/03/i-have-a-lot-of-work-to-do-oh-well-comic.jpg)

One option is to install MEGAN server (Ultimate edition) run the LCA and functional binning algos there and analyses it via your desktop when you connect with the server.
For someone who writes scripts, its a truly WTH moment. Luckily there’s `blast2lca` kindly provided by the author.

However there’s still several steps which are not clear,
1. Combine Annotations - How does one combine KO and TAXONOMIC annotations such that we will have a combined annotation for each query (be it short read/long read/contig).
2. gi2ko mapping file generator - KEGG annotations. In the community edition, the tool to generate the mapping file (gi kegg) is not included like in the ultimate edition. What if you’re not in a position to pay for the ultimate edition license (which includes with it the KEGG database) or have a older version of KEGG lying around somewhere in the server, what should you do? (NCBI has recently done away with the GI and I’ll update this in the future)  
3. complete pipeline from blast to combined output  - How to go all the way from the tabbed blast output to the KO and taxonomy combined output mentioned above.

Use `blast2lca++` tool of course!

![unicorn](http://img.memecdn.com/unicorn-farting-rainbows_o_1498739.jpg)

### Combine Annotations

In the root directory of the github [repo](https://github.com/etheleon/megan), you'll find a `parseMEGAN` python script.

It requires data be arranged in the following manner:

```
/projectDir
└── sampleDir/
     └── sample.daa / sample.m8 (tabbed blast)
```

You’ll be asked to specify the locations of the mapping files (KEGG and taxonomy as well as the path to the executable)

```
parseMEGAN $PROJECTDIR $SAMPLEDIR $SAMPLENAME taxOutput koOutput
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

In the `sample-combined.txt` file, you'll find a table:

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

### Full pipeline

```
fullPipeline $PROJECTDIR $SAMPLEDIR $SAMPLENAME $INPUTFILE taxOutput koOutput --blast2lca <path 2 the blast2lca script> --gi2tax <path to the taxonomy mapping file> --gi2kegg <path to the KEGG mapping file>
```

The `fullPipeline` script will take a `.m8` (tabbed blast)  file or meganised `.DAA` file as input (checked via regex `.m8` or `.daa`) and carry out the taxonomic and functional (KEGG) annotation and take the outputs and generate a combined output

![end](https://i.imgflip.com/123oks.jpg)

### gi2ko mapping file generator

Although MEGAN UE provides a KEGG mapper generating tool (not included with MEGAN CE), it doesnt take into consideration how NCBI has assigned a unique GI to each representative sequence in the non-redundant database (NCBI NR) under which are “duplicate” sequence GIs and ref IDs
when blast or diamond does the alignment it only returns the former and the rest. It makes the kegg to gi mapping irrelevent.

We’ve separately included in the tools folder of the python package the [ref2kegg.go nr-gi to kegg ortholog KO mapping file generator](https://github.com/etheleon/MEGAN/blob/master/tools/ref2kegg.go). The output could be to wrapper via the `--gi2kegg` flag. At the time of writing the parser is written in golang (typed lang) to increase the speed of parsing the NR fa file.



Hope this helps anyone doing any customised pipeline with MEGAN6!
Personally I feel strongly that MEGAN has now a open source version via MEGAN CE.

## Reference

1. Handelsman, J (2004). Metagenomics: application of genomics to uncultured microorganisms. Microbiol. Mol. Biol. Rev., 68, 4:669-85.
2. Huson, D. H., Tappu, R., Bazinet, A. L., Xie, C., Cummings, M. P., Nieselt, K., & Williams, R. (2017). Fast and simple protein-alignment-guided assembly of orthologous gene families from microbiome sequencing reads. Microbiome, 5(1), 11. http://doi.org/10.1186/s40168-017-0233-2
