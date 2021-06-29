# coselenss
COnditional SELection on the Excess of NonSynonymous Substitutions (coselenss) is an R package to detect gene-level differential selection between two groups of samples. If the samples are grouped based on the value of a binary variable (e.g., the presence/absence of some environmental stress or phenotypic trait), coselenss identifies genes that are differentially selected depending on the grouping variable and provides maximum likelihood estimates of the effect sizes. Coselenss makes extensive use of the ```dndscv``` R package. In short, the dndscv method estimates the number of nonsynonymous mutations that would be expected in the absence of selection by combining a nucleotide substitution model (196 rates encompassing all possible substitutions in each possible trinucleotide context, estimated from synonymous mutations in the dataset) and a set of genomic covariates which greatly improve the performance of the method at low mutation loads. Then, dn/ds is calculated as the ratio between the observed number of nonsynonymous substitutions (n_obs) and its neutral expectation (n_exp). Finally, the significance of dn/ds is computed through a likelihood ratio test, where the null hypothesis corresponds to dn/ds=1.

Coselenss expands the dndscv method in two substantial ways. First, it quantifies the difference (rather than the ratio) between the observed number of nonsynonymous substitutions and its neutral expectation (Δn = n_obs - n_exp). This change of perspective is most relevant for applications in which the variable of interest is the number (rather than the fraction) of mutations subject to positive (or negative) selection, such as for the inference of driver mutations in cancer. Second, it uses a modified likelihood ratio test that allows for comparison of Δn between two sets of samples, whose mutation rates and profiles are independently estimated. 

A full length tutorial on how to use coselenss can be found here.

# Installation
Coselenss makes heavy use of ```dndscv```, but a slightly customized version of this package is used under-the-hood and does **not** require installation. However, coselenss does require the following dependencies: ```BiocManager```, ```devtools```, ```seqinr```, ```MASS```, ```GenomicRanges```, ```Biostrings```, ```IRanges```, and ```MASS```. These can be installed by either ```install.packages()``` or ```BiocManager::install()```. To install coselenss use:
```
devtools::install_github("ggruenhagen3/coselenss")
```

# Input
* group1: mutation file for the first group of samples (for example, those that possess the trait of interest)
* group2: mutation file for the second group of samples (for example, those that do NOT possess the trait of interest)
* subset.genes.by (optional): genes to subset results by
* refdb (optional): reference database of coding sequences as an .rda file, the default is GRCh37/hg19

The input parameters group1 and group2 are two dataframes of mutations, one for each group of samples. Each dataframe of mutations should have 5 columns: sampleID, chr (chromosome), pos (position within the chromosome), ref (reference base), alt (mutated base). Only list independent events as mutations. An example of the format of the table:

|sampleID | chr | pos | ref | alt|
|---------|-----|-----|-----|----|
|sample1  | 1   | 123 | A   | T  |
|sample1  | 5   | 456 | C   | G  |
|sample2  | 2   | 321 | T   | A  |
|sample3  | 3   | 789 | A   | G  |
|sample3  | 11  | 987 | G   | C  |

