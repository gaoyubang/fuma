# FuMa (Fusion Matcher) #
## Introduction ##

FuMa (Fusion Matcher) matches predicted fusion events (both genomic and transcriptomic) according to chromosomal location or assocatiated gene annotation(s) where the latter should not be genome build specific. The FuMa project currently supports input files from:

+	ChimeraScan<sup>[1]</sup>
+	Complete Genomics<sup>[2]</sup>
+	DeFuse<sup>[3]</sup>
+	FusionCatcher<sup>[4]</sup>
+	(Trinity ->) GMAP<sup>[5]</sup>
+	STAR<sup>[6]</sup>
+	Tophat_Fusion<sup>[7]</sup>

Because RNA-Sequencing deals with samples that may have undergone splicing, reads may split up because of biological processes. If a fusion event takes place, the same thing may happen. Therefore we hypothesize that using spanning read distances may be unreliable, because there are known introns of > 100kb. Therefore, FuMa translates the breakpoint to gene names, and only overlaps breakpoints with the same gene-name(s).


## Installation ##
### Ubuntu ###
We advise you to run the following commands to install FuMa on Ubuntu:

	sudo apt-get install build-essential python-dev git python-pip
	sudo pip uninstall fuma
	
	git clone https://github.com/yhoogstrate/fuma.git
	
	cd fuma
	
	python setup.py build
	python setup.py test
	sudo python setup.py install
	
	fuma --version

The FuMa package has not been tested on different platforms, but the installation procedure should be similar.

### Galaxy ###

