---
layout: post
title: "nf-core"
date: 2025-09-21
author: "Amit Indap"
tags:
  - Nextflow
  - nf-core
  - GCP
---

I've been writing Nextflow workflows for about a year now, but suprisingly hadn't run any of the [nf-core](https://nf-co.re/) workflows until recently. Well, I finally got a chance to run the [nf-core/sarek](https://nf-co.re/sarek) to generate pre-processed BAM files starting from raw FASTQ files. 

I used a combination of  [Seqera AI](https://seqera.io/ask-ai/) and reading the [nf-core/sarek documentation](https://nf-co.re/sarek/usage) to get started. I wasn't interested in doing any variant calling, just wanted to get the pre-processed BAM files. (Meaning alignment, sorting, marking duplicates, base quality score recalibration). Additionally, I had a specific genome reference FASTA file I needed to use, so I had to specify that as well. From my config file:

```
params {
    input = 'gs://sarek-samplesheets/gcp-fastqs-samplesheet.csv'
    outdir = 'gs://sarek-results'

    genome = null
    igenomes_ignore = true
    fasta = 'gs://dog-pipeline-ref-fasta/canFam4.fa'
    fasta_fai = 'gs://dog-pipeline-ref-fasta/canFam4.fa.fai'  // Add this
    //bwa = 'gs://dog-pipeline-ref-fasta'  // Add this - your BWA index directory
    //dict = 'gs://dog-pipeline-ref-fasta/canFam4.dict'  // Add this

    dbsnp = 'gs://dog-ref-fasta/AutoAndXPAR.SNPs.CanineHD.vcf.gz'
    dbsnp_tbi = 'gs://dog-ref-fasta/AutoAndXPAR.SNPs.CanineHD.vcf.gz.tbi'

    // Alignment-only workflow
    // Stop after duplicate marking to avoid variant calling preparation
    tools = null
    save_mapped = true
    save_output_as_bam = true

    skip_tools = 'multiqc'  
}
```

The values for `fasta`, `fasta_fai`, and `dbsnp` were all files I had previously uploaded to a GCS bucket. I also had to set `igenomes_ignore = true` since I wasn't using any of the pre-defined iGenomes references.


Since I was running on GCP, I had to add the following to my config file:

```
// Google Cloud configuration
google {
    project = ''
    region = 'us-central1'
    
    batch {
        spot = true
        project = ''
        diskSize = '100 GB'
        taskPollingInterval = '60 sec'
        timeout = '15 min'
        serviceAccountEmail = ''
        stagingTransfer = true
    }
}

workDir = 'gs://dd-sarek-workdir'

// Logging
log.level = 'DEBUG'

// Capture execution reports
timeline {
    enabled = true
    file = "${params.outdir}/pipeline_info/execution_timeline.html"
    overwrite = true
}
report {
    enabled = true
    file = "${params.outdir}/pipeline_info/execution_report.html"
    fields = 'task_id,hash,native_id,name,status,exit,submit,start,complete,duration,realtime,cpu,memory,disk,read_bytes,write_bytes,vol_ctxt,inv_ctxt,env,workdir,script,tag'
    overwrite = true
}
trace {
    enabled = true
    file = "${params.outdir}/pipeline_info/execution_trace.txt"
    overwrite = true
}
dag {
    enabled = true
    file = "${params.outdir}/pipeline_info/pipeline_dag.html"
    overwrite = true
}
```

The actually values have been redacted for privacy, but you get the idea. Additionally, I enabled [fusion](https://seqera.io/fusion/) and [wave](https://seqera.io/wave/) in the config file:
```

docker {
    enabled = true
    
}

// Fusion and Wave configuration
fusion {
    enabled = true
}

wave {
    enabled = true
    strategy = ['conda','container']  // Add strategy
}
```

It was pretty straightforward to run the workflow on GCP using the following command:

```
nextflow run nf-core/sarek -c nextflow-sarek.config -profile docker

```


And boom! I had pre-processed BAM files in my output GCS bucket. 
Sarek is very flexible and ran it on dog, cat, and human samples this past week with no issues. 

nf-core is great, because it has a lot of other pre-built workflows for taxonomic profiling and genotype imputation, which I will have run recently as well. I'll write about those workflows in future posts.





