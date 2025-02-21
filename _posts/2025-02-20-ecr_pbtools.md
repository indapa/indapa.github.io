---
layout: post
title: "Docker images with PacBio secondary analysis tools"
date: 2025-02-20
author: "Amit Indap"
tags:
  - PacBio
  - Docker
---

I've been working a lot with PacBio Hifi data lately. 
After having zero experience with PacBio secondary analysis, I've implemented a [WGS](https://github.com/indapa/nextflow-hifi-wgs) and [RNA-seq](https://github.com/indapa/nextflow-isoseq-indapa) Nextflow pipeline. 

Those pipelines utilize [Seqera Wave containers](https://seqera.io/wave/), that are built on the fly as the pipeline runs.

But today I made a public Docker image with the PacBio secondary analysis tools that includes the following tools:

- [pbmm2](https://github.com/PacificBiosciences/pbmm2)

- [hificnv](https://github.com/PacificBiosciences/HiFiCNV)

- [pbvsv](https://github.com/PacificBiosciences/pbsv)

- [trgt](https://github.com/PacificBiosciences/trgt)

- [pb-CpG-tools](https://github.com/PacificBiosciences/pb-CpG-tools)

Here is the URI for the Docker image:

```

public.ecr.aws/q3t0i2q1/indapa-pbtools:latest

```

Feel free to use it in your Nextflow pipelines or other workflows.