---
layout: post
title: "Nextflow stub run"
date: 2025-05-31
author: "Amit Indap"
tags:
  - Nextflow
---

I'm still be doing a lot Nextflow development lately for [PacBio HiFi](https://www.pacb.com/technology/hifi-sequencing/) WGS secondary analysis. 
My workflow farily comprehensive, starting from un-aligned reads in BAM format, performing reference guided alignment, and then proceeding with DNA short variant calling, CNV and structural variant calling, and CpG and 6mA methylation calling.

Today, I finally got around to adding a [stub section](https://www.nextflow.io/docs/latest/process.html) to my Nextflow processes to test and ensure the accuracy and worflow logic without having to run the actual process commands. 

For example, here is my process for running pbmm2 for alignment:

```groovy

process pbmm2_align {
    label 'high_memory'
    publishDir "${params.aligned_output_dir}", mode: 'copy'
    tag "$sample_id"
    container "quay.io/pacbio/pbmm2:1.17.0_build1"

    input:
    path reference
    tuple val(sample_id), path(bam)
    val threads
    val sort_threads

    output:
    tuple path("${sample_id}.aligned.bam"), path("${sample_id}.aligned.bam.bai"), emit: aligned_bam
    
    script:
    """
    pbmm2 --version
    pbmm2 align \\
        --sort \\
        -j $threads \\
        -J $sort_threads \\
        --preset HIFI \\
        --sample ${sample_id} \\
        --log-level INFO \\
        --unmapped \\
        --bam-index BAI \\
        $reference \\
        $bam \\
        ${sample_id}.aligned.bam

    
    """
}

```

To add a stub section, simply add the `stub` directive to the process:

```groovy

process pbmm2_align {
    label 'high_memory'
    publishDir "${params.aligned_output_dir}", mode: 'copy'
    tag "$sample_id"
    container "quay.io/pacbio/pbmm2:1.17.0_build1"

    input:
    path reference
    tuple val(sample_id), path(bam)
    val threads
    val sort_threads

    output:
    tuple path("${sample_id}.aligned.bam"), path("${sample_id}.aligned.bam.bai"), emit: aligned_bam
    //path "${$sample_id}.read_length_and_quality.tsv", emit: bam_rl_qual
    
    script:
    """
    pbmm2 --version
    pbmm2 align \\
        --sort \\
        -j $threads \\
        -J $sort_threads \\
        --preset HIFI \\
        --sample ${sample_id} \\
        --log-level INFO \\
        --unmapped \\
        --bam-index BAI \\
        $reference \\
        $bam \\
        ${sample_id}.aligned.bam

    
    """

    stub:
    """
    # Create mock aligned BAM file
    touch ${sample_id}.aligned.bam
    
    # Create mock BAM index file
    touch ${sample_id}.aligned.bam.bai
    
    """
}
```

To run the workflow with the stub section, simply use `-stub-run`

```
nextflow run main.nf -stub-run
```

Before, when I was running my workflow without stubs, I would have first have to find a smaller dataset to run the workflow on, and then execute the process commands. While, still better than testing with a full dataset, it would still be cumbersome to iron out bugs in workflow logic. Testing with stubs allows me to quickly iterate and test worflow logic. 

My only regret is that I wish I would starting using stubs earlier when implementing workflows. 

