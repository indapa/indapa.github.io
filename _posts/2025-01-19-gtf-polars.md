Bioinformatics has a whole ecosystem of file formats. Gencode hosts GTF files for [human gene annotations](https://www.gencodegenes.org/human/) GTF stands for Gene Tranfer Format, defined by 9 
tab delimited columns, as described [here](https://biocorecrg.github.io/PhD_course_genomics_format_2021/gtf_format.html):

```
Column number	Column name	Details

1	seqname	name of the chromosome or scaffold; chromosome names can be given with or without the ‘chr’ prefix.
2	source	name of the program that generated this feature, or the data source (database or project name)
3	feature	feature type name, e.g. Gene, Variation, Similarity
4	start	Start position of the feature, with sequence numbering starting at 1.
5	end	End position of the feature, with sequence numbering starting at 1.
6	score	A floating point value.
7	strand	defined as + (forward) or - (reverse).
8	frame	One of ‘0’, ‘1’ or ‘2’. ‘0’ indicates that the first base of the feature is the first base of a codon, ‘1’ that the second base is the first base of a codon, and so on..
9	attribute	A semicolon-separated list of tag-value pairs, providing additional information about each feature.

```

I needed to parse GTF file from my [isoseq](https://isoseq.how/)  pipeline because I needed to map transcript id to gene name. With the help of
[GitHub Copilot Agent Mode](https://github.com/features/copilot/agents) to implement a GTF parser that uses [Polars Lazy API](https://docs.pola.rs/user-guide/concepts/lazy-api/)
The repo is [here](https://github.com/indapa/gtf-polars) and the relevant code is below:

```
  lf = pl.scan_csv(
        file_path,
        separator="\t",
        comment_prefix="#",
        has_header=False,
        new_columns=gtf_columns_list,
        infer_schema=False,
    )
```
So instead of [read_csv](https://docs.pola.rs/api/python/dev/reference/api/polars.read_csv.html), [scan_csv](https://docs.pola.rs/api/python/dev/reference/api/polars.scan_csv.html)
the query only evaluated, once its collected. 

```
from gtf_polars import parse_gtf
import polars as pl

lf = parse_gtf("gencode.v39.annotation.sorted.gtf", attributes_to_extract=["gene_id", "gene_name"])

df = (lf.filter(pl.col("feature") == 'transcript').select(['seqname', 'start','end','gene_id', 'gene_name']).collect())
```

Once we call ```collect``, Polars optimizes the execution plan behind the scenes to return the data frame we need. 

If you call [explain](https://docs.pola.rs/api/python/stable/reference/lazyframe/api/polars.LazyFrame.explain.html), you can see execution plan of how Polars plans its query:

```
print(lf.explain())

WITH_COLUMNS:
 [col("start").strict_cast(Int64), col("end").strict_cast(Int64), when([(col("score")) == (".")]).then(null.cast(String)).otherwise(col("score")).strict_cast(Float64).alias("score"), when([(col("frame")) == (".")]).then(null.cast(String)).otherwise(col("frame")).alias("frame"), col("attributes").str.extract(["(?:^|;\s*)gene_id\s+"([^"]*)""]).alias("gene_id"), col("attributes").str.extract(["(?:^|;\s*)gene_name\s+"([^"]*)""]).alias("gene_name"), col("attributes").str.extract(["(?:^|;\s*)transcript_id\s+"([^"]*)""]).alias("transcript_id")] 
  Csv SCAN [gencode.v39.annotation.sorted.gtf]
  PROJECT */9 COLUMNS

```

Reading this plan from the bottom up, Polars first registers the raw GTF data source (`Csv SCAN`) and prepares to project the required columns.
As the data flows upward into the `WITH_COLUMNS` step, Polars bundles type casting, handling null ('.') columns, and  regex string extraction into a single, highly parallelized expression layer. 

Crucially, because we are using Polars' Lazy API, this entire layout is just a blueprint. 
When we finally chain our downstream filters and selections, Polars applies [optimizations](https://docs.pola.rs/user-guide/lazy/optimizations/) like **projection pushdown** (only loading the columns we ask for) and **predicate pushdown**. 
Predicate pushdown is an optimization technique where Polars moves your filter criteria (`.filter()`) as close to the physical data source as possible.
Instead of loading the entire  GTF file into memory and filtering for transcripts afterward, Polars pushes that condition down to the file-reader level.
Rows that aren't transcripts are dropped on the fly while the file is being streamed, drastically slashing both memory usage and execution time.

Comparing eager vs. lazy evaluation in Polars, along with Pandas:

```
import time
import pandas as pd
import polars as pl
from gtf_polars import parse_gtf

file_path = "/Users/indapa/2X_health/Sandbox/IsoSeq/gencode.v39.annotation.sorted.gtf"
attributes = ["gene_id", "gene_name"]
gtf_columns = ["seqname", "source", "feature", "start", "end", "score", "strand", "frame", "attributes"]

# ==========================================
# 1. Benchmark Pandas (Strictly Eager)
# ==========================================
start_pd = time.perf_counter()

# Read entire file, filter for transcripts, and parse out attributes using regex
df_pd = pd.read_csv(
    file_path,
    sep="\t",
    comment="#",
    header=None,
    names=gtf_columns,
    dtype=str,  # Keep as string initially to match baseline parsing
)
df_pd_filtered = df_pd[df_pd["feature"] == "transcript"].copy()

# Standard Pandas extraction pattern for the blog comparison
df_pd_filtered["gene_id"] = df_pd_filtered["attributes"].str.extract(r'gene_id\s+"([^"]*)"')
df_pd_filtered["gene_name"] = df_pd_filtered["attributes"].str.extract(r'gene_name\s+"([^"]*)"')
df_pd_final = df_pd_filtered[["seqname", "start", "end", "gene_id", "gene_name"]]

end_pd = time.perf_counter()
pd_duration = end_pd - start_pd


# ==========================================
# 2. Benchmark Polars Eager (read_csv)
# ==========================================
start_pl_eager = time.perf_counter()

df_pl_eager = pl.read_csv(
    file_path,
    separator="\t",
    comment_prefix="#",
    has_header=False,
    infer_schema=False,
)
# (Followed by eager filtering and expressions...)

end_pl_eager = time.perf_counter()
pl_eager_duration = end_pl_eager - start_pl_eager


# ==========================================
# 3. Benchmark Polars Lazy (scan_csv + collect)
# ==========================================
start_pl_lazy = time.perf_counter()

lf = parse_gtf(file_path, attributes_to_extract=attributes)
df_pl_lazy = (
    lf.filter(pl.col("feature") == "transcript")
    .select(["seqname", "start", "end", "gene_id", "gene_name"])
    .collect()
)

end_pl_lazy = time.perf_counter()
pl_lazy_duration = end_pl_lazy - start_pl_lazy


# ==========================================
# Print Results
# ==========================================
print(f"Pandas (Eager)         : {pd_duration:.4f} seconds")
print(f"Polars Eager (read_csv): {pl_eager_duration:.4f} seconds")
print(f"Polars Lazy (scan_csv) : {pl_lazy_duration:.4f} seconds")
```

We get the following:

| Framework / Mode | Execution Time | Speedup vs. Pandas | Performance Paradigm |
| :--- | :--- | :--- | :--- |
| **Pandas** (Strictly Eager) | 12.8303s | Baseline (1x) | Single-threaded, loads everything upfront. |
| **Polars Eager** (`read_csv`) | 7.7018s | **1.7x faster** | Multithreaded Rust engine, but still pays the full memory tax. |
| **Polars Lazy** (`scan_csv`) | **1.4517s** | **8.8x faster** | Streaming execution graph with predicate & projection pushdowns. |





