

# Methodology used for collecting the data


1. Identify the data source and the annotations from the article
2. Create a new R script that converts the data to an `MSnSet` object
3. Add a new section in the `scpdata/inst/extdata/README.md` file
4. Add data documentation in `scpdata/R/data.R`
5. Update the `scpdata/README.md` file
6. Eventually add experimental code in the `vignettes/scpdata.Rmd` vignette and add utility functions in `scpdata/R/utils.R` script if needed


# From Specht et al. 2019


The 3 data files are available on the [SlavovLab](https://scope2.slavovlab.net/docs/data) website. The available files are:

* `Peptides-raw.csv`: Peptides x single cells at 1% FDR. The first 2 columns list the corresponding protein identifiers and peptide sequences and each subsequent column corresponds to a single cell. Peptide identification is based on spectra analyzed by MaxQuant and is enhanced by using DART-ID to incorporate retention time information. See Specht et al., 2019 for details.
* `Proteins-processed.csv`: Proteins x single cells at 1% FDR, imputed and batch corrected.
* `Cells.csv`: Annotation x single cells. Each column corresponds to a single cell and the rows include relevant metadata, such as, cell type if known, measurements from the isolation of the cell, and derivative quantities, i.e., rRI, CVs, reliability.

The `Peptides-raw.csv` data have already been processed to some extent. The `*.raw` files were analyzed with MaxQuant + DART-ID and the output `.txt` file was parsed into R. This output was further processed by the authors as follows:

- Only the single cell runs were kept for further analysis (experiment FP94 and FP97, no experimental information supplied)
- TMT reporter itensities (RI) were corrected for isotopic cross contamination. This is performed as follows: $cM = (C^{-1} M^T)^T$, where $M$ is the matrix containing the RI (cells $\times$ TMT reporter), and $C$ is 
$\mathbf{\color{red}{\text{no idea, this is a file loaded in the script called te269088_lot_correction.csv}}}$, (cells $\times$ cells).
- Filter out reverse hits (identified by MaxQuant), contaminants (identified by MaxQuant), and contaminated spectra (`PIF > 0.8`)
- Filter out peptides with low identification score  (`FDR >= 1%` or `PEP >= 0.02`)
- Filter out cells with less than 300 peptides
- Filter out peptides that are more than 10\% the intensity of the carrier
- Divide peptide intensities in every channel by the reference channel
- Zero or infinite intensities are replaced by `NA`'s
- Filter out cells that have a median CV larger than 0.43, for which the 30th quantile of the log10 transformed relative RIs is smaller than -2.5, or for which the median of the log10 transformed relative RI is larger than -1.3
- Divide column (cells) with median intensity and divide rows with mean intensity
- Remove rows (peptides) then columns (cells) that contain more than 99\% of missing data
- Log2 transform the data 

The peptide data (`Peptides-raw.csv`) and the meta data (`Cells.csv`) were combined into an MSnSet object (see `scpdata/inst/script/specht2019.R`).


# From Dou et al. 2019


The article contains 3 SCP data sets available on the [ACS Publications](https://pubs.acs.org/doi/10.1021/acs.analchem.9b03349) website:

* Supplementary data set 1, raw data for HeLa digest (`.xlsx`)
* Supplementary data set 2, raw data for testing boosting ratios (`.xlsx`)
* Supplementary data set 3, raw data for isobaric labelling-based single cell quantification and bulk-scale label free quantification (`.xlsx`)

## Supplementary data set 1

We arbitrarily call this data set `dou2019_1`. The `.xlsx` spreadsheet contrains the following sheets:

* `0 - Description`
* `1 - Run 1 raw data`: This table contains the raw data for the Run 1 only. It correspond to the assembly of MSGF+ identifications and MASIC reporter assemblies. The data was then isotope corrected and sum rolled-up to the protein level. Contaminant and reverse hit were removed from this table. Median values are indicated at the bottom of the table
* `2 - Run 1 processed data`: This table contains the raw data for the Run 1 alone. The protein data was log2 transformed, median normalized, the "sva::ComBat" function was used to remove TMT-set-dependent Batch effects. 
* `3 - Run 2 raw data`:	This table contains the raw data for the Run 2 only. It correspond to the assembly of MSGF+ identifications and MASIC reporter assemblies. The data was then isotope corrected and sum rolled-up to the protein level. Contaminant and reverse hit were removed from this table. Median values are indicated at the bottom of the table
* `4 - Run 2 processed data`:	This table contains the raw data for the Run 2 alone. The protein data was log2 transformed, median normalized, the "sva::ComBat" function was used to remove TMT-set-dependent Batch effects. 
* `5 - Run 1 and 2 raw data`:	This table contains the raw data for the *[Run 1 only]* (the authors probably mean Run 1 and Run 2). It correspond to the assembly of MSGF+ identifications and MASIC reporter assemblies. The data was then isotope corrected and sum rolled-up to the protein level. Contaminant and reverse hit were removed from this table. Median values are indicated at the bottom of the table
* `6 - Run 1 and 2 processed data`:	This table contains the raw data for the Run 1 and 2 together. The protein data was log2 transformed, median normalized, the "sva::ComBat" function was used to remove TMT-set-dependent Batch effects. The crossdataset missingness is indicated at the right of the table

We only used the sheet 5 (`Run 1 and 2 raw data`) containing the combined "raw" data for the two runs. This data set is converted to an `MSnSet` object in the script `scpdata/inst/script/dou2019.R`.

## Supplementary data set 2

We arbitrarily call this data set `dou2019_2`. The `.xlsx` spreadsheet contrains the following sheets:

* `0 - Description`
* `01 - No Boost raw data`:	This table contains the raw data for the Run 1 and 2 of the no boost. It correspond to the assembly of MSGF+ identifications and MASIC reporter assemblies. The data was then isotope corrected and sum rolled-up to the protein level. Contaminant and reverse hit were removed from this table.
* `02 - No Boost processed data`:	This table contains the raw data for the Run 1 and 2 of the no boost. The protein data was log2 transformed, median normalized, the "sva::ComBat" function was used to remove TMT-set-dependent Batch effects. 
* `03 - 5ng boost raw data`:	This table contains the raw data for the Run 1 and 2 of the 5ng boost. It correspond to the assembly of MSGF+ identifications and MASIC reporter assemblies. The data was then isotope corrected and sum rolled-up to the protein level. Contaminant and reverse hit were removed from this table.
* `04 - 5ng boost processed data`:	This table contains the raw data for the Run 1 and 2 of the 5ng boost. The protein data was log2 transformed, median normalized, the "sva::ComBat" function was used to remove TMT-set-dependent Batch effects. 
* `05 - 50ng boost raw data`:	This table contains the raw data for the Run 1 and 2 of the 50ng boost. It correspond to the assembly of MSGF+ identifications and MASIC reporter assemblies. The data was then isotope corrected and sum rolled-up to the protein level. Contaminant and reverse hit were removed from this table.
* `06 - 50ng processed data`:	This table contains the raw data for the Run 1 and 2 of the 50ng boost. The protein data was log2 transformed, median normalized, the "sva::ComBat" function was used to remove TMT-set-dependent Batch effects. 

We combined the data from sheet 01, 03, and 05, that is all tables containing the "raw" data. This corresponds to merging the data sets with no boosting, 5ng boosting, and 50 ng boosting. The boosting quantities are kept as feature data. Combining data is performed by matching the proteins beetween boosting batches. The data set is formated to an `MSnSet` object in the script `scpdata/inst/script/dou2019.R`.

## Supplementary data set 3

We arbitrarily call this data set `dou2019_3`. The `.xlsx` spreadsheet contrains the following sheets:

* `00 - Description`
* `01 - Raw sc protein data` Contains the assembly of MSGF+ identifications and MASIC assemblies isotope corrected and sum rolled-up to the protein level. Contaminant and reverse hit were removed from this table
* `02 - Processed sc protein data` Only the proteins with at least 2 unique peptides were conserved for further analysis the protein data was log2 transformed, filtered for outliers using the pmartR method (PMID: 30638385 ), median normalized, only proteins with at least 60% of values within a cell type were conserved for further analysis, the "sva::ComBat" function was used to remove TMT-set-dependent Batch effects. Paired two-tailed heteroscedastic T-tests were performed in order to establish protein enriched in a given condition and Z-scores were calculated for heatmap visualization.
* `03 - Bulk Proteomics` The bulk proteomics was generated on a lysate of each cell type in 5 technical replicates. iBAQ intensities were used the data was log transformed, median normalized, imputed using a normal distribution of width 0.5 at 1.8 standard deviation away from the median of the data distribution (as described in PMID: 27348712).
* `04 - C10_vs_Raw` This sheet contain the benchmaring of the single cell results to the bulk data for the C10 vs Raw comparison.
* `05 - SVEC_vs_Raw` This sheet contain the benchmaring of the single cell results to the bulk data for the SVEC vs Raw comparison.
* `06 - C10_vs_SVEC` This sheet contain the benchmaring of the single cell results to the bulk data for the C10 vs SVEC comparison.

We parsed the sheet 02 (`Raw sc protein data`) to an MSnSet without further modification (see script `scpdata/inst/script/dou2019.R`). 



