---
title: "Orthology Inference using orthologr"
date: "`r Sys.Date()`"
output: rmarkdown::html_vignette
vignette: >
  %\VignetteIndexEntry{Orthology Inference using orthologr}
  %\VignetteEngine{knitr::rmarkdown}
  %\usepackage[utf8]{inputenc}
---

```{r, echo = FALSE, message = FALSE}
options(width = 750)
knitr::opts_chunk$set(
  comment = "#>",
  error = FALSE,
  tidy = FALSE)
```

Orthology Inference is the process of detecting [orthologous genes](https://www.biostars.org/p/1595/) between a query organism and
a set of subject organisms. Motivated by the discussion on which orthology inference method, paradigm or program is 
the [most accurate](https://www.biostars.org/p/7568/) one, the `orthologr` package provides several interface functions to common
orthology inference tools such as [OrthoMCL](http://www.orthomcl.org/orthomcl/), [ProteinOrtho](https://www.bioinf.uni-leipzig.de/Software/proteinortho/), and [BLAST](http://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastDocs&DOC_TYPE=Download) _best reciprocal hit_, `OMA`, [FASTA](http://fasta.bioch.virginia.edu/fasta_www2/fasta_list2.shtml), and [HMM based methods](http://hmmer.janelia.org/). Future versions of the `orthologr` package will also implement interfaces to non-BLAST based methods such as
[MetaPhOrs](http://orthology.phylomedb.org/), [EnsemblCompara](http://www.ensembl.org/info/docs/api/compara/index.html), [PhylomeDB](http://phylomedb.org/), etc and will allow to benchmark the goodness of the corresponding orthology inference run via [Orthology Benchmarking](http://orthology.benchmarkservice.org/cgi-bin/gateway.pl).

To perform orthology inference you can start with the `orthologs()` function provided by `orthologr`.
The `orthologs()` function takes nucleotide or protein sequences stored in fasta files for a set of organisms 
and performs orthology inference to detect orthologous genes within the given organisms based on selected orthology inference programs.

The following interfaces are (yet) implemented in the `orthologs()` function:

### BLAST based methods:

- BLASTp best hit (BH)

- BLASTp reciprocal best hit (RBH)

- DELTA-BLAST reciprocal best hit (DELTA)

- ProteinOrtho

Using a simple example stored in the package environment of `orthologr` you
can get an impression on how to use the `orthologs()` function.

__Note:__ it is assumed that when using `orthologs()` all corresponding programs you
want to use are already installed on your machine and are executable via either
the default execution PATH or you specifically define the location of the executable file
via the `path` argument that can be passed to `orthologs()`.

```{r,eval=FALSE}

library(orthologr)

# perform orthology inference using BLAST reciprocal best hit
# and fasta sequence files storing protein sequences
orthologs(query_file      = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
          subject_files   = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
          seq_type        = "protein", 
          ortho_detection = "RBH", 
          clean_folders   = FALSE)


```


```
      query_id            subject_id evalue
1  AT1G01010.1 333554|PACid:16033839  0e+00
2  AT1G01020.1 470181|PACid:16064328 7e-166
3  AT1G01030.1 470180|PACid:16054974  0e+00
4  AT1G01040.1 333551|PACid:16057793  0e+00
5  AT1G01050.1 909874|PACid:16064489 2e-160
6  AT1G01060.3 470177|PACid:16043374  0e+00
7  AT1G01070.1 918864|PACid:16052578  0e+00
8  AT1G01080.1 909871|PACid:16053217 1e-178
9  AT1G01090.1 470171|PACid:16052860  0e+00
10 AT1G01110.2 333544|PACid:16034284  0e+00
11 AT1G01120.1 918858|PACid:16049140  0e+00
12 AT1G01140.3 470161|PACid:16036015  0e+00
13 AT1G01150.1 918855|PACid:16037307 3e-150
14 AT1G01160.1 918854|PACid:16044153  1e-93
15 AT1G01170.2 311317|PACid:16052302  3e-54
16 AT1G01180.1 909860|PACid:16056125  0e+00
17 AT1G01190.1 311315|PACid:16059488  0e+00
18 AT1G01200.1 470156|PACid:16041002 3e-172
19 AT1G01210.1 311313|PACid:16057125  7e-76
20 AT1G01220.1 470155|PACid:16047984  0e+00

```


This small example returns 20 orthologous genes between _Arabidopsis thaliana_ and _Arabidopsis lyrata_.
As you can see, the `query_file` and `subject_files` arguments take the proteomes of _Arabidopsis thaliana_ (`query_file`) and _Arabidopsis lyrata_ (`subject_files`) stored in fasta files. The `seq_type` argument specifies that you will pass protein sequences (proteomes)
to the `orthologs()` function. In case you only have either genomes (DNA sequences) or CDS files, you can
modify the `seq_type` argument to `seq_type = "dna"` (when working with only genome data) or `seq_type = "cds"` (when working with CDS files).
Internally the `orthologs()` function will perform a CDS prediction using `predict_cds()` and will furthermore translate the predicted
CDS sequences into protein sequences. Analogously when `seq_type = "cds"` is specified, internally the `orthologs()` function will
translate all CDS sequences into protein sequences to run orthology inference based on protein sequences.

__Note__: future versions of `orthologr` will allow to perform orthology inference using DNA sequences.
Nevertheless, since most orthology inference methods or paradigms rely on protein sequences, the first version
of `orthologr` will follow this paradigm.

In case you have to specify the path to the corresponding orthology inference method 
you can use the `path` argument as follows:



```{r,eval=FALSE}

library(orthologr)

# using an external execution path
orthologs(query_file      = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
          subject_files   = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
          seq_type        = "protein", 
          ortho_detection = "RBH", 
          path            = "here/path/to/blastp", 
          clean_folders   = FALSE,
          comp_cores      = 1)


```


When you are working on a multicore machine, you can also specify the `comp_cores`
argument that will allow you to run all analyses in parallel (to speed up computations).


```{r,eval=FALSE}

library(orthologr)

# running orthology inference in parallel using 2 cores
orthologs(query_file      = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
          subject_files   = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
          seq_type        = "protein", 
          ortho_detection = "RBH", 
          clean_folders   = FALSE,
          comp_cores      = 1)


```


```

      query_id            subject_id evalue
1  AT1G01010.1 333554|PACid:16033839  0e+00
2  AT1G01020.1 470181|PACid:16064328 7e-166
3  AT1G01030.1 470180|PACid:16054974  0e+00
4  AT1G01040.1 333551|PACid:16057793  0e+00
5  AT1G01050.1 909874|PACid:16064489 2e-160
6  AT1G01060.3 470177|PACid:16043374  0e+00
7  AT1G01070.1 918864|PACid:16052578  0e+00
8  AT1G01080.1 909871|PACid:16053217 1e-178
9  AT1G01090.1 470171|PACid:16052860  0e+00
10 AT1G01110.2 333544|PACid:16034284  0e+00
11 AT1G01120.1 918858|PACid:16049140  0e+00
12 AT1G01140.3 470161|PACid:16036015  0e+00
13 AT1G01150.1 918855|PACid:16037307 3e-150
14 AT1G01160.1 918854|PACid:16044153  1e-93
15 AT1G01170.2 311317|PACid:16052302  3e-54
16 AT1G01180.1 909860|PACid:16056125  0e+00
17 AT1G01190.1 311315|PACid:16059488  0e+00
18 AT1G01200.1 470156|PACid:16041002 3e-172
19 AT1G01210.1 311313|PACid:16057125  7e-76
20 AT1G01220.1 470155|PACid:16047984  0e+00

```

In this case 2 cores are being used to perform parallel processing, the `clean_folders` argument
specifies that all files returned by the corresponding orthology inference method
are removed after analyses.

## Program specific use of the orthologs() function

In this section small examples will illustrate the use
of the `orthologs()` function for each orthology inference program.


### BLASTp best hit

The BLASTp best hit method is a uni-directional BLAST best hit search of a `query organism A` 
against a `subject organism B` based on the `e-value`.

Orthology Inference using BLASTp best hit can be performed by specifying the argument `ortho_detection = "BH"`
and one computing core `comp_core = 1`:

```{r,eval=FALSE}

library(orthologr)

# perform orthology inference using BLAST best hit
# and fasta sequence files storing protein sequences
orthologs(query_file      = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
          subject_files   = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
          seq_type        = "protein", 
          ortho_detection = "BH", 
          clean_folders   = TRUE,
          detailed_output = FALSE,
          comp_cores      = 1)


```

```
       query_id            subject_id evalue
 1: AT1G01010.1 333554|PACid:16033839  0e+00
 2: AT1G01020.1 470181|PACid:16064328 7e-166
 3: AT1G01030.1 470180|PACid:16054974  0e+00
 4: AT1G01040.1 333551|PACid:16057793  0e+00
 5: AT1G01050.1 909874|PACid:16064489 2e-160
 6: AT1G01060.3 470177|PACid:16043374  0e+00
 7: AT1G01070.1 918864|PACid:16052578  0e+00
 8: AT1G01080.1 909871|PACid:16053217 1e-178
 9: AT1G01090.1 470171|PACid:16052860  0e+00
10: AT1G01110.2 333544|PACid:16034284  0e+00
11: AT1G01120.1 918858|PACid:16049140  0e+00
12: AT1G01140.3 470161|PACid:16036015  0e+00
13: AT1G01150.1 918855|PACid:16037307 3e-150
14: AT1G01160.1 918854|PACid:16044153  1e-93
15: AT1G01170.2 311317|PACid:16052302  3e-54
16: AT1G01180.1 909860|PACid:16056125  0e+00
17: AT1G01190.1 311315|PACid:16059488  0e+00
18: AT1G01200.1 470156|PACid:16041002 3e-172
19: AT1G01210.1 311313|PACid:16057125  7e-76
20: AT1G01220.1 470155|PACid:16047984  0e+00
```

The resulting table stores the orthologous gene pairs and the corresponding e-value of the best hit.
By specifying `detailed_output = TRUE` more information of the BLAST result can be obtained.


### BLASTp best reciprocal hit

The BLAST best reciprocal hit is a bi-directional BLAST best hit search of a `query organism A` 
against a `subject organism B` based on the `e-value`.

The Algorithm for BLAST best reciprocal hit runs as follows:

1) `bh_A <- best_hit(A,B)`

2) `bh_B <- best_hit(B,A)`

3) `join(bh_A,bh_B)` by tupel `(query_id, subject_id)`

In other words, only in case the tuple `(query_id, subject_id)` is returned as best hit
based on the `e-value` in both BLAST directions, the corresponding tupel `(query_id, subject_id)`
is retained as orthologous gene pair.

As demonstrated before a simple call of `orthologs()` using `ortho_detection = "RBH"`
and one computing core `comp_core = 1` can be performed as follows:

```{r,eval=FALSE}

library(orthologr)

# perform orthology inference using BLAST reciprocal best hit
# and fasta sequence files storing protein sequences
orthologs(query_file      = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
          subject_files   = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
          seq_type        = "protein", 
          ortho_detection = "RBH", 
          clean_folders   = FALSE,
          comp_cores      = 1)


```


```

      query_id            subject_id evalue
1  AT1G01010.1 333554|PACid:16033839  0e+00
2  AT1G01020.1 470181|PACid:16064328 7e-166
3  AT1G01030.1 470180|PACid:16054974  0e+00
4  AT1G01040.1 333551|PACid:16057793  0e+00
5  AT1G01050.1 909874|PACid:16064489 2e-160
6  AT1G01060.3 470177|PACid:16043374  0e+00
7  AT1G01070.1 918864|PACid:16052578  0e+00
8  AT1G01080.1 909871|PACid:16053217 1e-178
9  AT1G01090.1 470171|PACid:16052860  0e+00
10 AT1G01110.2 333544|PACid:16034284  0e+00
11 AT1G01120.1 918858|PACid:16049140  0e+00
12 AT1G01140.3 470161|PACid:16036015  0e+00
13 AT1G01150.1 918855|PACid:16037307 3e-150
14 AT1G01160.1 918854|PACid:16044153  1e-93
15 AT1G01170.2 311317|PACid:16052302  3e-54
16 AT1G01180.1 909860|PACid:16056125  0e+00
17 AT1G01190.1 311315|PACid:16059488  0e+00
18 AT1G01200.1 470156|PACid:16041002 3e-172
19 AT1G01210.1 311313|PACid:16057125  7e-76
20 AT1G01220.1 470155|PACid:16047984  0e+00

```

When you need more parameters returned by `RBH`, you can specify the `detailed_output = TRUE`
argument to obtain all BLAST parameters.

```{r,eval=FALSE}

library(orthologr)
library(dplyr)

# perform orthology inference using BLAST reciprocal best hit
# and fasta sequence files storing protein sequences
RBH <- orthologs(query_file = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
          subject_files     = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
          seq_type          = "protein", 
          ortho_detection   = "RBH",
          detailed_output   = TRUE, 
          clean_folders     = FALSE,
          comp_cores        = 1)


glimpse(RBH)

```

```
Variables:
$ query_id      (chr) "AT1G01010.1", "AT1G01020.1", "AT1G01030.1", "AT1G01040.1"...
$ subject_id    (chr) "333554|PACid:16033839", "470181|PACid:16064328", "470180|...
$ perc_identity (dbl) 73.99, 91.06, 95.54, 91.98, 100.00, 89.51, 95.08, 90.33, 9...
$ alig_length   (dbl) 469, 246, 359, 1970, 213, 648, 366, 300, 434, 528, 529, 45...
$ mismatches    (dbl) 80, 22, 12, 85, 0, 58, 14, 22, 8, 34, 4, 6, 68, 19, 0, 20,...
$ gap_openings  (dbl) 8, 0, 2, 10, 0, 5, 2, 2, 3, 0, 0, 1, 3, 2, 1, 1, 1, 0, 0, 0
$ q_start       (dbl) 1, 1, 1, 6, 1, 1, 1, 1, 1, 1, 1, 1, 5, 4, 2, 1, 1, 1, 1, 1
$ q_end         (dbl) 430, 246, 359, 1910, 213, 646, 366, 294, 429, 528, 529, 45...
$ s_start       (dbl) 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 1, 1, 16, 2, 4, 1, 6, 1, 1, 1
$ s_end         (dbl) 466, 246, 355, 1963, 213, 640, 362, 299, 433, 528, 529, 45...
$ evalue        (dbl) 0e+00, 7e-166, 0e+00, 0e+00, 2e-160, 0e+00, 0e+00, 1e-178,...
$ bit_score     (dbl) 627, 454, 698, 3704, 437, 1037, 696, 491, 859, 972, 1092, ...
```

In case you would like to store the corresponding `hit tables` returned by BLAST for
subsequent analyses, you can specify the `clean_folders = FALSE` argument. The corresponding
BLAST hit table can then be found in `file.path(tempdir(),"_blast_db")`.

A detailed overview of further analyses that can be done with the corresponding BLAST output
can be found in the [BLAST vignette](https://github.com/HajkD/orthologr/blob/master/vignettes/blast.Rmd).


### DELTA-BLAST best reciprocal hit

[Domain enhanced lookup time accelerated BLAST (DELTA-BLAST)](http://www.biomedcentral.com/content/pdf/1745-6150-7-12.pdf) is a new BLAST algorithm provided by NCBI. It was introduced as a useful program for the detection of remote protein homologs.
 
You can download the Conserved Domain Database (CDD) file `cdd.tar.gz` from [NCBI](ftp://ftp.ncbi.nlm.nih.gov/pub/mmdb/cdd) and store all `cdd_deltablast.* files` in a folder: `path/to/cdd_database/folder`.

More information on how to install and use DELTA-BLAST can be found [here](http://www.ncbi.nlm.nih.gov/books/NBK1763/#CmdLineAppsManual.Performing_a_DELTABLAS). Make sure that when using DELTA-BLAST in `orthologs()` the corresponding path to the `cdd_deltablast.* files` need to be specified in the `cdd.delta` argument.

The DELTA-BLAST best reciprocal hit is a bi-directional DELTA-BLAST best hit search of a `query organism A` 
against a `subject organism B` based on the `e-value`.

The Algorithm for DELTA-BLAST best reciprocal hit runs as follows:

1) `bh_A <- delta.blast(A,B)`

2) `bh_B <- delta.blast(B,A)`

3) `join(bh_A,bh_B)` by tupel `(query_id, subject_id)`

In other words, only in case the tuple `(query_id, subject_id)` is returned as best hit
based on the `e-value` in both DELTA-BLAST directions, the corresponding tupel `(query_id, subject_id)`
is retained as orthologous gene pair.

As demonstrated before a simple call of `orthologs()` using `ortho_detection = "DELTA"`, the `cdd.path = "path/to/cdd_database/folder"`, and one computing core `comp_core = 1` can be performed as follows:

```{r,eval=FALSE}

library(orthologr)

# perform orthology inference using DELTA-BLAST reciprocal best hit
# and fasta sequence files storing protein sequences
orthologs(query_file      = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
          subject_files   = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
          seq_type        = "protein", 
          ortho_detection = "DELTA",
          cdd.path        = "path/to/cdd_database/folder",
          clean_folders   = FALSE,
          detailed_output = FALSE,
          comp_cores      = 1)


```


```
      query_id            subject_id evalue
1  AT1G01010.1 333554|PACid:16033839  0e+00
2  AT1G01020.1 470181|PACid:16064328  3e-88
3  AT1G01030.1 470180|PACid:16054974  0e+00
4  AT1G01040.1 333551|PACid:16057793  0e+00
5  AT1G01050.1 909874|PACid:16064489 2e-128
6  AT1G01060.3 470177|PACid:16043374  0e+00
7  AT1G01070.1 918864|PACid:16052578  2e-68
8  AT1G01080.1 909871|PACid:16053217  4e-77
9  AT1G01090.1 470171|PACid:16052860  0e+00
10 AT1G01110.2 333544|PACid:16034284  0e+00
11 AT1G01120.1 918858|PACid:16049140  0e+00
12 AT1G01140.3 470161|PACid:16036015 9e-179
13 AT1G01150.1 918855|PACid:16037307 3e-120
14 AT1G01160.1 918854|PACid:16044153  4e-45
15 AT1G01170.2 311317|PACid:16052302  5e-54
16 AT1G01180.1 909860|PACid:16056125 8e-156
17 AT1G01190.1 311315|PACid:16059488  0e+00
18 AT1G01200.1 470156|PACid:16041002 2e-114
19 AT1G01210.1 311313|PACid:16057125  2e-40
20 AT1G01220.1 470155|PACid:16047984  0e+00
```

When you need more parameters returned by `DELTA-BLAST`, you can specify the `detailed_output = TRUE`
argument to obtain all DELTA-BLAST parameters.

```{r,eval=FALSE}

library(orthologr)
library(dplyr)

# perform orthology inference using DELTA-BLAST reciprocal best hit
# and fasta sequence files storing protein sequences
DELTA <- orthologs(query_file = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
          subject_files     = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
          seq_type          = "protein", 
          ortho_detection   = "DELTA",
          cdd.path          = "path/to/cdd_database/folder",
          detailed_output   = TRUE, 
          clean_folders     = FALSE,
          comp_cores        = 1)


glimpse(DELTA)

```

```
Variables:
$ query_id                (chr) "AT1G01010.1", "AT1G01020.1", "AT1G01030.1", "AT1G01040.1", "...
$ subject_id              (chr) "333554|PACid:16033839", "470181|PACid:16064328", "470180|PAC...
$ perc_identity           (dbl) 73.77, 91.06, 95.26, 91.57, 100.00, 89.51, 93.44, 78.60, 96.7...
$ num_ident_matches       (dbl) 346, 224, 342, 1803, 213, 580, 342, 235, 420, 494, 525, 446, ...
$ alig_length             (dbl) 469, 246, 359, 1969, 213, 648, 366, 299, 434, 528, 529, 453, ...
$ mismatches              (dbl) 81, 22, 13, 95, 0, 58, 20, 59, 8, 34, 4, 6, 71, 32, 0, 20, 31...
$ gap_openings            (dbl) 9, 0, 2, 8, 0, 5, 2, 1, 3, 0, 0, 1, 3, 1, 1, 1, 1, 0, 0, 0
$ q_start                 (dbl) 1, 1, 1, 6, 1, 1, 1, 1, 1, 1, 1, 1, 5, 4, 2, 1, 1, 1, 1, 1
$ q_end                   (dbl) 430, 246, 359, 1910, 213, 646, 366, 294, 429, 528, 529, 452, ...
$ s_start                 (dbl) 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 1, 1, 16, 2, 4, 1, 6, 1, 1, 1
$ s_end                   (dbl) 466, 246, 355, 1963, 213, 640, 362, 299, 433, 528, 529, 453, ...
$ evalue                  (dbl) 0e+00, 3e-88, 0e+00, 0e+00, 2e-128, 0e+00, 2e-68, 4e-77, 0e+0...
$ bit_score               (dbl) 637, 257, 502, 2362, 357, 1003, 214, 232, 658, 763, 747, 504,...
$ score_raw               (dbl) 1643, 657, 1294, 6123, 917, 2592, 544, 592, 1698, 1971, 1928,...
$ query_coverage_per_subj (dbl) 100, 100, 100, 99, 100, 100, 100, 100, 100, 100, 100, 100, 83...
```

In case you would like to store the corresponding `hit tables` returned by DELTA-BLAST for
subsequent analyses, you can specify the `clean_folders = FALSE` argument.
When `clean_folders` is set to `FALSE`, a folder named `_blast_db` will be created 
inside the `tempdir()` folder of your R session storing the corresponding DELTA-BLAST output.
The corresponding DELTA-BLAST hit table can then be found in `file.path(tempdir(),"_blast_db")`.
A detailed overview of further analyses that can be done with the corresponding DELTA-BLAST output
can be found in the [BLAST vignette](https://github.com/HajkD/orthologr/blob/master/vignettes/blast.Rmd).


### ProteinOrtho

ProteinOrtho is a orthology inference program to detect orthologous genes within different species. For this purpose it compares similarities of given gene sequences and clusters them to find significant groups. To enhance the prediction accuracy, the relative order of genes (synteny) can be used as additional feature for the discrimination of orthologs ([Lechner et al., 2011](https://www.bioinf.uni-leipzig.de/Software/proteinortho/manual.html)).

#### Installing ProteinOrtho for use in orthologr

Please follow the [installation instructions](https://www.bioinf.uni-leipzig.de/Software/proteinortho/manual.html) from the ProteinOrtho web page.

When you have ProteinOrtho installed on your machine, make sure that you can
execute the file `proteinortho5.pl` from your default execution `PATH`, e.g. `/usr/local/bin/` (MacOS, Linux).

You can test whether ProteinOrtho can be executed properly by typing the following
command in your R session:

```{r,eval=FALSE}

system("proteinortho5.pl -h")

```

This way you should see the following output:

```
Usage: proteinortho5.pl [OPTIONS] FASTA1 FASTA2 [FASTA...]
Options: -e=          E-value for blast [default: 1e-05]
         -p=          blast program {blastn|blastp|blastn+|blastp+}
                      [default: blastp+]
         -project=    prefix for all result file names [default: myproject]
         -synteny     activate PoFF extension to separate similar sequences
                      by contextual adjacencies (requires .gff for each .fasta)
         -dups=       PoFF: number of reiterations for adjacencies heuristic,
                      to determine duplicated regions (default: 0)
         -cs=         PoFF: Size of a maximum common substring (MCS) for
                      adjacency matches (default: 3)
         -alpha=      PoFF: weight of adjacencies vs. sequence similarity
                      (default: 0.5)
         -keep        stores temporary blast results for reuse
         -force       forces recalculation of blast results in any case
         -cpus=       number of processors to use [default: auto]
         -selfblast   apply selfblast, detects paralogs without orthologs
         -singles     report singleton genes without any hit
         -identity=   min. percent identity of best blast hits [default: 25]
         -cov=        min. coverage of best blast alignments in % [default: 50]
         -conn=       min. algebraic connectivity [default: 0.1]
         -sim=        min. similarity for additional hits (0..1) [default: 0.95]
         -step=       1 -> generate indices
                      2 -> run blast (and ff-adj, if -synteny is set)
                      3 -> clustering
                      0 -> all (default)
         -blastpath=  path to your local blast (if not installed globally)
         -verbose     keeps you informed about the progress
         -clean       remove all unnecessary files after processing
         -graph       generate .graph files (pairwise orthology relations)
         -debug       gives detailed information for bug tracking

         More specific blast parameters can be defined by
         -blastParameters='[parameters]' (e.g. -blastParameters='-seg no')

         In case jobs should be distributed onto several machines, use
         -startat=    File number to start with (default: 0)
         -stopat=     File number to end with (default: -1)
 
Invalid command line option: '-h'!

```

#### In case you have problems with ProteinOrtho 

If you cannot reproduce the previous result, then `proteinortho5.pl` cannot be executed properly 
from your default execution `PATH` (e.g. `/usr/local/bin/`). 

In this case first you can try to store all files within the `proteinortho_v5.07`
folder in `/usr/local/bin/` (MacOS and Linux) or your default `Program Files` folder path (Windows).

Furthermore, you can also try the following steps:

* specify the `path` argument inside the `orthologs()` function to pass 
the path to your `proteinortho_v5.07` folder
* read the documentation of the `ProteinOrtho()` function: `?ProteinOrtho`
* formulate a bug report

__Note:__ when using the synteny option of ProteinOrtho, we noticed that the
necessary file `proteinortho5_clustering.cpp` stored in the `proteinortho_v5.07` folder
needs to be stored in `/usr/local/bin/` and in our case also needed to be recompiled with
`g++ proteinortho5_clustering.cpp -o proteinortho5_clustering` inside the `Terminal`.


#### Running orthologs() with ProteinOrtho

```{r,eval=FALSE}

library(orthologr)

# perform orthology inference using ProteinOrtho
# and fasta sequence files storing protein sequences
orthologs(query_file      = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
          subject_files   = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
          seq_type        = "protein", 
          ortho_detection = "PO", 
          clean_folders   = FALSE)


```

```

      ortho_lyra_aa.fasta ortho_thal_aa.fasta evalue_ab bitscore_ab evalue_ba bitscore_ba
 1: 311313|PACid:16057125         AT1G01210.1     7e-76         215     7e-76         215
 2: 311315|PACid:16059488         AT1G01190.1     0e+00        1036     0e+00        1036
 3: 311317|PACid:16052302         AT1G01170.2     4e-54         158     3e-54         158
 4: 333544|PACid:16034284         AT1G01110.2     0e+00         978     0e+00         972
 5: 333551|PACid:16057793         AT1G01040.1     0e+00        3704     0e+00        3704
 6: 333554|PACid:16033839         AT1G01010.1     0e+00         615     0e+00         627
 7: 470155|PACid:16047984         AT1G01220.1     0e+00        2106     0e+00        2106
 8: 470156|PACid:16041002         AT1G01200.1    3e-172         470    3e-172         470
 9: 470161|PACid:16036015         AT1G01140.3     0e+00         918     0e+00         918
10: 470171|PACid:16052860         AT1G01090.1     0e+00         859     0e+00         859
11: 470177|PACid:16043374         AT1G01060.3     0e+00        1033     0e+00        1037
12: 470180|PACid:16054974         AT1G01030.1     0e+00         698     0e+00         698
13: 470181|PACid:16064328         AT1G01020.1    6e-158         434    7e-166         454
14: 909860|PACid:16056125         AT1G01180.1     0e+00         599     0e+00         576
15: 909871|PACid:16053217         AT1G01080.1    5e-169         466    1e-178         491
16: 909874|PACid:16064489         AT1G01050.1    2e-160         437    2e-160         437
17: 918854|PACid:16044153         AT1G01160.1     2e-86         249     1e-93         268
18: 918855|PACid:16037307         AT1G01150.1    2e-150         421    3e-150         421
19: 918858|PACid:16049140         AT1G01120.1     0e+00        1092     0e+00        1092
20: 918864|PACid:16052578         AT1G01070.1     0e+00         696     0e+00         696

```

Again the `comp_cores` argument can be used to run ProteinOrtho in parallel.

```{r,eval=FALSE}

library(orthologr)

# perform orthology inference using ProteinOrtho
# and fasta sequence files storing protein sequences
orthologs(query_file      = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
          subject_files   = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
          seq_type        = "protein", 
          comp_cores      = 2,
          ortho_detection = "PO", 
          clean_folders   = FALSE)


```


You can run ProteinOrtho with multiple subject organisms.

```{r,eval=FALSE}

library(orthologr)

# defining 3 subject organisms: A. lyrata, B. rapa, and T. halophila
subject_organisms <- c(system.file('seqs/example_brapa_aa.faa', package = 'orthologr'),
                       system.file('seqs/example_alyra_aa.faa', package = 'orthologr'),
                       system.file('seqs/example_thalo_aa.faa', package = 'orthologr'))

ProteinOrtho(query_file = system.file('seqs/example_athal_aa.faa', package = 'orthologr'),
          subject_files = subject_organisms,
          seq_type      = "protein")


```

```

$comparison_1
       # example_athal_aa.faa  example_alyra_aa.faa evalue_ab bitscore_ab evalue_ba bitscore_ba
1  AT1G01010.1|PACid:19656964 333554|PACid:16033839       0.0         627       0.0         615
2  AT1G01020.1|PACid:19655142 470181|PACid:16064328    7e-166         454    6e-158         434
3  AT1G01030.1|PACid:19649747 470180|PACid:16054974       0.0         698       0.0         698
4  AT1G01040.1|PACid:19652432 333551|PACid:16057793       0.0        3704       0.0        3704
5  AT1G01050.1|PACid:19652974 909874|PACid:16064489    2e-160         437    2e-160         437
6  AT1G01070.1|PACid:19650807 918864|PACid:16052578       0.0         696       0.0         696
7  AT1G01080.1|PACid:19651136 909871|PACid:16053217    1e-178         491    5e-169         466
8  AT1G01090.1|PACid:19650207 470171|PACid:16052860       0.0         859       0.0         859
9  AT1G01110.2|PACid:19658169 333544|PACid:16034284       0.0         972       0.0         978
10 AT1G01120.1|PACid:19655257 918858|PACid:16049140       0.0        1092       0.0        1092
11 AT1G01160.1|PACid:19654307 918854|PACid:16044153     1e-93         268     2e-86         249
12 AT1G01180.1|PACid:19654493 909860|PACid:16056125       0.0         572       0.0         594
13 AT1G01190.1|PACid:19654322 311315|PACid:16059488       0.0        1036       0.0        1036
14 AT1G01200.1|PACid:19649914 470156|PACid:16041002    3e-172         470    3e-172         470
15 AT1G01220.1|PACid:19652890 470155|PACid:16047984       0.0        2106       0.0        2106
16 AT1G01225.1|PACid:19656715 470154|PACid:16066159    6e-168         461    1e-163         450
17 AT1G01230.1|PACid:19650274 470153|PACid:16038331    3e-117         324    3e-117         324
18 AT1G01240.3|PACid:19650438 470152|PACid:16047967       0.0         568       0.0         545
19 AT1G01250.1|PACid:19649318 470149|PACid:16044741    1e-124         345    1e-124         345
20 AT1G01260.3|PACid:19651080 470147|PACid:16058975       0.0        1108       0.0        1117

$comparison_2
     # example_brapa_aa.faa  example_alyra_aa.faa evalue_ab bitscore_ab evalue_ba bitscore_ba
1  Bra032623|PACid:22715924 918854|PACid:16044153     1e-93         268     2e-98         281
2  Bra032626|PACid:22713115 470155|PACid:16047984       0.0        1946       0.0        1946
3  Bra032627|PACid:22713604 470154|PACid:16066159    1e-159         439    1e-160         442
4  Bra032629|PACid:22714275 470152|PACid:16047967    3e-152         427    8e-136         385
5  Bra032630|PACid:22715365 470149|PACid:16044741    2e-106         298    4e-106         298
6  Bra033273|PACid:22692002 470147|PACid:16058975       0.0         846       0.0         858
7  Bra033276|PACid:22693754 470153|PACid:16038331    3e-111         310    3e-111         310
8  Bra033277|PACid:22693004 470156|PACid:16041002    5e-160         439    5e-160         439
9  Bra033278|PACid:22692201 311315|PACid:16059488       0.0         992       0.0         992
10 Bra033279|PACid:22694360 909860|PACid:16056125       0.0         508       0.0         516
11 Bra033283|PACid:22693298 918858|PACid:16049140       0.0        1015       0.0        1015
12 Bra033284|PACid:22692650 333544|PACid:16034284       0.0         842       0.0         807
13 Bra033286|PACid:22694322 470171|PACid:16052860       0.0         818       0.0         818
14 Bra033287|PACid:22693868 909871|PACid:16053217    3e-128         358    2e-128         360
15 Bra033290|PACid:22692027 918864|PACid:16052578       0.0         621       0.0         621
16 Bra033292|PACid:22694379 909874|PACid:16064489    4e-158         432    4e-158         432
17 Bra033293|PACid:22692618 333551|PACid:16057793       0.0        3257       0.0        3334
18 Bra033294|PACid:22691990 470180|PACid:16054974       0.0         528       0.0         574
19 Bra033295|PACid:22694612 470181|PACid:16064328    4e-147         407    1e-139         387
20 Bra033296|PACid:22693237 333554|PACid:16033839    7e-137         398    1e-131         385

$comparison_3
           # example_thalo_aa.faa  example_alyra_aa.faa evalue_ab bitscore_ab evalue_ba bitscore_ba
1  Thhalv10006531m|PACid:20187082 333551|PACid:16057793       0.0        3484       0.0        3514
2  Thhalv10006637m|PACid:20187150 470155|PACid:16047984       0.0        2022       0.0        2022
3  Thhalv10007139m|PACid:20185859 470147|PACid:16058975       0.0        1032       0.0         992
4  Thhalv10007299m|PACid:20188214 333544|PACid:16034284       0.0         872       0.0         844
5  Thhalv10007320m|PACid:20187751 311315|PACid:16059488       0.0         959       0.0         959
6  Thhalv10007351m|PACid:20187717 918858|PACid:16049140       0.0        1055       0.0        1055
7  Thhalv10007665m|PACid:20186865 333554|PACid:16033839       0.0         538       0.0         532
8  Thhalv10007695m|PACid:20187891 470171|PACid:16052860       0.0         844       0.0         844
9  Thhalv10008068m|PACid:20187490 470180|PACid:16054974       0.0         566       0.0         596
10 Thhalv10008123m|PACid:20187436 470152|PACid:16047967       0.0         530       0.0         523
11 Thhalv10008287m|PACid:20186971 909860|PACid:16056125       0.0         546       0.0         557
12 Thhalv10008355m|PACid:20185661 909871|PACid:16053217    2e-163         452    5e-170         469
13 Thhalv10008482m|PACid:20185881 918854|PACid:16044153    4e-100         288    2e-100         288
14 Thhalv10008506m|PACid:20187047 470154|PACid:16066159    1e-160         442    4e-162         446
15 Thhalv10008618m|PACid:20186467 470181|PACid:16064328    1e-153         423    6e-146         404
16 Thhalv10008767m|PACid:20187619 909874|PACid:16064489    1e-158         433    1e-158         433
17 Thhalv10008983m|PACid:20187021 470153|PACid:16038331    6e-116         320    6e-116         320
18 Thhalv10009311m|PACid:20187590 470156|PACid:16041002    2e-163         447    2e-163         447
19 Thhalv10009402m|PACid:20186843 470149|PACid:16044741    2e-119         332    2e-119         332
20 Thhalv10009501m|PACid:20185805 918864|PACid:16052578       0.0         639       0.0         630

$comparison_4
     # example_brapa_aa.faa       example_athal_aa.faa evalue_ab bitscore_ab evalue_ba bitscore_ba
1  Bra032623|PACid:22715924 AT1G01160.1|PACid:19654307     3e-88         254     7e-97         275
2  Bra032626|PACid:22713115 AT1G01220.1|PACid:19652890       0.0        1953       0.0        1953
3  Bra032627|PACid:22713604 AT1G01225.1|PACid:19656715    1e-157         435    3e-162         446
4  Bra032629|PACid:22714275 AT1G01240.3|PACid:19650438    3e-155         435    4e-142         401
5  Bra032630|PACid:22715365 AT1G01250.1|PACid:19649318    5e-103         290    6e-103         290
6  Bra033273|PACid:22692002 AT1G01260.3|PACid:19651080       0.0         870       0.0         856
7  Bra033276|PACid:22693754 AT1G01230.1|PACid:19650274    1e-110         308    8e-111         308
8  Bra033277|PACid:22693004 AT1G01200.1|PACid:19649914    8e-158         433    8e-158         433
9  Bra033278|PACid:22692201 AT1G01190.1|PACid:19654322       0.0         986       0.0         986
10 Bra033279|PACid:22694360 AT1G01180.1|PACid:19654493       0.0         509       0.0         509
11 Bra033283|PACid:22693298 AT1G01120.1|PACid:19655257       0.0        1014       0.0        1014
12 Bra033284|PACid:22692650 AT1G01110.2|PACid:19658169       0.0         843       0.0         803
13 Bra033286|PACid:22694322 AT1G01090.1|PACid:19650207       0.0         811       0.0         811
14 Bra033287|PACid:22693868 AT1G01080.1|PACid:19651136    1e-125         352    4e-126         353
15 Bra033290|PACid:22692027 AT1G01070.1|PACid:19650807       0.0         610       0.0         610
16 Bra033292|PACid:22694379 AT1G01050.1|PACid:19652974    4e-158         432    4e-158         432
17 Bra033293|PACid:22692618 AT1G01040.1|PACid:19652432       0.0        3408       0.0        3430
18 Bra033294|PACid:22691990 AT1G01030.1|PACid:19649747       0.0         533       0.0         573
19 Bra033295|PACid:22694612 AT1G01020.1|PACid:19655142    2e-138         385    9e-135         375
20 Bra033296|PACid:22693237 AT1G01010.1|PACid:19656964    1e-139         404    6e-135         392

$comparison_5
           # example_thalo_aa.faa       example_athal_aa.faa evalue_ab bitscore_ab evalue_ba bitscore_ba
1  Thhalv10006531m|PACid:20187082 AT1G01040.1|PACid:19652432       0.0        3628       0.0        3603
2  Thhalv10006637m|PACid:20187150 AT1G01220.1|PACid:19652890       0.0        2026       0.0        2026
3  Thhalv10007139m|PACid:20185859 AT1G01260.3|PACid:19651080       0.0        1036       0.0         987
4  Thhalv10007299m|PACid:20188214 AT1G01110.2|PACid:19658169       0.0         875       0.0         840
5  Thhalv10007320m|PACid:20187751 AT1G01190.1|PACid:19654322       0.0         951       0.0         951
6  Thhalv10007351m|PACid:20187717 AT1G01120.1|PACid:19655257       0.0        1055       0.0        1055
7  Thhalv10007665m|PACid:20186865 AT1G01010.1|PACid:19656964       0.0         515       0.0         521
8  Thhalv10007695m|PACid:20187891 AT1G01090.1|PACid:19650207       0.0         842       0.0         842
9  Thhalv10008068m|PACid:20187490 AT1G01030.1|PACid:19649747       0.0         584       0.0         608
10 Thhalv10008123m|PACid:20187436 AT1G01240.3|PACid:19650438       0.0         515       0.0         530
11 Thhalv10008287m|PACid:20186971 AT1G01180.1|PACid:19654493       0.0         549       0.0         549
12 Thhalv10008355m|PACid:20185661 AT1G01080.1|PACid:19651136    9e-157         435    1e-172         476
13 Thhalv10008482m|PACid:20185881 AT1G01160.1|PACid:19654307     5e-90         261     2e-99         284
14 Thhalv10008506m|PACid:20187047 AT1G01225.1|PACid:19656715    1e-159         440    1e-164         452
15 Thhalv10008618m|PACid:20186467 AT1G01020.1|PACid:19655142    6e-141         391    1e-141         393
16 Thhalv10008767m|PACid:20187619 AT1G01050.1|PACid:19652974    1e-158         433    1e-158         433
17 Thhalv10008983m|PACid:20187021 AT1G01230.1|PACid:19650274    2e-115         319    2e-115         319
18 Thhalv10009311m|PACid:20187590 AT1G01200.1|PACid:19649914    2e-161         442    2e-161         442
19 Thhalv10009402m|PACid:20186843 AT1G01250.1|PACid:19649318    4e-119         331    6e-119         331
20 Thhalv10009501m|PACid:20185805 AT1G01070.1|PACid:19650807       0.0         633       0.0         625

$comparison_6
           # example_thalo_aa.faa     example_brapa_aa.faa evalue_ab bitscore_ab evalue_ba bitscore_ba
1  Thhalv10006531m|PACid:20187082 Bra033293|PACid:22692618       0.0        3495       0.0        3449
2  Thhalv10006637m|PACid:20187150 Bra032626|PACid:22713115       0.0        1963       0.0        1963
3  Thhalv10007139m|PACid:20185859 Bra033273|PACid:22692002       0.0         872       0.0         850
4  Thhalv10007299m|PACid:20188214 Bra033284|PACid:22692650       0.0         848       0.0         849
5  Thhalv10007320m|PACid:20187751 Bra033278|PACid:22692201       0.0         968       0.0         968
6  Thhalv10007351m|PACid:20187717 Bra033283|PACid:22693298       0.0        1036       0.0        1036
7  Thhalv10007665m|PACid:20186865 Bra033296|PACid:22693237    2e-150         432    1e-147         425
8  Thhalv10007695m|PACid:20187891 Bra033286|PACid:22694322       0.0         830       0.0         830
9  Thhalv10008068m|PACid:20187490 Bra033294|PACid:22691990       0.0         587       0.0         570
10 Thhalv10008123m|PACid:20187436 Bra032629|PACid:22714275    6e-140         396    2e-153         431
11 Thhalv10008287m|PACid:20186971 Bra033279|PACid:22694360       0.0         508       0.0         508
12 Thhalv10008355m|PACid:20185661 Bra033287|PACid:22693868    3e-129         361    7e-129         360
13 Thhalv10008482m|PACid:20185881 Bra032623|PACid:22715924    3e-115         325    2e-114         323
14 Thhalv10008506m|PACid:20187047 Bra032627|PACid:22713604    6e-167         459    2e-169         465
15 Thhalv10008618m|PACid:20186467 Bra033295|PACid:22694612    2e-141         392    3e-141         392
16 Thhalv10008767m|PACid:20187619 Bra033292|PACid:22694379    2e-158         432    2e-158         432
17 Thhalv10008983m|PACid:20187021 Bra033276|PACid:22693754    9e-111         308    1e-110         308
18 Thhalv10009311m|PACid:20187590 Bra033277|PACid:22693004    2e-159         437    2e-159         437
19 Thhalv10009402m|PACid:20186843 Bra032630|PACid:22715365    1e-106         299    9e-107         300
20 Thhalv10009501m|PACid:20185805 Bra033290|PACid:22692027       0.0         667       0.0         665

$proteinortho_tbl
   X..Species Genes Alg..Conn.       example_athal_aa.faa  example_alyra_aa.faa     example_brapa_aa.faa
1           4     4          1 AT1G01010.1|PACid:19656964 333554|PACid:16033839 Bra033296|PACid:22693237
2           4     4          1 AT1G01020.1|PACid:19655142 470181|PACid:16064328 Bra033295|PACid:22694612
3           4     4          1 AT1G01030.1|PACid:19649747 470180|PACid:16054974 Bra033294|PACid:22691990
4           4     4          1 AT1G01040.1|PACid:19652432 333551|PACid:16057793 Bra033293|PACid:22692618
5           4     4          1 AT1G01050.1|PACid:19652974 909874|PACid:16064489 Bra033292|PACid:22694379
6           4     4          1 AT1G01070.1|PACid:19650807 918864|PACid:16052578 Bra033290|PACid:22692027
7           4     4          1 AT1G01080.1|PACid:19651136 909871|PACid:16053217 Bra033287|PACid:22693868
8           4     4          1 AT1G01090.1|PACid:19650207 470171|PACid:16052860 Bra033286|PACid:22694322
9           4     4          1 AT1G01110.2|PACid:19658169 333544|PACid:16034284 Bra033284|PACid:22692650
10          4     4          1 AT1G01120.1|PACid:19655257 918858|PACid:16049140 Bra033283|PACid:22693298
11          4     4          1 AT1G01160.1|PACid:19654307 918854|PACid:16044153 Bra032623|PACid:22715924
12          4     4          1 AT1G01180.1|PACid:19654493 909860|PACid:16056125 Bra033279|PACid:22694360
13          4     4          1 AT1G01190.1|PACid:19654322 311315|PACid:16059488 Bra033278|PACid:22692201
14          4     4          1 AT1G01200.1|PACid:19649914 470156|PACid:16041002 Bra033277|PACid:22693004
15          4     4          1 AT1G01220.1|PACid:19652890 470155|PACid:16047984 Bra032626|PACid:22713115
16          4     4          1 AT1G01225.1|PACid:19656715 470154|PACid:16066159 Bra032627|PACid:22713604
17          4     4          1 AT1G01230.1|PACid:19650274 470153|PACid:16038331 Bra033276|PACid:22693754
18          4     4          1 AT1G01240.3|PACid:19650438 470152|PACid:16047967 Bra032629|PACid:22714275
19          4     4          1 AT1G01250.1|PACid:19649318 470149|PACid:16044741 Bra032630|PACid:22715365
20          4     4          1 AT1G01260.3|PACid:19651080 470147|PACid:16058975 Bra033273|PACid:22692002
             example_thalo_aa.faa
1  Thhalv10007665m|PACid:20186865
2  Thhalv10008618m|PACid:20186467
3  Thhalv10008068m|PACid:20187490
4  Thhalv10006531m|PACid:20187082
5  Thhalv10008767m|PACid:20187619
6  Thhalv10009501m|PACid:20185805
7  Thhalv10008355m|PACid:20185661
8  Thhalv10007695m|PACid:20187891
9  Thhalv10007299m|PACid:20188214
10 Thhalv10007351m|PACid:20187717
11 Thhalv10008482m|PACid:20185881
12 Thhalv10008287m|PACid:20186971
13 Thhalv10007320m|PACid:20187751
14 Thhalv10009311m|PACid:20187590
15 Thhalv10006637m|PACid:20187150
16 Thhalv10008506m|PACid:20187047
17 Thhalv10008983m|PACid:20187021
18 Thhalv10008123m|PACid:20187436
19 Thhalv10009402m|PACid:20186843
20 Thhalv10007139m|PACid:20185859

```

The resulting output is a list storing all combinations of pairwise comparisons
as data frames inside each list element. The last list element defines the gene relationships
in terms of gene ids.


In case you would like to use the `synteny` option of ProteinOrtho, we refer
to the `ProteinOrtho()` function. When performing synteny, a *.gff file needs
to be stored in a folder named `_ProteinOrtho` that needs to be created inside the
current working directory of your R session (`dir.create("_ProteinOrtho")`).

```{r,eval=FALSE}

library(orthologr)

# it is also possible to run ProteinOrtho using the synteny option
# make sure you can provide a *.gff file
# see https://www.bioinf.uni-leipzig.de/Software/proteinortho/manual.html for details
ProteinOrtho(query_file      = system.file('seqs/ortho_thal_aa.fasta', package = 'orthologr'),
             subject_files   = system.file('seqs/ortho_lyra_aa.fasta', package = 'orthologr'),
             seq_type        = "protein",
             format          = "fasta",
             po_params       = "-synteny",
             clean_folders   = FALSE)

```
