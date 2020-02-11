# Metagenomics

In the `run_dbcan_meta.py` script we made several changes to the run_dbcan script.

These include:

**1. Corrected typos in using fraggene scan to call genes on reads/assemblies**

Mis-spelling complete as "comlete"
https://github.com/linnabrown/run_dbcan/blob/0ee6f21f8b61940743bbcfcc1d3184968b631c7b/run_dbcan.py#L140

Same line was breaking because of the spaces in the subprocess call but when I switched it to below where there are no spaces after flags and their parameters it worked.

```call(['FragGeneScan', '-s', input, '-o', '%sfragGeneScan'%outPath,'-w','1','-t','complete','-p','10'])```

**2. Functionality in fraggenescan to choose CPUs and use reads or metagenomic assemblies**

CPU usage: `--frag_cpus` default is 10
Reads or assemlies: Option to use 1 (for metagenomes) or 0 (for reads)

**3. An identity threshold cutoff for DIAMOND**

This is very important for reads.

Investigation in DIAMOND results suggested for us that there is a clear peak of hits at and above 97% identity, and the rest is noise, so we recommend `--dia_id 97`; we also found that the `--dia_eval 1e-5` works well with reads and with defaults you get zero hits due to 

When running on reads, we noticed that decreasing speed on ~10 million (~300bp merged) reads on an EC2 instance with 16 threads and 64k memory goes:
DIAMOND only (~1.5 hours), DIAMOND + HotPep (~10 hours), DIAMOND + HotPep + HMMER (66 hours)

For more than ~10 million reads, we required an instance 128K memory. An example command is:

`python run_dbcan_meta.py ${sample}.fasta meta --out_dir ${sample} --out_pre ${sample}_ --tools hotpep diamond --dia_eval 1e-5 --hotpep_cpu 8 --dia_cpu 8 --dia_id 97`
