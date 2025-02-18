# MERFISH: Multiplexed error-robust fluorescence in situ hybridization
### By Nick Monell, Benjamin Esser and Tusha Karnani

## Contents
* [What is MERFISH?](#what-is-merfish)
* [Procedure](#procedure)
* [Computational Decoding](#computational-decoding)
* [Validation](#validation)
* [Output](#output)
* [Applications of MERFISH](#applications-of-merfish)
* [Conclusion](#conclusion)
* [References](#references)

## What is MERFISH?
MERFISH is an imaging technique and spatial transcriptomics technology that measures RNA expression and and location in a tissue. It was pubished in 2015 by Chen et al in Science.

MERFISH is built upon small molecule fluorescence in situ hybridization (smFISH) technology, but uses a combinatorial barcoding scheme, which allows for imaging of thousands of RNA species at once at single-cell resolution.

<div align="center">
<img src="https://raw.githubusercontent.com/tkarnani/BENG183_2024Fall_Applied-Genomic-Technologies/main/Final_Paper/Group_9_MERFISH/Images/overview.jpeg" width="45%" style="display: block; margin: auto;"/>
  
Figure 1: Overview of MERFISH process and output. <b> Figure from Kok Hao Chen et al.</b> <i>Science</i> <b>348</b>, aaa6090 (2015). <a href="https://doi.org/10.1126/science.aaa6090">DOI:10.1126/science.aaa6090</a>
</div>

## Procedure
At a very high level, MERFISH involves only a few steps. First, the sample is treated with encoding probes which will bind to specific mRNA. Next, the sample is treated with readout probes which will bind to specific encoding probes and emit fluorescence, which is imaged and can then be decoded to detect and quantify RNA species while preserving spatial information.

<div align="center">
<img src="https://raw.githubusercontent.com/tkarnani/BENG183_2024Fall_Applied-Genomic-Technologies/main/Final_Paper/Group_9_MERFISH/Images/probes.jpeg" width="45%" style="display: block; margin: auto;"/>
  
Figure 2: Encoding probes, made up of target and readout sequences, bind RNA species as well as readout probes over successive rounds of readout hybridization. <b> Figure from Moffit et al.</b> <i>PNAS</i> <b>113</b>, 1612826113 (2016). <a href="https://doi.org/10.1073/pnas.1612826113">DOI:10.1073/pnas.1612826113</a>
</div>

Encoding probes contain
  1. Target sequence: RNA sequence complementary to the mRNA that encoding probe is measuring
  2. Readout sequences: RNA sequence complementary to a specific fluorescently labeled readout probe

All of the encoding probes with the same target sequence, when put together, will have a unique combination of readout sequences and will therefore bind a unique combination of fluorescently labeled readout probes. Therefore, each mRNA species that we want to measure will have its own fluorescent fingerprint that we can use to identify it.

After binding our encoding probes to the sample's mRNA, we treat our sample with each of our fluorescently labeled readout probes.
Between readout probes, we take an image to record the fluorescence coming from the sample and then treat with photobleach to wash away the bound readout probes.

## Computational Decoding
MERFISH utilizes a high-throughput encoding system to differentiate between hundreds to thousands of RNA species in a single experiment. The encoding system is as follows:

* We will assign an 'n-bit' binary string to each RNA species where n is the number of imaging rounds in our procedure. Each round will correspond to either a 1 (fluorescence detected) or 0 (no fluorescence) on an RNA based on if a fluorescently labeled probe is attached to that RNA. For example, if we wanted to encode the gene ACTC1 and used 12 imaging rounds, we would use a string of twelve 1's and 0's that may look like `010010101000`, indicating that we expect a fluorescent readout in imaging rounds 2, 5, 7, and 9 at every spatial position where the ACTC1 gene is located
* Important to note, RNA may be scattered everywhere on the surface we are imaging. If the goal is to determine both the count and spatial location of many different types of RNA, we must first locate where each RNA is (generally) and then identify which species is in each position. Locating the general RNA position is as easy as determining wherever there is a fluorescent peak in any imaging round. Determining which RNA it is depends on what readout we see over all imaging rounds for that position.
  
<div align="center">
<img src="https://raw.githubusercontent.com/tkarnani/BENG183_2024Fall_Applied-Genomic-Technologies/main/Final_Paper/Group_9_MERFISH/Images/encoding.jpg" width="45%" style="display: block; margin: auto;"/>

Figure 3: Demonstration of detected RNA spots (general) and how to decode each spot into an RNA species based on fluorescent readout over 16 hybridization rounds. <b> Figure from Kok Hao Chen et al.</b> <i>Science</i> <b>348</b>, aaa6090 (2015). <a href="https://doi.org/10.1126/science.aaa6090">DOI:10.1126/science.aaa6090</a>
</div>

* Considering all possible n-length binary strings, we can encode $2^n - 1$ gene species using this system (removing 1 for the string containing only 0's as this would be an undetectable position). This makes it highly scalable for multiplexing. This means the minimum number of imaging rounds we must perform, n, is $\log_2(x + 1)$ where x is the number of gene species we are interested in.
* For our encoding system, we will apply some "rule" to regulate which binary strings we can use to encode RNA species. Remember, MERFISH is Error Robust meaning if a probe doesn't bind properly in a particular hybridization round or fluorescent readout is unresolvable, we should still be able to determine which RNA species was there based on the rest of our code. One such system is Modified Hamming Distance 4 (MDH4), consisting of n-bit binary strings with a maximum of 4 'fluorescence-on' (1) bits that are at least different from any other string by 4-bit positions. Valid codes would be `000000001111` or `100100100100` while an invalid code may be `110001101010`. Because we know that each code is supposed to have 4 on-bits if there is any single-bit error we can correct it by referencing the readout with our codebook, finding which string position the error is in, and decoding which RNA species it is. 
The benefit of encoding and decoding in MERFISH is the redundancy and error correction, allowing us to resolve the count and position of every RNA species even in extensive datasets.
## Validation
The results of decoding should be validated to ensure we are confident in the RNA counts for each species.
#### Control Threshold
Within the encoding scheme designed for the experiment, encode several 'control words', or n-bit strings that follow the same encoding rules (MDH4, Reed-Muller, BCH, etc.) but do not encode for any RNA species. This means that we do not have any designed probe sequence for these codes, so their respective fluorescent readout should not appear in the data or we have made a misidentification. Generate a confidence score for each decoded readout, the maximum confidence for our control words is the confidence threshold we will use to flag other RNA detection. Any spots that have a confidence lower than this threshold are unreliable as they achieve the same confidence as known errors. 

<div align="center">
<img src="https://raw.githubusercontent.com/tkarnani/BENG183_2024Fall_Applied-Genomic-Technologies/main/Final_Paper/Group_9_MERFISH/Images/validation.jpg" width="30%" style="display: block; margin: auto;"/>

Figure 4: Comparison of confidence ratio between detected RNA (blue) and control words (red) with dashed line representing maximum confidence ratio of control words. <b> Figure from Kok Hao Chen et al.</b> <i>Science</i> <b>348</b>, aaa6090 (2015). <a href="https://doi.org/10.1126/science.aaa6090">DOI:10.1126/science.aaa6090</a>
</div>

#### RNA-Seq Reference
Use the counts for each RNA species as a form of validation. Compare the actual RNA counts for each species with a known method of RNA-seq. If copy numbers show a strong correlation, we have strong evidence of the method's accuracy.

## Output
The MERFISH approach allows parallelization of measurements of many individual RNA species and covariation analysis between different RNA species. 

#### Expression Covariation
Analysis of covariations in the expression levels of different genes can reveal which genes are coregulated and elucidate gene regulatory pathways. Using a heirarchical clustering approach, genes can be grouped based on the covariation of their expression analysis data. This can help recognize groups with substantially correlated expression patterns, i.e. they have more correlation in expression patterns with genes in the group compared to those outside. Gene ontology (GO) enrichment analysis can then further look into the functions of these genes and help identify correlated genes/pathways.
<div align="center">
<img src="https://raw.githubusercontent.com/tkarnani/BENG183_2024Fall_Applied-Genomic-Technologies/main/Final_Paper/Group_9_MERFISH/Images/correlation.png" width="45%" style="display: block; margin: auto;"/>

Figure 5: Matrix of the pairwise correlation coefficients of the cell-to-cell variation in expression for the measured genes, shown together with the hierarchical clustering tree. Two of the seven groups are enlarged on the right. <b> Figure from Kok Hao Chen et al.</b> <i>Science</i> <b>348</b>, aaa6090 (2015). <a href="https://doi.org/10.1126/science.aaa6090">DOI:10.1126/science.aaa6090</a>
</div>

#### Spatial Distribution
Some RNA transcripts enriched in the perinuclear region, some enriched in the cell periphery, and some scattered throughout the cell.
It determines the correlation coefficients for the spatial density profiles of all pairs of RNA species and organized these RNAs according to the pairwise correlations again using a hierarchical clustering approach.
The spatial pattern that observed reflects their cotranslational enrichment at the ER since they pass through the same/similar secretion pathways.
<div align="center">
<img src="https://raw.githubusercontent.com/tkarnani/BENG183_2024Fall_Applied-Genomic-Technologies/main/Final_Paper/Group_9_MERFISH/Images/spatial.jpeg" width="60%" style="display: block; margin: auto;"/>

Figure 6: Distinct spatial distributions of RNAs observed in the 140-gene measurements. <b> Figure from Kok Hao Chen et al.</b> <i>Science</i> <b>348</b>, aaa6090 (2015). <a href="https://doi.org/10.1126/science.aaa6090">DOI:10.1126/science.aaa6090</a>
</div>

After post-processing, MERFISH data can take various forms, for example represented as a 2D image with a large number of channels (associated with the measured gene set), as a list of points in space with attached gene count information (reconstructing single-cell RNAseq information combined with location), or simply as a long list of single mRNA molecules with their detected location.

## Applications of MERFISH
MERFISH (Multiplexed Error-Robust Fluorescence In Situ Hybridization) has revolutionized spatial transcriptomics by enabling the high-throughput and spatially resolved analysis of gene expression. Its ability to detect thousands of RNA species while preserving spatial context has found applications across various fields of biology and medicine.

<div align="center">
<img src="https://raw.githubusercontent.com/tkarnani/BENG183_2024Fall_Applied-Genomic-Technologies/main/Final_Paper/Group_9_MERFISH/Images/applications.png" width="45%" style="display: block; margin: auto;"/>

Figure 7: Left- Spatial distribution of all cell clusters in one mouse brain coronal section. Right- Spatial distribution of identified cells across a human lung cancer sample. <b> Figure from Developing Spatial Transcriptomics Data Analysis Solutions to Empower Researchers </b> <i>Vizgen</i>
</div>

#### Developmental Biology
- **Tissue Morphogenesis**: MERFISH reveals how gene expression drives the formation and differentiation of tissues during embryonic development.
- **Cell Lineage Tracing**: It enables the study of how single cells contribute to tissue formation, providing a spatial view of developmental trajectories.
#### Cancer Research
- **Tumor Microenvironment**: MERFISH maps gene expression in tumor and surrounding stromal cells, providing a spatial understanding of the tumor microenvironment.
It identifies interactions between cancer cells, immune cells, and other stromal components.
- **Understanding Metastasis**: By spatially resolving gene expression in metastatic tumors, MERFISH reveals pathways that cancer cells use to invade new tissues.
- **Therapeutic Targeting**: It helps pinpoint specific cell types or molecular pathways for targeted therapies, such as immune checkpoint inhibitors.
#### Infectious Diseases and Immunology
- **Host-Pathogen Interactions**: It helps study how pathogens interact with host cells, revealing spatial patterns of immune activation or suppression.
- **Response to Vaccines**: It helps assess immune responses to vaccines in different tissues by tracking spatial gene expression changes.

## Conclusion
MERFISH exemplifies the evolution of biological research tools, moving beyond standard sequencing to address not just the "what" but also the "where" of gene expression. This spatial resolution allows researchers to explore how cells interact within their native environments and how these interactions change during development, disease, or treatment.

## References
1. [Kok Hao Chen et al., Spatially resolved, highly multiplexed RNA profiling in single cells, Science](https://www.science.org/doi/10.1126/science.aaa6090)
2. [Miller et al., Space-feature measures on meshes for mapping spatial transcriptomics, Medical Image Analysis](https://www.sciencedirect.com/science/article/pii/S1361841523003286?casa_token=ZqXh4KXN5sEAAAAA:QbjS1rtmAQC0Vo4oWw7EceIGqjlB2ZW4n4OvEGTubU225DrdeRs8NogIdwxOdNJ_H1dix1b4mw#b73)
3. [Developing Spatial Transcriptomics Data Analysis Solutions to Empower Researchers, Vizgen](https://vizgen.com/developing-spatial-transcriptomics-data-analysis-solutions-to-empower-researchers/)
4. [Moffitt et al., High-throughput single-cell gene-expression profiling with multiplexed error-robust fluorescence in situ hybridization, PNAS](https://doi.org/10.1073/pnas.1612826113)
