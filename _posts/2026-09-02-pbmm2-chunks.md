---
layout: post
title: "Scatter-Gather PacBio HiFi Alignments in Nextflow Using pbmm2 Native Chunking"
date: 2026-09-01
author: "Amit Indap"
tags:
  - Nextflow
  - PacBio
  - pbmm2
  - AWS
---


I've been trying to cost-optimize my PacBio HiFi nextflow pipelines. Having monolithic, compute intensive processes like alignment or variant calling is cumbersome and expensive, becasue if it fails, you have to start all over again. Additionaly, I usually run them using on-demand instances on AWS because the chances of getting pre-empted are greater. 

[nf-core/sarek](https://github.com/nf-core/sarek) splits input fastq files into smaller chunks with the help of [fastp](https://github.com/OpenGene/fastp). The sharded fastq chunks are mapped separtely and then merged back before proceeding to downstream steps. 

I've been been wanting to use the same approach for PacBio HiFi alignments. After some investigation, [version 26.2.99 of pbmm2](https://github.com/PacificBiosciences/pbmm2/releases/tag/v26.2.99) has a ```chunk``` and ```chunk-mode``` option:

```
--chunk                    STR    Align only chunk K of N ZMWs (1-based, e.g. 2/5).
                                  Single, unaligned, indexed(.pbi/.zmi) BAM input only.

--chunk-mode               STR    Chunk selection with --chunk: 'contiguous' (default) or 'scatter' (sample ZMWs
                                    across the whole file, chaotic but deterministic). Valid choices: (contiguous,
                                    scatter). [contiguous]
```


So these options provide built in scatter/chunking capability, so you can parallelize the alignment of a single uBAM across mutiple compute nodes (or in my case AWS spot instances) Even better is you don't have physically split the uBAM on disk.  

You can see my nextflow subworkflow PBMM2_SPOT_WGS [here](https://github.com/indapa/nextflow-hifi-wgs/blob/master/subworkflows/reference_alignment/main.nf#L15) with the relevant process definitions [here](https://github.com/indapa/nextflow-hifi-wgs/blob/master/modules/pbtools/main.nf#L77)

I was able to align a 30X HG002 uBAM from [PacBio](https://www.pacb.com/connect/datasets/) in 1h32min for an estimated cost of $2.10



![nf-metrics](/docs/assets/pbmm2_spot_metrics.png)

![nf-dag](/docs/assets/pbmm2_dag.png)


