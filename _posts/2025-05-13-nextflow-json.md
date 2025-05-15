---
layout: post
title: "Parsing JSON with Nextflow"
date: 2025-05-13
author: "Amit Indap"
tags:
  - SeqeraAI
  - Nextflow
  - Groovy
---

 I had a LinkedIn [post](https://www.linkedin.com/posts/aindap_github-nextflow-iopatterns-a-curated-activity-7324199462325669888-fwOM?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAASW-wBf3YPPqgSpB2CcVl-jJrrxf_XC94) a few weeks ago about design patterns in Nextflow described in this [repo](https://github.com/nextflow-io/patterns?tab=readme-ov-file) To my suprise, it went kinda viral by my modest standards - 40 likes and 2400 impressions. 

 One of the patterns in the in the repo is [executing a process/workflow on each record in a CSV file](https://github.com/nextflow-io/patterns/blob/master/docs/process-per-csv-record.md). It uses the [splitCSV](https://www.nextflow.io/docs/latest/reference/operator.html#splitcsv) operator that parses the rows of CSV file. 

 This week though, I had to find a solution where I needed to parse a JSON file to launch my pipeline. I'm not a Nextlflow / Groovy expert, but with the help of [SeqeraAI](https://seqera.io/ask-ai/chat), I was able to find a solution with [JsonSlurper](https://groovy-lang.org/processing-json.html#json_jsonslurper) to parse the JSON file.

 Assuming I have a JSON file that looks like this:

```json

{
    "organism": "human",
    "samples": [
	    {
           "sample_id": "1234",
           "vcf_file": "/path/to/1234.vcf.gz",
            "vcf_index_file": "/path/to/1234.vcf.gz.tbi"
           
        },
	    {
           "sample_id": "12345",
           "vcf_file": "/path/to/12345.vcf.gz",
            "vcf_index_file": "/path/to/12345.vcf.gz.tbi"
           
        }
	
    ]
}
```

It's pretty straightforward to parse the JSON and launch my process:

```groovy

import groovy.json.JsonSlurper

// Define the input parameters
params.json_file = 'file.json'

// Read and parse the JSON file
def jsonSlurper = new JsonSlurper()
def jsonData = jsonSlurper.parse(new File(params.json_file))

// Create a channel from the parsed JSON data
def samplesChannel = Channel.fromList(jsonData.samples)



// Process the samples
process processSamples {
    input:
    tuple val(sample_id), val(vcf_file), val(vcf_index_file)

    output:
    stdout

    script:
    """
    echo "Processing sample: ${sample_id}"
    echo "VCF file: ${vcf_file}"
    echo "VCF index file: ${vcf_index_file}"
    """
}

// Execute the workflow
workflow {
    samplesChannel
        .map { sample -> [sample.sample_id, sample.vcf_file, sample.vcf_index_file] }
        .set { samples }

    processSamples(samples)
        .view()

    
}
```

Running the above code will produce the following output:

```bash

Processing sample: 1234
VCF file: /path/to/1234.vcf.gz
VCF index file: /path/to/1234.vcf.gz.tbi

Processing sample: 12345
VCF file: /path/to/12345.vcf.gz
VCF index file: /path/to/12345.vcf.gz.tbi

```

And with that, my Nextflow journey continues ... 