Because usage of FuMa via the command line can be experienced as complicated, we also provide FuMa as galaxy (http://galaxyproject.org/) package. The tool shed repository is:

https://testtoolshed.g2.bx.psu.edu/view/yhoogstrate/fuma

In order to install FuMa via Galaxy, you have to make sure you have the test-toolsshed (https://testtoolshed.g2.bx.psu.edu/) in your tool_sheds_conf.xml or config/tool_sheds_conf.xml. To install, follow the procedure via the galaxy admin panel. We have made FuMa publicly available at the following galaxy instance:

[http://galaxy-demo.ctmm-trait.nl/](http://galaxy-demo.ctmm-trait.nl/)

## Command Line Usage ##
To run FuMa via the command line, each dataset should be given as a separate file. Similarly, the corresponding gene reference(s) have to be provided for and linked to each experiment separately. Also, the file format has to be specified for each dataset. As you can imagine, this is a rather complex information structure and therefore, unfortunately, the command line arguments are not simple either.

The command line usage of FuMa is:

	usage: fuma [-h] [-V] [--verbose]
	            [-a [ADD_GENE_ANNOTATION [ADD_GENE_ANNOTATION ...]]] -s ADD_SAMPLE
	            [ADD_SAMPLE ...]
	            [-l [LINK_SAMPLE_TO_ANNOTATION [LINK_SAMPLE_TO_ANNOTATION ...]]]
	            [-f {summary,list,extensive}] [-o OUTPUT]
	
	optional arguments:
	  -h, --help            show this help message and exit
	  -V, --version         show program's version number and exit
	  --formats             show accepted dataset formats
	  --verbose             increase output verbosity
	  -a [ADD_GENE_ANNOTATION [ADD_GENE_ANNOTATION ...]], --add-gene-annotation [ADD_GENE_ANNOTATION [ADD_GENE_ANNOTATION ...]]
	                        annotation_alias:filename * file in BED format
	  -s ADD_SAMPLE [ADD_SAMPLE ...], --add-sample ADD_SAMPLE [ADD_SAMPLE ...]
	                        sample_alias:type:filename
	  -l [LINK_SAMPLE_TO_ANNOTATION [LINK_SAMPLE_TO_ANNOTATION ...]], --link-sample-to-annotation [LINK_SAMPLE_TO_ANNOTATION [LINK_SAMPLE_TO_ANNOTATION ...]]
	                        sample_alias:annotation_alias
	  -f {summary,list,extensive}, --format {summary,list,extensive}
	                        Output-format
	  -o OUTPUT, --output OUTPUT
	                        output filename; '-' for stdout
	
	For more info please visit:
	<https://github.com/yhoogstrate/fuma>


### -a ADD_GENE_ANNOTATION ###
Gene annotations have to be provided in a simple tab-delimited file of the following syntax:

	chr1   100000000  120000000  GeneNameA
	chr2   100000000  120000000  GeneNameB
	chr21  100000000  120000000  GeneNameC
	chr22  100000000  120000000  GeneNameD
	chrX   140000000  160000000  GeneNameX
	chrY   140000000  160000000  GeneNameY

This format is compatible with the BED format (https://genome.ucsc.edu/FAQ/FAQformat.html), but requires a 4th column and requires it to contain unique gene names. Additional columns are allowed, but are nowhere taken into account.
**Do not provide BED files that describe one exon per line, but provide BED files that describe one gene per line instead.**
This is because gene annotations often contain a few duplicates on the same chromosome. When genes are merged back together on the basis of the gene names, duplicates on the same chromosome that span a large distance may introduce large uncertainty. On the other hand, **if you only want to apply matching only in exon regions, you SHOULD use BED files with one exon per line.**

In FuMa the gene annotation argument is provided in two parts, the *alias* followed by the *filename*, separated with a colon:

	-f "hg19:somefile.bed"

In this case the alias (*hg19*) of the bedfile, will later be used to link it to datasets.
In case you want multiple references, you can provide arguments delimited with whitespaces:

	-f "hg18:somefile_hg18.bed" "hg19:somefile_hg19.bed"

### -s ADD_SAMPLE  ###
To provide FuMa a fusion gene detection experiment, it should be provided with the "-s" argument which should follow the following syntax:

*sample_alias*:*format*:*filename*

The *sample_alias* will be used for two things: (1) as column header and alias in the final output and (2) later on to link the references to the samples. The *format* is the file format in which the fusion genes are described. Note some tools have multiple formats, since their interim output can also be loaded in FuMa.

#### Formats ####

FuMa supports the following file formats:

| Tools              | File                  | Format string
|:-------------------|:----------------------|:-------------
| ChimeraScan        | chimeras.bedpe        | chimerascan  
| Complete Genomics  | highConfidenceJu*.tsv | complete-genomics
| Complete Genomics  | allJunctionsBeta*.tsv | complete-genomics
| DeFuse             | results.txt           | defuse
| DeFuse             | results.classify.txt  | defuse
| DeFuse             | results.filtered.txt  | defuse
| Fusion Catcher     | final-list_cand*.txt  | fusion-catcher_final
| FusionMap          |                       | fusionmap
| Trinity + GMAP     |                       | trinity-gmap
| OncoFuse           |                       | oncofuse
| RNA STAR           | Chimeric.out.junction | rna-star_chimeric
| TopHat Fusion pre  | fusions.out           | tophat-fusion_pre
| TopHat Fusion post | potential_fusion.txt  | tophat-fusion_post_potential_fusion
| TopHat Fusion post | result.txt            | tophat-fusion_post_result

Or run the following command line argument to get an overview of the versions at the command line:

	fuma --formats

### -l LINK_SAMPLE_TO_ANNOTATION ###

Each dataset must be annotated with only one dataset. This can be achieved using the following argument syntax:

*sample_alias*:*annotation_alias*

In case you have a particular same *s* and a reference *ref*, you can link *s* to *ref* as follows:

	-l "s:ref"

In case you have two samples, one on *ref1* and one on *ref2*, you can provide it as follows:

	-l "defuse_hg18:hg18" "chimerascan_hg19:hg19"

### -f output format ###

FuMa has the built-in option for multiple output formats. The most straight-forward format is the '*list*' output format which contains per (matched) fusion gene, for each matching tool, the genomic locations and identifier(s). In the following example we have three fusion genes; one detected by TopHat fusion, one by STAR and one by BOTH. The corresponding output in '*list*' format would be:

| Left Genes | Right Genes | STAR             | TopHat Fusion
|:-----------|:------------|:-----------------|:-------------
| FOO1       | BAR1        | UID_A=chr1:12-34 | 
| FOO2       | BAR2        |                  | TID_A=chr4:66-77 
| DOX1       | BOX5        | UID_B=chr5:85-95 | TID_B=chr5:88-99 

The tools may predict multiple fusion genes with the same left- and right genes, which FuMa will consider as duplicates. In case we observe a duplicate, we simply print BOTH identifiers delimited with a comma into a cell such that duplicate entries can still be found back in the output:

| Left Genes | Right Genes | FusionMap
|:-----------|:------------|:---------
| FOO1       | BAR1        | UID_A=chr1:12-34,UID_B=chr1:12-34

When a breakpoint location spans multiple gene annotations, the genes in the column are delimited with a colon:

| Left Genes | Right Genes | OncoFuse
|:-----------|:------------|:---------
| FOO1:FOO2  | BAR1        | UID_A=chr1:12-34

The Galaxy wrapper has the option to replace the columns to TRUE or FALSE depending on whether a match was found or not.

The output format '*extensive*' is a Complete Genomics data formatted file that only contains those fusion genes that have at least one match. This format is in particular useful if the output of one run needs to be used for another run.

The output format '*summary*' is a set of tables that contains the numbers of detected matches per dataset combination.

### Example 01: one sample, two tools ###

Imagine we have run sample FOO with Defuse and ChimeraScan, on the same reference genome (hg19).
The genes 

	fuma \
	    -a  "hg19:genes_hg19.bed" \
	    \
	    -s  "chimerascan:chimerascan:FOO_chimerascan/chimeras.bedpe" \
	        "defuse:defuse:FOO_defuse/results.tsv" \
	    -l  "chimerascan:hg19" \
	        "defuse:hg19" \
	    -f  "list" \
	    -o  "chimerascan_defuse_overlap.txt"

### Example 02: one sample, one tool, different reference genomes ###

If we want to compare the differences between analyses on different genome builds, we can simply add an experiment twice but change the reference.
Imagine we have run a sample with TopHat-Fusion on reference genomes hg18 and hg19, we can run FuMa as follows:

	fuma \
	    -a  "hg18:genes_hg18.bed" \
	    -a  "hg19:genes_hg19.bed" \
	    \
	    -s  "thf_hg18:Tophat-Fusion Post result:thf_hg18/result.txt" \
	        "thf_hg19:Tophat-Fusion Post result:thf_hg19/result.txt" \
	    -l  "thf_hg18:hg18" \
	        "thf_hg18:hg19" \
	    -f  "list" \
	    -o  "thf_hg18_hg19_overlap.txt"

### Example 03: Edgren dataset as part of Chimera supplement ###
The publicly available data from the Edgren dataset has been performed on FusionMap, ChimeraScan and DeFuse as proof of concept data for the Chimera package (Edgren et al., 2011; Beccuti et al., 2014). To obtain these result you should run the following command at the command line:

	wget http://www.bioconductor.org/packages/release/bioc/src/contrib/chimera_1.10.0.tar.gz
	tar -xzf chimera_1.10.0.tar.gz

Within the source of the chimera package, you can find the files with the following command line command:

	find . –type f | grep -i -E "Edgr[e]{1,2}n"

Please check whether the output is identical to:

	./chimera/inst/examples/Edgreen_fm.txt
	./chimera/inst/examples/edgren.stat.detection.txt
	./chimera/inst/examples/Edgren_df.tsv
	./chimera/inst/examples/Edgren_cs.txt
	./chimera/inst/examples/Edgren_true.positives.txt

To get a gene reference and the True positivies with genomic coordinates, run at the command line:

	wget https://testtoolshed.g2.bx.psu.edu/repos/yhoogstrate/fuma/raw-file/tip/test-data/refseq_genes_hg19.bed
	wget https://testtoolshed.g2.bx.psu.edu/repos/yhoogstrate/fuma/raw-file/tip/test-data/edgren_tp.txt

We can proceed with FuMa by running at the command line:

	edir="./chimera/inst/examples/"
	fuma \
	    -a  "hg19:refseq_genes_hg19.bed" \
	    \
	    -s  "chimerascan:chimerascan:"$edir"Edgren_cs.txt" \
	        "defuse:defuse:"$edir"Edgren_df.tsv" \
	        "fusionmap:fusionmap:"$edir"Edgreen_fm.txt" \
	        "edgren_TP:fusionmap:edgren_tp.txt" \
	    -l  "fusionmap:hg19" \
	        "defuse:hg19" \
	        "chimerascan:hg19" \
	        "edgren_TP:hg19" \
	    -f  "list" \
	    -o  "edgren_fuma_list.txt"
	
	fuma-list-to-boolean-list \
	-o "edgren_fuma_booleanlist.txt" \
	   "edgren_fuma_list.txt"

To find all fusion genes present in 3 or more datasets, run at the commandline:

	grep -E $'TRUE\tTRUE\tTRUE' "edgren_fuma_booleanlist.txt"

This returns a list of 9 fusion genes, in which we can find the following line:

	NM_018837:NM_198596:NM_001161841	NM_006420	TRUE	TRUE	TRUE	TRUE

This line corresponds to the RefSeq IDs of ARFGEF2-SULF2. 

## References ##
<sup>[1]</sup> **ChimeraScan**

Publication: [http://dx.doi.org/10.1093/bioinformatics/btr467](http://dx.doi.org/10.1093/bioinformatics/btr467)

Code: [https://code.google.com/p/chimerascan/](https://code.google.com/p/chimerascan/)

<sup>[2]</sup> **CompleteGenomics** (DNA)

Publication: [http://dx.doi.org/10.1089/cmb.2011.0201](http://dx.doi.org/10.1089/cmb.2011.0201)

<sup>[3]</sup> **DeFuse**

Publication: [http://dx.doi.org/10.1371/journal.pcbi.1001138](http://dx.doi.org/10.1371/journal.pcbi.1001138)

Code: [http://sourceforge.net/projects/defuse/](http://sourceforge.net/projects/defuse/)

<sup>[4]</sup> **FusionCatcher**

Publication: [http://dx.doi.org/10.1101/011650](http://dx.doi.org/10.1101/011650)

Code: [https://code.google.com/p/fusioncatcher](https://code.google.com/p/fusioncatcher)

<sup>[5]</sup> **GMAP**

Publication: [http://dx.doi.org/bioinformatics/bti310](http://dx.doi.org/bioinformatics/bti310)

Code: [http://research-pub.gene.com/gmap/](http://research-pub.gene.com/gmap/)

<sup>[6]</sup> **STAR**

Publication: [http://dx.doi.org/10.1093/bioinformatics/bts635](http://dx.doi.org/10.1093/bioinformatics/bts635)

Code: [https://code.google.com/p/rna-star/](https://code.google.com/p/rna-star/)

<sup>[7]</sup> **TopHat Fusion**

Publication: [http://dx.doi.org/10.1186/gb-2011-12-8-r72](http://dx.doi.org/10.1186/gb-2011-12-8-r72)

Code: [http://ccb.jhu.edu/software/tophat/fusion_index.html](http://ccb.jhu.edu/software/tophat/fusion_index.html)

