---
title: "The DeepBlue epigenomic data server - R package"
author: "Felipe Albrecht"
date: "`r Sys.Date()`"
output:
  html_document:
    toc: true
    number_of_sections: true
vignette: >
  %\VignetteIndexEntry{The DeepBlue epigenomic data server - R package}
  %\VignetteDepends{DeepBlueR,ggplot2,Gviz,tidyr}
  %\VignettePackage{DeepBlueR}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---

![DeepBlueR logo](logo.png)

# Introduction

The DeepBlue Epigenomic Data Server is a tool that allows researchers to access
data from various epigenomic mapping consortia such as DEEP, BLUEPRINT or
ROADMAP. DeepBlue can be accessed through a
[web interface](http://deepblue.mpi-inf.mpg.de) or programmatically via its
[API](http://deepblue.mpi-inf.mpg.de/api.php). The usage of the API is
documented with [examples](http://deepblue.mpi-inf.mpg.de/examples.php),
[use cases](http://deepblue.mpi-inf.mpg.de/use_cases.php), and a
[user manual](http://deepblue.mpi-inf.mpg.de/manual). While the description of
the API is language agnostic, the examples and use cases shown online are
focused on the python language. However, the R package presented here also
enables access to the DeepBlue API directly within the R statistical environment
and provides convenient functionality for triggering operations on the DeepBlue
server as well as for data retrievel using R functions. In the following, we
give a brief introduction to the package and subsequently show how python
examples from the online documentation can be reproduced with it.

## What is DeepBlue ?
A wealth of epigenomic data has been collected over the past decade by large
epigenomic mapping consortia. Event though most of these data are publicly
available, the task of identifiying, downloading and processing data from
various experiments is challenging. Recognizing that these tedious steps
need to be tackled programmatically, we developed the DeepBlue epigenomic
data server. Epigenome data from the different epigenome mapping consortia
are accessible with standardized metadata. An experiment is the most important
entity in DeepBlue and typically encompasses a single file (usually a bed or wig
file) with a set of mandatory metadata: name, genome assembly, epigenetic mark,
biosource, sample, technique, and project. For the sake of organization, all
metadata fields are part of controlled vocabularies, some of which are imported
from ontologies (CL, EFO, and UBERON, to name a few). DeepBlue also contains
annotations, i.e. auxiliary data that is helpful in epigenomic analysis, such as,
for example, CpG Islands, promoter regions, and genes. DeepBlue provides
different types of commands, such as listing and searching commands as well as
commands for data retrieval. A typical work-flow for the latter is to select,
filter, transform, and finally download the selected data. For a more thorough
description of DeepBlue we refer to the 
[DeepBlue publication](http://dx.doi.org/10.1093/nar/gkw211) in the 2016 NAR
webserver issue. If you find DeepBlue useful and use it in your project consider 
citing this paper.


Important note: With the exception of data aggregation tasks,
DeepBlue does not alter the imported data, i.e. it remains exactly as
provided by the epigenome mapping consortia.

# Getting started
## Installation

Automatic installation of DeepBlueR and its companion packages can be
performed through the Bioconductor installer:

```{r, eval = FALSE, echo=TRUE, warning=FALSE, message=FALSE}
## try http:// if https:// URLs are not supported
source("https://bioconductor.org/biocLite.R")
biocLite("DeepBlueR")
```

The package name is ```DeepBlue``` and it can be loaded via:
```{r, echo=FALSE, warning=FALSE, message=FALSE, error=FALSE}
library(DeepBlueR)
```

You can test your installation by saying hello to the DeepBlue server:
```{r, echo=TRUE, eval = FALSE, warning=FALSE, message=FALSE}
deepblue_info("me")
```

## Overview of DeepBlue commands

DeepBlue provides a comprehensive programmatic interface for finding, selecting,
filtering, summarizing and downloading annotated genomic region sets. Downloaded
 region sets are stored using the GenomicRanges R package, which allows for
  downloaded region sets to be further processed, visualized and analyzed with
  existing R packages such as LOLA or GViz.

A list of all commands available by DeepBlue is provided in its
 [API page](http://deepblue.mpi-inf.mpg.de/api.php). The vast majority of
 these commands is also available through this R package and can be listed as follows:

 ```
 help(package="DeepBlueR")
 ```

For reference, following are listed the most used DeepBlue commands. Note that
 each command has the prefix 'deepblue_*', e.g. deepblue_select_genes.

| Category        | Command               | Description                        |
|-----------------|-----------------------|------------------------------------|
| Information     | info                  | Information about an entity        |
| List and search | list_genomes          | List registered genomes            |
|                 | list_biosources       | List registered biosources         |
|                 | list_samples          | List registered samples            |
|                 | list_epigenetic marks | List registered epigenetic marks   |
|                 | list_experiments      | List available experiments         |
|                 | list_annotations      | List available annotations         |
|                 | search                | Perform a full-text search         |
| Selection       | select_regions        | Select regions from experiments    |
|                 | select_experiments    | Select regions from experiments    |
|                 | select_annotations    | Select regions from annotations    |
|                 | select_genes          | Select genes as regions            |
|                 | tiling_regions        | Generate tiling regions            |
|                 | input_regions         | Upload and use a small region-set  |
| Operation       | aggregate             | Aggregate and summarize regions    |
|                 | filter_regions        | Filter regions by theirs attributes|
|                 | flank                 | Generate flanking regions          |
|                 | intersection          | Filter overlapping regions         |
|                 | merge_queries         | Merge two regions set              |
| Result          | count_regions         | Count selected regions             |
|                 | score_matrix          | Request a score matrix             |
|                 | get_regions           | Request the selected regions       |
| Request         | get_request data      | Obtain the requested data          |


This package provides a set of functions for facilitate the API usage:

| Category | Command               | Description                             |
|----------|-----------------------|-----------------------------------------|
| Request  | batch_export_results  | Download the data a set of requests     |
|          | download_request_data | Download and convert the requested data |



# DeepBlue usage examples

In the following we give a number of increasingly complex examples illustrating
what DeepBlue can achieve for you in your epigenomic data analysis work-flow.
We go beyond the online description of these examples by showing how the
retrieved information can be further used in R.

One of the first tasks in DeepBlue is listing the data. This can be achieved
in three ways:

* Using full-text search with the ```search``` command
* Listing the available data with the ```list_*``` commands
* Using the companion [DeepBlue web interface](http://deepblue.mpi-inf.mpg.de/)
site for listing the data

### Full-text search

In this example, we use the command ```search``` to find experiments that
contain the texts H3k27AC, blood, and peaks in their metadata. We put the names
 in single quotes to show that these names must be in the metadata.

```{r, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
# We are selecting the experiments with terms 'H3k27AC', 'blood', and
# 'peak' in the metadata.
experiments_found = deepblue_search(
    keyword="'H3k27AC' 'blood' 'peak'", type="experiments")

custom_table = do.call("rbind", lapply(experiments_found, function(experiment){
  experiment_id = experiment[[1]]
  # Obtain the information about the experiment_id
  info = deepblue_info(experiment_id)
  experiment_info = info[[1]]
  # Print the experiment name, project, biosource, and epigenetic mark.
  with(experiment_info, { data.frame(name = name, project = project,
    biosource = sample_info$biosource_name, epigenetic_mark = epigenetic_mark)
      })
}))
  head(custom_table)
```

### Listing experiments

We use the ```list_experiments``` command to list all experiments with the
corresponding values in theirs metadata.

```{r, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
experiments = deepblue_list_experiments(type="peaks", epigenetic_mark="H3K4me3",
    biosource=c("inflammatory macrophage", "macrophage"),
    project="BLUEPRINT Epigenome")
do.call("rbind", lapply(experiments,
    function(x){ data.frame(id = x[[1]], experiment_name = x[[2]])}))
```

### Accessing the extra-metadata
The extra-metadata is important because it contains information that is not
stored in the mandatory metadata fields. We use the info command to access an
experiment's metadata and its extra-metadata fields.
The following example prints the ```file_url``` attribute that it is contained
in the data imported from the ENCODE project.

```{r, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
info = deepblue_info("e30000")
print(info[[1]]$extra_metadata$file_url)
```

### Select epigenomic data
We use the ```select_experiments``` command to select all genomic regions from
the two informed experiments. We use the ```count_regions``` command with
the ```query_id``` value returned by the ```select_experiments```.
The ```count_regions``` command is asynchronous. It means that the user receives
a ```request_id``` and should check the status of this request. The specific
DeepBlueR package command ```download_request_data``` wait for the processing
finishes in the server, download the data, and convert to a GRanges object.


```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
query_id = deepblue_select_experiments(
    experiment_name=c("BL-2_c01.ERX297416.H3K27ac.bwa.GRCh38.20150527.bed",
        "S008SGH1.ERX406923.H3K27ac.bwa.GRCh38.20150728.bed"))
# Count how many regions where selected
request_id = deepblue_count_regions(query_id=query_id)
# Download the request data as soon as processing is finished
requested_data = deepblue_download_request_data(request_id=request_id)
print(paste("The selected experiments have", requested_data, "regions."))
```

### Output with desired columns
We use the ```select_experiments``` command to select the genomic regions from
the experiments that are in the chromosome 1, position 0 to 50,000,000.

We use the ```get_regions``` command with the ```query_id``` value returned by
the ```select_experiments``` and the desired file columns.
The columns @NAME and @BIOSOURCE include the experiment name and the experiment
biosource in the row output.

The ```get_regions``` command is asynchronous. It means that the user receives a
```request_id``` and should check the status of this request.
The specific DeepBlueR package command ```download_request_data``` wait for the
processing finishes in the server, download the data, and convert to a GRanges
object.

```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
query_id = deepblue_select_experiments (
    experiment_name = c("BL-2_c01.ERX297416.H3K27ac.bwa.GRCh38.20150527.bed",
        "S008SGH1.ERX406923.H3K27ac.bwa.GRCh38.20150728.bed"),
        chromosome="chr1", start=0, end=50000000)

# Retrieve the experiments data (The @NAME meta-column is used to include the
# experiment name and @BIOSOURCE for experiment's biosource
request_id = deepblue_get_regions(query_id=query_id,
    output_format="CHROMOSOME,START,END,SIGNAL_VALUE,PEAK,@NAME,@BIOSOURCE")
regions = deepblue_download_request_data(request_id=request_id)
regions
```

### Filter epigenomic data by metadata
We use the ```list_samples``` command to obtain all samples from the biosource
myeloid cell from the BLUEPRINT project. The ```list_samples``` returns a list
of samples with their IDs and content.

We extract the IDs from this list and use it in the ```select_regions```
command.

The ```select_regions``` command selects the genomic regions that are in the
chromosome 1, position 0 to 50,000 of all experiments that have the given
samples IDs.

Then, we use the ```get_regions``` command with the parameters: ```query_id```
returned by the ```select_regions``` and the desired file columns.
The columns @NAME, SAMPLE_ID, and @BIOSOURCE include the experiment name,
the sample ID, and the experiment biosource in the row output.

The ```get_regions``` command is asynchronous. It means that the user receives
a ```request_id``` and should check the status of this request. The specific
DeepBlueR package command ```download_request_data``` wait for the processing
finishes in the server, download the data, and convert to a GRanges object.

```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
samples = deepblue_list_samples(
    biosource="myeloid cell",
    extra_metadata = list("source" = "BLUEPRINT Epigenome"))
samples_ids = deepblue_extract_ids(samples)
query_id = deepblue_select_regions(genome="GRCh38", sample=samples_ids,
    chromosome="chr1", start=0, end=50000)
request_id = deepblue_get_regions(query_id=query_id,
    output_format="CHROMOSOME,START,END,@NAME,@SAMPLE_ID,@BIOSOURCE")
regions = deepblue_download_request_data(request_id=request_id)
head(regions,1)
```

### Filter epigenomic data by region attributes
We use the ```select_experiments``` command for selecting the genomic
regions from the experiments that are in the
chromosome 1, position 0 to 50,000,000.

We filter the genomic regions that have the value of the column SIGNAL_VALUE
larger than 10.

We filter the genomic regions that have the value of the column PEAK larger
than 1000.

Then, we use the ```get_regions``` command with the parameters: ```query_id```
returned by the select_experiments and the desired file columns.
The columns @NAME and @BIOSOURCE include the experiment name and the experiment
biosource in the row output.

The ```get_regions``` command is asynchronous. It means that the user receives
a ```request_id``` and should check the status of this request. The specific
DeepBlueR package command ```download_request_data``` wait for the processing
finishes in the server, download the data, and convert to a GRanges object.

```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
query_id = deepblue_select_experiments(
    experiment_name = c("BL-2_c01.ERX297416.H3K27ac.bwa.GRCh38.20150527.bed",
        "S008SGH1.ERX406923.H3K27ac.bwa.GRCh38.20150728.bed"),
    chromosome="chr1", start=0, end=50000000)
query_id_filter_signal = deepblue_filter_regions(
    query_id=query_id, field="SIGNAL_VALUE", operation=">",
    value="10", type="number")
query_id_filters = deepblue_filter_regions(
    query_id=query_id_filter_signal, field="PEAK", operation=">",
    value="1000", type="number")
request_id = deepblue_get_regions(query_id=query_id_filters,
    output_format="CHROMOSOME,START,END,SIGNAL_VALUE,PEAK,@NAME,@BIOSOURCE")
regions = deepblue_download_request_data(request_id=request_id)
regions
```

### Find overlapping regions
We use the ```select_experiments``` command to select the genomic regions
from the experiments that are in the chromosome 1, position 0 to 50,000,000.

We use the ```select_annotations``` command to select the genomic regions
in the chromosome 1 of the annotation promoters of the genome assembly GRCh38.

The command intersection filters all regions of the ```query_id``` that
overlap with at least one ```promoters_id``` region.

We use the ```get_regions``` command with the parameters: ```query_id```
returned by the ```select_experiments``` and the desired file columns.
The columns @NAME and @BIOSOURCE include the experiment name and the experiment
biosource in the row output.

The ```get_regions``` command is asynchronous. It means that the user receives
a ```request_id``` and should check the status of this request. The specific
DeepBlueR package command ```download_request_data``` wait for the processing
finishes in the server, download the data, and convert to a GRanges object.

```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
query_id = deepblue_select_experiments(
    experiment_name = c("BL-2_c01.ERX297416.H3K27ac.bwa.GRCh38.20150527.bed",
        "S008SGH1.ERX406923.H3K27ac.bwa.GRCh38.20150728.bed"),
    chromosome="chr1", start=0, end=50000000)
promoters_id = deepblue_select_annotations(annotation_name="promoters",
    genome="GRCh38", chromosome="chr1")
intersect_id = deepblue_intersection(
    query_data_id=query_id, query_filter_id=promoters_id)
request_id = deepblue_get_regions(
    query_id=intersect_id,
    output_format="CHROMOSOME,START,END,SIGNAL_VALUE,PEAK,@NAME,@BIOSOURCE")
regions = deepblue_download_request_data(request_id=request_id)
regions
```

```{r, message=FALSE, error=FALSE, warning=FALSE}
library(Gviz)
atrack <- AnnotationTrack(regions, name = "Overlapping regions",
    group = regions$`@BIOSOURCE`, genome="hg38")
gtrack <- GenomeAxisTrack()
itrack <- IdeogramTrack(genome = "hg38", chromosome = "chr1")
plotTracks(list(itrack, atrack, gtrack), groupAnnotation="group", fontsize=18,
           background.panel = "#FFFEDB", background.title = "darkblue")
```


### Retrieve DNA sequences
We use the ```select_experiments``` command to select the genomic regions from
the experiments that are in the chromosome 1, position 0 to 50,000,000.

We filter the genomic regions that have the value of the
column SIGNAL_VALUE higher than 10.

We filter the genomic regions that have the value of the
column PEAK higher than 1000.

The meta-column @LENGTH contains the genomic region length,
and we filter the genomic regions where this value is smaller than 2,000.

We use the ```get_regions``` with the ```query_id``` returned by
the ```select_experiments``` and the desired file columns. In this case,
we use the meta-column @SEQUENCE, that includes the DNA Sequence in the
genomic region output.

The ```get_regions``` command is asynchronous. It means that the user receives
a ```request_id``` and should check the status of this request.
The specific DeepBlueR package command ```download_request_data``` wait for the
processing finishes in the server, download the data, and convert to
a GRanges object.


```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
query_id = deepblue_select_experiments(
    experiment_name = c("BL-2_c01.ERX297416.H3K27ac.bwa.GRCh38.20150527.bed",
        "S008SGH1.ERX406923.H3K27ac.bwa.GRCh38.20150728.bed"),
    chromosome="chr1", start=0, end=50000000)
query_id_filter_signal = deepblue_filter_regions(query_id=query_id,
    field="SIGNAL_VALUE", operation=">", value="10", type="number")
query_id_filters = deepblue_filter_regions(query_id=query_id_filter_signal,
    field="PEAK", operation=">", value="1000", type="number")
query_id_filter_length = deepblue_filter_regions (query_id=query_id_filters,
    field="@LENGTH", operation="<", value="2000", type="number")
request_id = deepblue_get_regions(query_id=query_id_filter_length,
    output_format="CHROMOSOME,START,END,@NAME,@BIOSOURCE,@LENGTH,@SEQUENCE")
regions = deepblue_download_request_data(request_id=request_id)
head(regions, 1)
```

### DNA pattern matching operations
We use the ```find_pattern``` command to generate an annotation of the genomic
locations where the pattern TATAA happens in the genome assembly GRCh38.

The ```find_pattern``` command requires permission to include annotations.
If the user (e.g. anonymous user) does not have this permission,
an error will be returned.

Nevertheless, we have already processed this pattern, and the annotation was
generated with the name "Pattern TATAAA (non-overlap) in the genome GRCh38".
We selected the genomic regions of the first chromosome of this annotation with
the select_annotations command.

We use the ```select_experiments``` command to select the genomic regions from
the experiments that are in the chromosome 1, position 0 to 50,000,000.

The command ```intersection``` selects all regions of the ```query_id``` that
overlap with at least one tataa_regions region.

We use the ```get_regions``` with the ```query_id``` returned by the
```select_experiments``` and the desired file columns. In this case,
we use the meta-column @SEQUENCE, that includes the DNA Sequence in the genomic
region output.

The ```get_regions``` command is asynchronous. It means that the user receives
a ```request_id``` and should check the status of this request. The specific
DeepBlueR package command ```download_request_data``` wait for the processing
finishes in the server, download the data, and convert to a GRanges object.

```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
tataa_regions = deepblue_select_annotations(
    annotation_name="Pattern TATAAA (non-overlap) in the genome GRCh38",
    genome="GRCh38", chromosome="chr1")
query_id = deepblue_select_experiments(
    experiment_name= c("BL-2_c01.ERX297416.H3K27ac.bwa.GRCh38.20150527.bed",
        "S008SGH1.ERX406923.H3K27ac.bwa.GRCh38.20150728.bed"),
    chromosome="chr1", start=0, end=50000000)
overlapped = deepblue_intersection(query_data_id=query_id,
                                   query_filter_id=tataa_regions)
request_id=deepblue_get_regions(overlapped,
    "CHROMOSOME,START,END,SIGNAL_VALUE,PEAK,@NAME,@BIOSOURCE,@LENGTH,@SEQUENCE,@PROJECT")
regions = deepblue_download_request_data(request_id=request_id)
head(regions, 5)
```

### Genes
We use the ```select_genes``` command to select the gene RP11-34P13
from the gene set gencode v23.

The selected genes behave like a regular genomic region that,
for example, can be filtered by their content.

We use the @GENE_ATTRIBUTE meta-column to select the genomic regions
that are lincRNA.

We use the ```get_regions``` with the ```query_id``` returned
by the ```select_experiments``` and the desired file columns.

The ```get_regions``` command is asynchronous. It means that the user
receives a ```request_id``` and should check the status of this request.
The specific DeepBlueR package command ```download_request_data``` wait for
the processing finishes in the server, download the data, and convert to a
GRanges object.

```{R, echo=TRUE, eval=FALSE, warning=FALSE, message=FALSE}
q_genes = deepblue_select_genes(genes_name="RP11-34P13", gene_model="gencode v23")
q_filter = deepblue_filter_regions(query_id=q_genes,
    field="@GENE_ATTRIBUTE(gene_type)", operation="==",
    value="lincRNA", type="string")
request_id=deepblue_get_regions(q_filter, "CHROMOSOME,START,END,GTF_ATTRIBUTES")
regions = deepblue_download_request_data(request_id=request_id)
```

### Aggregate and summarize regions
We use the ```get_regions``` command with the ```query_id``` value returned by
the select_experiments and the desired file columns.

We use the ```select_annotations``` command to select the genomic regions,
from position 0 to 50,000,000 in the chromosome 1 of the annotation CpG Islands
of the genome assembly GRCh38.

The command ```aggregate``` aggregates the ```query_id``` regions using the
```cpg_islands``` regions as boundaries.

The aggregate values can be accessed through the @AGG meta-columns.

```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
query_id = deepblue_select_experiments (
    experiment=c("GC_T14_10.CPG_methylation_calls.bs_call.GRCh38.20150707.wig"),
    chromosome="chr1", start=0, end=50000000)
cpg_islands = deepblue_select_annotations(annotation_name="CpG Islands",
    genome="GRCh38", chromosome="chr1", start=0, end=50000000)
# Aggregate
overlapped = deepblue_aggregate (data_id=query_id, ranges_id=cpg_islands,
    column="VALUE" )

# Retrieve the experiments data (The @NAME meta-column is used to include
# the experiment name and @BIOSOURCE for experiment's biosource
request_id = deepblue_get_regions(query_id=overlapped,
    output_format=
      "CHROMOSOME,START,END,@AGG.MIN,@AGG.MAX,@AGG.MEAN,@AGG.VAR")
regions = deepblue_download_request_data(request_id=request_id)
regions
```

### Tiling regions
We use the ```get_regions``` command with the ```query_id``` value returned by
the ```select_experiments``` and the desired file columns.

We use the ```tiling_regions``` command to generate a set of consecutive genomic
regions of size 100,000 on the chromosome 1 of the genome assembly GRCh38.

The command ```aggregate``` aggregates the ```query_id``` regions by their
column named VALUE, using the ```cpg_islands``` regions as boundaries.

The aggregation values can be accessed through the @AGG meta-columns.

```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
# Selecting the data from 2 experiments:
#    GC_T14_10.CPG_methylation_calls.bs_call.GRCh38.20150707.wig
# As we already know the experiments names, we keep all others fields empty.
# We are selecting all regions of chromosome 1
query_id = deepblue_select_experiments(
    experiment=c("GC_T14_10.CPG_methylation_calls.bs_call.GRCh38.20150707.wig"),
    chromosome="chr1")

#  Tiling regions of 100.000 base pairs
tiling_id = deepblue_tiling_regions(size=100000,
    genome="GRCh38", chromosome="chr1")

# Aggregate
overlapped = deepblue_aggregate (data_id=query_id,
    ranges_id=tiling_id, column="VALUE")

# Retrieve the experiments data (The @NAME meta-column is used to include the
# experiment name and @BIOSOURCE for experiment's biosource
request_id = deepblue_get_regions(query_id=overlapped,
    output_format="CHROMOSOME,START,END,@AGG.MEAN,@AGG.SD")

regions = deepblue_download_request_data(request_id=request_id)
regions
```

Such data can now be plotted using any of the common R plotting mechanisms
and packages. An example is shown here:

```{r}
library(ggplot2)
plot_data <- as.data.frame(regions)
plot_data[,grepl("X.", colnames(plot_data))] <-
    apply(plot_data[,grepl("X.", colnames(plot_data))], 2, as.numeric)
AGG.plot <- ggplot(plot_data, aes(start)) +
    geom_ribbon(aes(ymin = X.AGG.MEAN - (X.AGG.SD / 2),
        ymax = X.AGG.MEAN + (X.AGG.SD / 2)), fill = "grey70") +
    geom_line(aes(y = X.AGG.MEAN))
print(AGG.plot)
```


### Flanking regions
We use the ```select_genes``` command to generate a set of genes from
the gene set gencode v19.

The ```flank``` command obtains flanking regions based on the existing regions.
First, we generate regions that starts 2500bp before the regions and with the
length of 2000bp. After, we generate the regions that starts 1500 bases pair
after the regions end and have 500 base pairs. We consider the regions strand
in both cases.

The ```merge_queries``` command merges the region sets defined by the query IDs.
We merge the two flanking regions sets with the genes' regions set.

We use the ```get_regions``` with the ```query_id``` that is returned
by the ```merge_queries```.

```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
# Select the RP11-34P13 gene locations from gencode v23
q_genes = deepblue_select_genes(
    genes_name=
        c("RNU6-1100P", "CICP7", "MRPL20", "ANKRD65",
            "HES2", "ACOT7", "HES3", "ICMT"), gene_model="gencode v19")

# Obtain the regions that starts 2500 bases pair before the regions start and
# have 2000 base pairs.
# The 4th argument inform that DeepBlue must consider the region strand
# (column STRAND) to calculate the new region
before_flank_id = deepblue_flank(query_id=q_genes,
     start=-2500, length=2000, use_strand=TRUE)

# Obtain the regions that starts 1500 bases pair after the
# regions end and have 500 base pairs.
# The 4th argument inform that DeepBlue must consider the
# region strand (column STRAND) to calculate the new region
after_flank_id = deepblue_flank(query_id=q_genes,
    start=1500, length=500, use_strand=TRUE)

# Merge both flanking regions set and genes set
flank_merge_id = deepblue_merge_queries(
    query_a_id =before_flank_id, query_b_id=after_flank_id)
all_merge_id = deepblue_merge_queries(
    query_a_id=q_genes, query_b_id=flank_merge_id)

# Request the regions
request_id = deepblue_get_regions(query_id=all_merge_id,
    output_format="CHROMOSOME,START,END,STRAND,@LENGTH")

regions = deepblue_download_request_data(request_id=request_id)
regions
```

### Calculated columns
We use the ```get_regions``` command with the ```query_id``` value returned by
the ```select_experiments``` and the desired file columns.

We use the ```select_annotations``` command to select the genomic regions, from
position 0 to 50,000,000 in the chromosome 1 of the annotation CpG Islands of
the genome assembly GRCh38.

The command ```aggregate``` aggregates the ```query_id``` regions by their
column named VALUE, using the ```cpg_islands``` regions as boundaries.

We select the ```aggregated``` regions that aggregated at least one region from
the selected experiments (@AGG.COUNT > 0).

The aggregation values can be accessed through the @AGG meta-columns.

We use the ```get_regions``` with the ```query_id``` value returned by the
```select_experiments``` and the desired file columns.
We use the @CALCULATED meta-column to transform the aggregate region @AGG.MEAN
value to its log scale value.


```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
# Select the RP11-34P13 gene locations from gencode v23

# Selecting the data from 2 experiments:
#    GC_T14_10.CPG_methylation_calls.bs_call.GRCh38.20150707.wig
# As we already know the experiments names, we keep all others fields empty.
# We are selecting all regions of chromosome 1
query_id = deepblue_select_experiments(
    experiment="GC_T14_10.CPG_methylation_calls.bs_call.GRCh38.20150707.wig",
    chromosome="chr1")

# Select the CpG Islands annotation from GRCh38
cpg_islands = deepblue_select_annotations(
    annotation="CpG Islands", genome="GRCh38", chromosome="chr1")

# Aggregate
overlapped = deepblue_aggregate(
    data_id=query_id, ranges_id=cpg_islands, column="VALUE")

# Select the aggregated regions that aggregated at least one region from the
# selected experiments (@AGG.COUNT > 0)
filtered = deepblue_filter_regions(query_id=overlapped,
    field="@AGG.COUNT", operation=">", value="0", type="number")

# We remove all regions where the aggregation mean is zero.
filtered_zeros = deepblue_filter_regions(query_id=filtered,
    field="@AGG.MEAN", operation="!=", value="0.0", type="number")

# Retrieve the experiments data (The @NAME meta-column is used to include the
# experiment name and @BIOSOURCE for experiment's biosource
request_id = deepblue_get_regions(query_id=filtered_zeros,
    output_format=
    "CHROMOSOME,START,END,@CALCULATED(return math.log(value_of('@AGG.MEAN'))),@AGG.MEAN,@AGG.COUNT")

regions = deepblue_download_request_data(request_id=request_id)

# We have to perform a manual conversion because the
# package can't know the type for calculated columns
regions$`@CALCULATED(return math.log(value_of('@AGG.MEAN')))` =
    as.numeric(regions$`@CALCULATED(return math.log(value_of('@AGG.MEAN')))`)

head(regions, 5)
```

Any numerical values returned by DeepBlue can also be conveniently displayed
using, for example, the DataTrack feature of the GViz bioconductor package as
shown here:

```{r, warning=FALSE, error=FALSE}
library(Gviz)
atrack <- AnnotationTrack(regions,
    name = "CpGs", group = regions$`@BIOSOURCE`, genome="hg38")
gtrack <- GenomeAxisTrack()
itrack <- IdeogramTrack(genome = "hg38", chromosome = "chr1")
dtrack <- DataTrack(regions,
    data="@AGG.MEAN", name = "Log of average methylation value")
plotTracks(list(itrack, atrack, dtrack, gtrack), type="histogram", fontsize=18,
           background.panel = "#FFFEDB", background.title = "darkblue")
```


### Score matrix
The list ```experiments``` contains the names of the experiments fow which we
want to build a score matrix.

We will build the score matrix using the column named VALUE.

We select the CpG islands, which will be used as aggregated regions boundaries.

The ```score_matrix``` command receives the dictionary with the
experiments names and columns that will be used for aggregation,
the regions' boundaries, and the operation that will be performed
(min, max, mean, var, sd, median, count).

The ```score_matrix``` command is asynchronous. It means that the user receives
a ```request_id``` and should check the status of this request. The specific
DeepBlue-R package command ```download_request_data``` wait for the processing
finishes in the server, download the data, and convert to a GRanges object.


```{R, echo=TRUE, eval=TRUE, warning=FALSE, message=FALSE}
experiments =
    c("GC_T14_10.CPG_methylation_calls.bs_call.GRCh38.20150707.wig",
        "C003N351.CPG_methylation_calls.bs_call.GRCh38.20150707.wig",
        "C005VG51.CPG_methylation_calls.bs_call.GRCh38.20150707.wig",
        "S002R551.CPG_methylation_calls.bs_call.GRCh38.20150707.wig",
        "NBC_NC11_41.CPG_methylation_calls.bs_call.GRCh38.20150707.wig",
        "bmPCs-V156.CPG_methylation_calls.bs_call.GRCh38.20150707.wig",
        "S00BS451.CPG_methylation_calls.bs_call.GRCh38.20150707.wig",
        "S00D1DA1.CPG_methylation_calls.bs_call.GRCh38.20150707.wig",
        "S00D39A1.CPG_methylation_calls.bs_call.GRCh38.20150707.wig")

experiments_columns = list()
for (experiment_name in experiments) {
    experiments_columns[[experiment_name]] = "VALUE"
}

cpgs = deepblue_select_annotations(
    annotation_name="Cpg Islands",
    chromosome="chr22", start=0, end=18000000, genome="GRCh38")

request_id = deepblue_score_matrix(
    experiments_columns=experiments_columns,
    aggregation_function="mean", aggregation_regions_id=cpgs)

score_matrix = deepblue_download_request_data(request_id=request_id)
head(score_matrix, 5)
```

```{r, warning=FALSE, echo=TRUE, error=FALSE, eval=TRUE}
library(ggplot2)
score_matrix = tidyr::gather(score_matrix,
    "experiment", "methylation", -CHROMOSOME, -START, -END)
score_matrix$START <- as.factor(score_matrix$START)
ggplot(score_matrix, aes(x=START, y=experiment, fill=methylation)) +
    geom_tile() +
    theme(axis.text.x=element_text(angle=-90))
```

