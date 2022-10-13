# ![alt text](https://github.com/ggruenhagen3/coselens/blob/master/icon.png?raw=true)  Coselens 
COnditional SELection on the Excess of NonSynonymous Substitutions (coselens) is an R package to detect gene-level differential selection between two groups of samples. If the samples are grouped based on the value of a binary variable (e.g., the presence/absence of some environmental stress or phenotypic trait), coselens identifies genes that are differentially selected depending on the grouping variable and provides maximum likelihood estimates of the effect sizes. Coselens makes extensive use of the ```dndscv``` R package. In short, the dndscv method estimates the number of nonsynonymous mutations that would be expected in the absence of selection by combining a nucleotide substitution model (196 rates encompassing all possible substitutions in each possible trinucleotide context, estimated from synonymous mutations in the dataset) and a set of genomic covariates which greatly improve the performance of the method at low mutation loads (read more about dndscv [here](https://github.com/im3sanger/dndscv)). Then, dn/ds is calculated as the ratio between the observed number of nonsynonymous substitutions (n_obs) and its neutral expectation (n_exp). Finally, the significance of dn/ds is computed through a likelihood ratio test, where the null hypothesis corresponds to dn/ds=1.

Coselens expands the dndscv method in two substantial ways. First, it quantifies the difference (rather than the ratio) between the observed number of nonsynonymous substitutions and its neutral expectation (Δn = n_obs - n_exp). This change of perspective is most relevant for applications in which the variable of interest is the number (rather than the fraction) of mutations subject to positive (or negative) selection, such as for the inference of driver mutations in cancer. Second, it uses a modified likelihood ratio test that allows for comparison of Δn between two sets of samples, whose mutation rates and profiles are independently estimated. 

A full length tutorial on how to use coselens can be found here.

# Installation
Coselens makes heavy use of ```dndscv```, but a slightly customized version of this package is used under-the-hood and does **not** require installation. However, coselens does require the following dependencies: ```BiocManager```, ```devtools```, ```geometry```, ```seqinr```, ```MASS```, ```GenomicRanges```, ```Biostrings```, and ```IRanges```. These can be installed by either ```install.packages()``` or ```BiocManager::install()```. To install coselens use:
```
devtools::install_github("ggruenhagen3/coselens")
```

# Input
* group1: mutation file for the first group of samples (for example, those that possess the trait of interest)
* group2: mutation file for the second group of samples (for example, those that do NOT possess the trait of interest)
* subset.genes.by (optional): genes to subset results by
* sequenced.genes (optional): the gene_list paramater from dndscv, which is a list of genes to restrict the analysis (use for targeted sequencing studies)
* ... other paramters passed to dncdscv, all defaults from ```dndscv``` are used except max_muts_per_gene_per_sample is set to Infinity

The input parameters group1 and group2 are two dataframes of mutations, one for each group of samples. Each dataframe of mutations should have 5 columns: sampleID, chr (chromosome), pos (position within the chromosome), ref (reference base), alt (mutated base). Only list independent events as mutations. An example of the format of the table:

|sampleID | chr | pos | ref | alt|
|---------|-----|-----|-----|----|
|sample1  | 1   | 123 | A   | T  |
|sample1  | 5   | 456 | C   | G  |
|sample2  | 2   | 321 | T   | A  |
|sample3  | 3   | 789 | A   | G  |
|sample3  | 11  | 987 | G   | C  |

By default, coselens assumes that the mutation data is mapped to the GRCh37/hg19 assembly of the human reference genome. To use coselens with different species or assemblies, an alternative reference database (RefCDS object) must be provided with the option refdb. The generation of alternative reference databases can be done using the dndscv package and is explained in [this tutorial](http://htmlpreview.github.io/?http://github.com/im3sanger/dndscv/blob/master/vignettes/buildref.html):

# Output
Coselens returns a list of 4 dataframes. The first is probably the most pertinent to the majority of users, it contains information on effect sizes and p-values for differential selection of genes between the input groups. If a list of genes is provided through the subset.genes.by option, the results and Benjamini-Hochberg corrections are restricted to those genes. Detailed descriptions of each of the 4 dataframes are below.

* summary: a summary of coselens output that should be sufficient for most users
  <details>
  <summary>More Details</summary>
  <br>
 
  * Output Column Descriptions
    * gene_name: name of gene that conditional selection was calculated in
    * num.driver.sub.group1: estimate of the number of drivers in group 1 based excess of non-synonymous mutations
    * num.driver.sub.group2: estimate of the number of drivers in group 2 based excess of non-synonymous mutations
    * num.driver.ind.group1: estimate of the number of drivers in group 1 based excess of indels
    * num.driver.ind.group2: estimate of the number of drivers in group 2 based excess of indels
    * psub: p-value for conditional selection in non-synonymous substitutions
    * pind: p-value for conditional selection in indels
    * pglobal: Fisher's combined p-value for psub and pind
    * qsub: q-value of psub using Benjamini-Hochberg correction
    * qind: q-value of pind using Benjamini-Hochberg correction
    * qglobal: q-value of pglobal using Benjamini-Hochberg correction
  </details>

* full: similar to summary, but with more columns
  <details>
  <summary>More Details</summary>
  <br>

  * Output Column Descriptions (same as summary w/ the following additions):
    * num.driver.mis.group1: estimate of the number of drivers in group 1 based excess of missense mutations
    * num.driver.mis.group2: estimate of the number of drivers in group 2 based excess of missense mutations
    * num.driver.trunc.group1: estimate of the number of drivers in group 1 based excess of truncating mutations
    * num.driver.trunc.group2: estimate of the number of drivers in group 2 based excess of truncating mutations
    * pmis: p-value for conditional selection in missense mutations
    * ptrunc: p-value for conditional selection in truncating mutations
    * psub.group1: p-value for selection in group 1 for non-synonymous substitutions
    * psub.group2: p-value for selection in group 2 for non-synonymous substitutions
    * pmis.group1: p-value for selection in group 1 for missense substitutions
    * pmis.group2: p-value for selection in group 2 for missense substitutions
    * ptrunc.group1: p-value for selection in group 1 for truncating substitutions
    * ptrunc.group2: p-value for selection in group 2 for truncating substitutions
    * pind.group1: p-value for selection in group 1 for indels
    * pind.group2: p-value for selection in group 2 for indels
    * qsub.group1: q-value of psub.group1 using Benjamini-Hochberg correction
    * qsub.group2: q-value of psub.group2 using Benjamini-Hochberg correction
    * qmis.group1: q-value of pmis.group1 using Benjamini-Hochberg correction
    * qmis.group2: q-value of pmis.group2 using Benjamini-Hochberg correction
    * qtrunc.group1: q-value of ptrunc.group1 using Benjamini-Hochberg correction
    * qtrunc.group2: q-value of ptrunc.group2 using Benjamini-Hochberg correction
    * qind.group1: q-value of pind.group1 using Benjamini-Hochberg correction
    * qind.group2: q-value of pind.group2 using Benjamini-Hochberg correction
  </details>

* mle_submodel_group1: fitted substitution models for group 1 from dndscv
* mle_submodel_group2: fitted substitution models for group 2 from dndscv

Note that the Fisher’s combined value of pglobal/qglobal may be too conservative if the sensitivity of the pall or pind test is low. In our experience with cancer somatic mutation data, the indel test (pind) only reaches acceptable sensitivity levels if the sample size is large (>100) and indels are frequent. Otherwise, the low sensitivity of the indel test results in non-significant values of pglobal, despite the presence of substantial differential selection on substitutions. Thus, we recommend using pall/qall to assess significance of differential selection on substitutions, while restricting pglobal/qglobal to datasets with large sample sizes and genes in which indels are the main subject of selection.

# Example
Coselens was developed to discover epistatic interactions between cancer genes in specific cancer types. To do that, we searched for differential selection in cancer genes when mutations in another cancer gene were present/absent. As an example, we took a somatic mutation dataset built from biopsies from patients with colorectal cancer (COAD) and split them into two groups, those with mutations in APC and those without mutations in APC. Let's begin, by loading these two mutation datasets.

```
library("coselens")
data("group1", package = "coselens")  # mutations from patients with    mutations in APC
data("group2", package = "coselens")  # mutations from patients without mutations in APC
data("cancer_genes", package = "coselens")  # cancer genes
```

Next, let's detect differential selection in genes with APC in COAD (runtime ~5.5 minutes on a laptop).

```
coselens_res = coselens(group1, group2, subset.genes.by = cancer_genes)
```

Use ```head(coselens_res$summary)```, the output should look like this:

| gene_name | num.driver.sub.group1 | num.driver.sub.group2 | num.driver.ind.group1 | num.driver.ind.group2 | psub | pind | pglobal | qsub | qind | qglobal |
|-----------|--------------------|--------------------|------|--------|------|-----|---------|---------|--------|
|ABL1|-8.257161e-05|0.021727512 |-0.0018375172|0.0007703715 |0.4441491|0.5359755|0.5797215|1|1|1|
|ACO1|-1.254983e-03|-0.004122997|0.0061894558 |-0.0053114291|0.9125477|0.2687322|0.5899164|1|1|1|
|ACVR1|3.590744e-04|0.008633316 |-0.0008275762|-0.0032018083|0.7083432|1.0000000|0.9525987|1|1|1|
|ACVR1B|3.921533e-02|0.039206586|-0.0008321666|0.0039877767 |0.9904685|0.3391245|0.7023388|1|1|1|
|ACVR2A|1.293825e-02|0.018095464|0.0119255807 |0.0107532703 |0.6835661|1.0000000|0.9436165|1|1|1|
|ACVR2B|8.398367e-03|0.028399806|-0.0006982841|-0.0027780460|0.2041043|1.0000000|0.5284514|1|1|1|

Let's find the genes subject to significant significant differential selection by doing the following:

```
head(coselens_res$summary[which(coselens_res$summary$qglobal < 0.05),])
```

The output should look like this:
| gene_name | num.driver.sub.group1 | num.driver.sub.group2 | num.driver.ind.group1 | num.driver.ind.group2 | psub | pind | pglobal | qsub | qind | qglobal |
|-----------|--------------------|--------------------|------|--------|------|-----|---------|---------|--------|
|APC|0.98535779 |-0.04327314|0.421980339 |-0.01550676|1.661006e-08|0.2386762|8.065981e-08|7.308425e-06|1|3.532899e-05|
|BRAF|0.03621197|0.22591651 |-0.001109252| 0.00270349|6.747822e-08|0.4244034|5.260378e-07|1.484521e-05|1|1.152023e-04|

We detected 2 significant genes, but APC was the gene used to separate individuals in the beginning, so it's expected that it should be significant. We can remove APC because it is the trivial solution. The q-value of BRAF is really low this provides strong evidence of conditional selection between APC and BRAF in COAD. Now that you see how the tool works, feel free to think outside the box and make discoveries of your own!