By default, coselenss assumes that the mutation data is mapped to the GRCh37/hg19 assembly of the human reference genome. To use coselenss with different species or assemblies, an alternative reference database (RefCDS object) must be provided with the option refdb. The generation of alternative reference databases can be done using the dndscv package and is explained in this tutorial: [link to http://htmlpreview.github.io/?http://github.com/im3sanger/dndscv/blob/master/vignettes/buildref.html]

# Output
Coselenss returns a dataframe with effect sizes and p-values for differential selection in of the reference genome. If a list of genes is provided through the subset.genes.by option, the results and Benjamini-Hochberg corrections are restricted to those genes. The columns returned are described as follows:
* gene_name: name of gene in which differential selection was studied.
* num.drivers.group1: estimate of the excess of non-synonymous mutations (Δn) in group 1, with respect to the neutral expectation. In the absence of negative selection, this number corresponds to the average number of driver (i.e., positively selected) mutations per sample  in that gene.
* num.drivers.group2: idem, for group 2.
* pmis: p-value for differential selection in missense mutations.
* ptrunc: p-value for differential selection in truncating mutations.
* pall: p-value for differential selection in all substitutions (pmis and ptrunc).
* pind: p-value for differential selection in small indels.
* pglobal: Fisher's combined p-value for pall and pind.
* qglobal: q-value of pglobal using Benjamini-Hochberg correction.

Note that the Fisher’s combined value of pglobal/qglobal may be too conservative if the sensitivity of the pall or pind test is low. In our experience with cancer somatic mutation data, the indel test (pind) only reaches acceptable sensitivity levels if the sample size is large (>100) and indels are frequent. Otherwise, the low sensitivity of the indel test results in non-significant values of pglobal, despite the presence of substantial differential selection on substitutions. Thus, we recommend using pall/qall to assess significance of differential selection on substitutions, while restricting pglobal/qglobal to datasets with large sample sizes and genes in which indels are the main subject of selection.

# Example
Coselenss was developed to discover epistatic interactions between cancer genes in specific cancer types. To do that, we searched for differential selection in cancer genes when mutations in another cancer gene were present/absent. As an example, we took a somatic mutation dataset built from biopsies from patients with colorectal cancer (COAD) and split them into two groups, those with mutations in APC and those without mutations in APC. Let's begin, by loading these two mutation datasets.

```
library("coselenss")
data("group1", package = "coselenss")  # mutations from patients with    mutations in APC
data("group2", package = "coselenss")  # mutations from patients without mutations in APC
```

Next, let's detect differential selection in genes with APC in COAD (runtime ~5.5 minutes on a laptop).

```
coselenss_res = coselenss(group1, group2)
```

Use ```head(coselenss_res)```, the output should look like this:

| gene_name | num.drivers.group1 | num.drivers.group2 | pmis | ptrunc | pall | pind| pglobal | qglobal |
|-----------|--------------------|--------------------|------|--------|------|-----|---------|---------|
|A1BG       |-0.0002757828|0.003490537|0.6774850|1.0000000|0.9171490|0.28669884|0.6141904|1|
|A1CF       |-0.0033004353|-0.008295631|0.6058041|1.0000000|0.8753205|1.00000000|0.9918827|1|
|A2M        |0.0037921837|0.029329399|0.3043810|0.5185936|0.4791242|NaN|NaN|NaN|
|A2ML1      |0.0295412248|0.040366524|0.7277502|0.7012937|0.8744515|0.09369533|0.2869149|1|
|A3GALT2    |-0.0047941596|-0.006902210|0.8620675|1.0000000|0.9850200|1.00000000|0.9998872|1|
|A4GALT     |-0.0012767999|0.017041030|0.1117167|1.0000000|0.2822721|0.23622026|0.2472351|1|

Let's find the genes subject to significant significant differential selection by doing the following:

```
coselenss_res[which(coselenss_res$qglobal < 0.05),]
```

The output should look like this:
| gene_name | num.drivers.group1 | num.drivers.group2 | pmis | ptrunc | pall | pind| pglobal | qglobal |
|-----------|--------------------|--------------------|------|--------|------|-----|---------|---------|
|APC        |0.98535779|-0.04327314|4.599838e-02|1.669909e-08|1.661006e-08|0.2386762|8.065981e-08|0.001610938|
|BRAF       |0.03621197|0.22591651|2.276400e-08|1.821872e-01|6.747822e-08|0.4244034|5.260378e-07|0.005253014|

We detected two genes, but APC was the gene used to separate individuals in the beginning, so it's expected that it should be significant. We can remove APC because it is the trivial solution, but we have just discovered that there may be conditional selection between APC and BRAF in COAD. Feel free to think outside the box and make discories of your own using our tool!
