---
title: "NGS data processing for precision treatments of brain tumors"
tags: [Bioinformatics, NGS]
categories:
  - Portfolio
date: 2019-07-01
---

### 1. Overview  
Large-scale tumor molecular profiling based on next-generation sequencing (NGS) technologies have revolutionized the field of precision oncology (Malone *et al*., 2020). I worked for the Institution for Refractory Cancer Research (IRCR) at Samsung Medical Center. One of the goals of the IRCR is to reveal the genetic mechanism that gives rise to glioblastoma (GBM), a malignant brain tumor, which can be used to inform therapeutic decisions in daily practice.  

### 2. GliomaSCAN  
In support of this effort, we designed a GBM-specific NGS panel called GliomaSCAN (Shin *et al*., 2020). I participated in advancing the NGS data pipeline and conducting molecular pathology and GMB oncology research to determine whether GliomaSCAN’s results were to other clinical evidence. Targeted gene panels like GliomaSCAN are more frequently used in clinical settings because they provide deeper coverage of selected areas of interest, have a faster turnaround time, and produce more clinically relevant data than broader genomic profiling accomplished using whole-exome sequencing (WES) or whole-genome sequencing (WGS) (Malone *et al*., 2020). Moreover, target-enrichment approaches are less costly than WES or WGS. The South Korean Ministry of Health and Welfare began covering NGS panels under South Korea’s national health insurance.   

### 3. Bioinformatics workflow  
Although there are some variations, tumor-specific molecular aberration profiling is conducted in three main steps.  
The first step involves dealing with FASTQ files produced by Illumina sequencers.  
These files can be processed using QC tools and aligned with reference sequences via Burrows-Wheeler transformations (Li & Durbin, 2009).  
The alignment results can then be saved as either sequence alignment map (SAM) files or binary SAM (BAM) files. The next step is profiling the tumor’s genotypes.  
SAM and BAM files must undergo alignment post-processing before the variant-calling step to remove artifacts created during PCR and local realigning sequence fragments. This processing can be accomplished by many open-source tools, such as SAMtools and Picard.   
<img src="/assets/img_ngs/Bioinformatics_variation.png" alt="bi" width="450" />   
The final step is variant-calling. Variants are single-nucleotide polymorphisms, insertion and deletion of nucleotide sequences. Mutect (Cibulskis *et al*., 2013) and GATK (McKenna *et al*., 2010) can be used for variant-calling. Note that GATK version 4 includes Mutect2 by default.  
For more information on how to validate the applicability of our GliomaSCAN process, please refer to our published paper[<i class="fas fa-paperclip"></i>](https://doi.org/10.4143/crt.2019.036){:target="_blank"}.  
  
### 4. Radiogenomics  
Many clinicians have paid attention to radiogenomics, a combination of genetic information and radiology (Ellingson, 2015). In oncology, radiogenomic approaches are used to generate information about the patient rather than making treatment decisions based on population data or clinical information, such as tumor stage, patient age, or patient gender (Lo Gullo *et al*., 2020). This type of approach is why I helped implement deep learning to solve issues in MR image processing.  
<img src="/assets/img_ngs/Biomarker.png" alt="biomarker" width="450" />  
In 2017, deep Learning-based brain tumor diagnosis methods were becoming popular. I observed radiologists at Samsung Medical Center manually segmenting GBM ranges from MR images. To eliminate this laborious task, I pioneered deep learning segmentation with IRCR scientists. My Github repo[<i class="fas fa-github"></i>](https://doi.org/10.4143/crt.2019.036){:target="_blank"} shows an example of how to process MR image data with Python.  
   
-----  
  
Cibulskis, K., Lawrence, M. S., Carter, S. L., Sivachenko, A., Jaffe, D., Sougnez, C., Gabriel, S., Meyerson, M., Lander, E. S., & Getz, G. (2013). Sensitive detection of somatic point mutations in impure and heterogeneous cancer samples. *Nature Biotechnology, 31*(3). https://doi.org/10.1038/nbt.2514  
  
Ellingson, B. M. (2015). Radiogenomics and Imaging Phenotypes in Glioblastoma: Novel Observations and Correlation with Molecular Characteristics. *Current Neurology and Neuroscience Reports, 15*(1). https://doi.org/10.1007/s11910-014-0506-0  
  
Li, H., & Durbin, R. (2009). Fast and accurate short read alignment with Burrows-Wheeler transform. *Bioinformatics, 25*(14). https://doi.org/10.1093/bioinformatics/btp324  
  
Lo Gullo, R., Daimiel, I., Morris, E. A., & Pinker, K. (2020). Combining molecular and imaging metrics in cancer: radiogenomics. *Insights into Imaging, 11*(1). https://doi.org/10.1186/s13244-019-0795-6  
  
Malone, E. R., Oliva, M., Sabatini, P. J. B., Stockley, T. L., & Siu, L. L. (2020). Molecular profiling for precision cancer therapies. *Genome Medicine, 12*(1). https://doi.org/10.1186/s13073-019-0703-1  
  
McKenna, A., Hanna, M., Banks, E., Sivachenko, A., Cibulskis, K., Kernytsky, A., Garimella, K., Altshuler, D., Gabriel, S., Daly, M., & DePristo, M. A. (2010). The Genome Analysis Toolkit: A MapReduce framework for analyzing next-generation DNA sequencing data. *Genome Research, 20*(9). https://doi.org/10.1101/gr.107524.110  
  
Shin, H., Sa, J. K., Bae, J. S., Koo, H., Jin, S., Cho, H. J., Choi, S. W., Kyoung, J. M., Kim, J. Y., Seo, Y. J., Joung, J.-G., Kim, N. K. D., Son, D.-S., Chung, J., Lee, T., Kong, D.-S., Choi, J. W., Seol, H. J., Lee, J.-I., … Nam, D.-H. (2020). Clinical targeted next-generation sequencing panels for detection of somatic variants in gliomas. *Cancer Research and Treatment, 52*(1). https://doi.org/10.4143/crt.2019.036
