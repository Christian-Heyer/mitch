# mitch
mitch is an R package for multi-dimensional enrichment analysis. At it's heart, it uses a rank-MANOVA based statistical approach to detect sets of genes that exhibit enrichment in the multidimensional space as compared to the background. Mitch is useful for pathway analysis of profiling studies with two to or more contrasts, or in studies with multiple omics profiling, for example proteomic, transcriptomic, epigenomic analysis of the same samples. Mitch is perfectly suited for pathway level differential analysis of scRNA-seq data.

<img align="center" width="160" height="200" src="https://github.com/markziemann/mitch_paper/blob/master/figs/mitch.png">

## Installation
```
if(!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install("mitch")

library("mitch")
```

## Workflow overview
### Importing gene sets
mitch has a function to import GMT files to R lists (adapted from [Yu et al, 2012](https://dx.doi.org/10.1089%2Fomi.2011.0118) in the [clusterProfiler](http://bioconductor.org/packages/release/bioc/html/clusterProfiler.html) package). For example:
```
download.file("https://reactome.org/download/current/ReactomePathways.gmt.zip", destfile="ReactomePathways.gmt.zip")
unzip("ReactomePathways.gmt.zip")
genesets <- gmt_import("ReactomePathways.gmt")
```
### Importing profiling data
mitch accepts pre-ranked data supplied by the user, but also has a function called `mitch_import` for importing tables generated by Limma, edgeR, DESeq2, ABSSeq, Sleuth, Seurat and Muscat. By default, only the genes that are detected in all contrasts are included, but this behaviour can be modified. The below example imports two edgeR tables called "dge1" and "dge2". Where gene identifiers are coded in the row names.
```
x <- list("dge1"=dge1, "dge2"=dge2)
y <- mitch_import(x, DEtype="edger")
```
mitch can do unidimensional analysis if you provide it a single profile as a dataframe. 
```
y <- mitch_import(df, DEtype="edger")
```

If the gene identifiers are not given in the rownames, then the column can be specified with the `geneIDcol` parameter like this:
```
y <- mitch_import(df, DEtype="edger", geneIDcol="MyGeneIDs")
```
By default, differential gene activity is scored using the directional nominal p-value.

S = -log10(p-value) * sgn(logFC)

If this is not desired, then users can perform their own custom scoring procedure.

There are many cases where the gene IDs don't match the gene sets. To overcome this, `mitch_import` also accepts a two-column table that relates gene identifiers in the profiling data to those in the gene sets. 

`?mitch_import` provides more instructions on using this feature.
### Calculating enrichment
The `mitch_calc` function performs multivariate enrichment analysis of the supplied gene sets in the scored profiling data.  At its simpest form `mitch_calc` function accepts the scored data as the first argument and the genesets as the second argument. Users can prioritise enrichments based on small adjusted p-values, or by the observed effect size (magnitude of the enrichment score).
```
res <- mitch_calc(y, genesets, priority="significance")
res <- mitch_calc(y, genesets, priority="effect")
```
You can peek at the top results with `head` like this:

```
head(res$enrichment_result)
```

By default, `mitch_calc` uses mclapply to speed up calculations on all but one available CPU threads. This behaviour can be modified by setting the `cores` to a desred number.
```
res <- mitch_calc(y, genesets, priority="significance", cores=4)
```
By default, gene sets with fewer than 10 members present in the profiling data are discarded. This threshold can be modified using the `minsetsize` option. There is no upper limit of gene set size.
```
res <- mitch_calc(y, genesets, priority="significance", minsetsize=20)
```
By default, in downstream visualisation steps, charts are made from the top 50 gene sets, but this can be modified using the `resrows` option. 
```
res <- mitch_calc(y, genesets, priority="significance", resrows=100)
```
### Generate a HTML report
Can be done like this:
```
mitch_report(res, "myreport.html")
```
Take a look at an [example report](https://github.com/markziemann/mitch_paper/blob/master/figs/myreport.html).

### Generate high resolution plots
In case you want the charts in PDF format, these can be generated as such:
```
mitch_plots(res, outfile="mycharts.pdf")
```
Take a look at an [example plot set](https://github.com/markziemann/mitch_paper/blob/master/figs/mycharts.pdf).
