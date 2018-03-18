Report Matches to Potti Columns
================
Keith Baggerly
2018-03-18

-   [Executive Summary](#executive-summary)
    -   [Introduction](#introduction)
        -   [GitHub Site](#github-site)
    -   [Data and Methods](#data-and-methods)
    -   [Results](#results)
    -   [Conclusions](#conclusions)
-   [Data Gathering](#data-gathering)
-   [Data Exploration](#data-exploration)
-   [Data Preprocessing](#data-preprocessing)
-   [Checking Column Matches](#checking-column-matches)

Executive Summary
=================

Introduction
------------

In 2006, [Potti et al](https://www.nature.com/articles/nm1491) claimed to have found a way to use microarray profiles of a specific panel of cell lines (the NCI60) to predict cancer patient response to chemotherapeutics from a similar profile of the patient's tumor. Using different subsets of cell lines, they made predictions for several different drugs. We wanted to apply their method, so we asked them to send us lists of which cell lines were used to make predictions for which drugs. The method doesn't work; we describe our full analyses [here](https://projecteuclid.org/euclid.aoas/1267453942).

The first dataset (an expression matrix with genes in rows and samples in columns) we received from Potti et al didn't have the cell lines labeled; for each column we could tell which drug it was supposed to be associated with, and whether it was in contrast group "0" or "1". We want to see if we can identify where the numbers came from and see if there were any oddities that should have raised red flags early on.

### GitHub Site

Full details supporting this report are available at

<https://github.com/kabagg/enar_2018_rr_short_course>

Relative links make use of the R package `here`.

``` r
library(here)
```

Data and Methods
----------------

We examined 3 datasets:

-   The data table from Potti et al, available at
-   Gene expression values from microarray profiling (in triplicate) of the NCI60 cell lines by Novartis, at the individual replicate level, available from URL as FILENAME
-   Gene expression values from microarray profiling of tumors from 24 breast cancer patients treated with single agent docetaxel, split into those who did and did not respond to treatment available, available from the Gene Expression Omnibus (GEO) as GSE349 (resistant), URL, and GSE350 (sensitive), URL. This cohort was used by Potti et al to validate their predictions for docetaxel.

All of these datasets involve measurements from Affymetrix U-95A or U-95Av2 gene chips, which give measurements on 12625 probesets, of which 67 are controls and 12558 correspond to genes.

The initial Potti data matrix is 12558 by 134; there are no labeled rows for the control probes. The first probeset listed is 36460\_at.

The NCI60 data matrix is 12625 by 180; the 59 cell lines in the panel were mostly run in triplicate, but a few were run 2 or 4 times. Replicates were labeled A, B, C, or D as appropriate. As with the Potti et al data matrix, the first listed probeset is 36460\_at.

The GEO data matrix is 12625 by 24. Values are reported first for the control probes and then for the genes; the probeset ordering is different than that in the NCI60 dataset.

We first examined the raw datasets for basic dimensions and signs of structure, after which we processed the datasets to put each in the form of gene by sample expression matrices with row and column names and associated data frames of sample and/or gene annotation, as available. We then tried matching the expression values for specific probesets to see if we could identify replicate columns within a dataset (e.g., cell lines used to supply information about more than one drug) or between datasets to identify cell lines by matching the unlabeled columns in the Potti dataset with annotation information in the other datasets. We also checked correlations within and between datasets to see what the range of concordance was.

All analysis R and Rmd files are stored in R/.

Results
-------

Expression values were reported to 6 decimal places, so exact matching gave a shortcut means of identification. Checking all values of 36460\_at in the Potti et al dataset against the NCI60 data matrix gave precise identifications for all of the cell lines used for 5 of the 7 drugs investigated; in all cases the "A" replicate values were used. We found no matches for any of the columns provided for Docetaxel or Cytoxan (20 columns each) in the Potti et al dataset.

Checking pairwise correlations within the Potti dataset showed reasonable positive correlations for all pairs of samples for which we'd found a match (expected, as many genes perform the basic functions of cellular maintenance regardless of background), but correlations between the matched columns and those for Docetaxel and Cytoxan were starkly different, all being around 0. Correlations within the Docetaxel and Cytoxan columns were again positive, and indeed showed the two sets of columns were identical modulo swapping of contrast group labels - the 10 cell lines labeled as belonging to group "0" for Docetaxel were the same as those comprising group "1" for Cytoxan, and vice versa.

Examination of the minimum values of the Docetaxel and Cytoxan columns showed these were all tied at 5.89822, whereas the minimums for all other data columns were much smaller. Since our earlier skims of the data from GEO (which had been processed differently than the NCI60 data) had also showed tied minimum values of 5.89822, we checked for matches between the Docetaxel/Cytoxan columns and the GEO columns. Starting with row 68 in the GEO dataset (right after the control probes), we found matches for all of the Docetaxel and Cytoxan columns with values listed *in the order in which they were provided at GEO*. This matching was exact for 12535 rows; the remaining 23 still had the GEO values but in a slightly different order. Since the probeset orders were different between the NCI and GEO datasets, this effectively produced a random scrambling of the gene values between the two datasets, for which we would expect the correlation to be 0.

We produced a table mapping each column of the Potti et al dataset to its corresponding NCI60 or GEO sample in [results/potti\_matches.csv](/Users/kabaggerly/Professional/Talks/2018/2018_03_25_ENAR/TempGit/enar_2018_rr_short_course/results/potti_matches.csv).

Conclusions
-----------

Given the table we received from Potti et al, we can infer which cell lines were used in each contrast group for 5 of the 7 drugs examined. We can't easily say whether contrast group 0 should correspond to sensitive and 1 to resistant or the converse from the data supplied.

The matches we find for Docetaxel and Cytoxan to values in the Docetaxel test set make us uneasy. Combining training and test data in a single data matrix can readily lead to confusion, and we see no reason why the test data should be here, let alone also be present for Cytoxan. We also don't understand why the Docetaxel and Cytoxan columns should be mirror images of each other.

Data Gathering
==============

Harvesting data from the web is described in [results/01\_gather\_data.md](/Users/kabaggerly/Professional/Talks/2018/2018_03_25_ENAR/TempGit/enar_2018_rr_short_course/results/01_gather_data.md).

The raw data files chemo.zip, WEB\_DATA\_NOVARTIS\_ALL.ZIP, GSE349\_family.soft.gz, and GSE350\_family.soft.gz are stored in [data/](/Users/kabaggerly/Professional/Talks/2018/2018_03_25_ENAR/TempGit/enar_2018_rr_short_course/data).

Data Exploration
================

Initial examination of the raw data supplied is described in [results/02\_describing\_raw\_data.md](/Users/kabaggerly/Professional/Talks/2018/2018_03_25_ENAR/TempGit/enar_2018_rr_short_course/results/02_describing_raw_data.md).

We note the common ordering of the initial probesets between the Potti et al and NCI60 datasets, note that the level of precision in reporting should let matches serve as unique identifiers, and confirm the existence of at least one such match.

We note the NCI60 data table initially involves more rows than would be expected from 180 arrays, suggesting some duplication which would need to be removed. We also note that information about which replicate array (A, B, C, or D) a given value came from is only present in a column also listing a gene cluster (GD) identifier for the gene being measured, thus combining sample (replicate) and gene information.

We note the GEO data probesets are differently ordered than the NCI60 ones, and note the presence of tied values, particularly at the value of 5.89822. This corresponds to the minimum value per column, which should map to essentially no expression of that gene.

Data Preprocessing
==================

Reorganization of the raw data into expression matrices and annotation data frames is described in [results/03\_preprocessing\_data\_base\_r\_version.md](/Users/kabaggerly/Professional/Talks/2018/2018_03_25_ENAR/TempGit/enar_2018_rr_short_course/results/03_preprocessing_data_base_r_version.md).

We store the reorganized data from each dataset in RData files in results/: potti\_data.RData, nci60\_data.RData, and geo\_data.RData, respectively.

Checking Column Matches
=======================

Identification of column matches is described in [results/04\_check\_sample\_matches\_and\_corrs.md](/Users/kabaggerly/Professional/Talks/2018/2018_03_25_ENAR/TempGit/enar_2018_rr_short_course/results/04_check_sample_matches_and_corrs.md).

Correlations between pairs of columns in the Potti et al dataset show clear bifurcations at the boundaries between the columns for Docetaxel and Cytoxan and the rest.

![Potti et al pairwise correlations](/Users/kabaggerly/Professional/Talks/2018/2018_03_25_ENAR/TempGit/enar_2018_rr_short_course/results/04_check_sample_matches_and_corrs_files/figure-markdown_github/plot_raw_potti_cors-1.png)

Checking the sample minimums also highlights the discordance, while also showing that the minimum values in the problematic columns are coming in at 5.89822, which we previously saw to be the minimum of the data columns from GEO.

![Potti et al pairwise correlations](/Users/kabaggerly/Professional/Talks/2018/2018_03_25_ENAR/TempGit/enar_2018_rr_short_course/results/04_check_sample_matches_and_corrs_files/figure-markdown_github/plot_potti_minimums-1.png)

Checking the number of exact matches between the values in the Docetaxel columns in the Potti et al dataset (which are the same as the columns reported for Cytoxan as well) with the data values from GEO using the ordering from GEO shows perfect fits for 12535 of 12558 values for each sample; the last 23 GEO rows also match the last 23 values in the Potti et al columns, but not in linear order.

![Potti et al matches to GEO](/Users/kabaggerly/Professional/Talks/2018/2018_03_25_ENAR/TempGit/enar_2018_rr_short_course/results/04_check_sample_matches_and_corrs_files/figure-markdown_github/plot_potti_doce_geo_matches-1.png)
