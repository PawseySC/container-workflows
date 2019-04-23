---
title: "Docker: A Bioinformatics Example"
teaching: 10
exercises: 0
questions:
objectives:
- Run a real-world bioinformatics application in a docker container

keypoints:
---

### Running BLAST from a container ###
We'll be running a BLAST (Basic Local Alignment Search Tool) example with a container from [BioContainers](https://biocontainers.pro).  BLAST is a tool bioinformaticians use to compare a sample genetic sequence to a database of known seqeuences; it's one of the most widely used bioinformatics tools.

To begin, we'll pull the BLAST container (this will take a little bit):

```
> docker pull biocontainers/blast:v2.2.31_cv2
v2.2.31_cv2: Pulling from biocontainers/blast
22dc81ace0ea: Pull complete 
1a8b3c87dba3: Pull complete 
...
Digest: sha256:238717ec69830ec62a19fc05c6f70183f218a13f7678864060f0157dc63dc54f
Status: Downloaded newer image for biocontainers/blast:v2.2.31_cv2
```

We can run a simple command to verify the container works:

```
> docker run biocontainers/blast:v2.2.31_cv2 blastp -help
USAGE
  blastp [-h] [-help] [-import_search_strategy filename]
...
 -use_sw_tback
   Compute locally optimal Smith-Waterman alignments?
```

Let's download some data to start blasting:

```
> mkdir blast-example
> cd blast-example
> wget http://www.uniprot.org/uniprot/P04156.fasta
```

This is a human prion FASTA sequence.  We'll also need a reference database to blast against:

```
> curl -O ftp://ftp.ncbi.nih.gov/refseq/D_rerio/mRNA_Prot/zebrafish.1.protein.faa.gz
> gunzip zebrafish.1.protein.faa.gz
```

We need to prepare the zebrafish database with `makeblastdb` for the search, so we'll run the following (see a previous lesson for details on `-v`):

```
> docker run -v `pwd`:/data/ biocontainers/blast:v2.2.31_cv2 makeblastdb -in zebrafish.1.protein.faa -dbtype prot
```

After the docker container has terminated, you should see several new files in the current directory.  We can now do the final alignment step:

```
> docker run -v `pwd`:/data/ biocontainers/blast:v2.2.31_cv2 blastp -query P04156.fasta -db zebrafish.1.protein.faa -out results.txt
```
The final results are stored in `results.txt`;
```
> less results.txt
                                                                     Score     E
equences producing significant alignments:                          (Bits)  Value

 XP_017207509.1 protein piccolo isoform X2 [Danio rerio]             43.9    2e-04
 XP_017207511.1 mucin-16 isoform X4 [Danio rerio]                    43.9    2e-04
 XP_021323434.1 protein piccolo isoform X5 [Danio rerio]             43.5    3e-04
 XP_017207510.1 protein piccolo isoform X3 [Danio rerio]             43.5    3e-04
 XP_021323433.1 protein piccolo isoform X1 [Danio rerio]             43.5    3e-04
 XP_009291733.1 protein piccolo isoform X1 [Danio rerio]             43.5    3e-04
 NP_001268391.1 chromodomain-helicase-DNA-binding protein 2 [Dan...  35.8    0.072
...
```
We can see that several proteins in the zebrafish genome match those in the human prion (interesting?).


### Conclusion ###
There are a lot of applications (not just bioinformatics) already wrapped up in container images.  Here's a small list of some of the repositories we use at Pawsey:

* [Docker Hub](https://hub.docker.com)
* [Bioboxes](bioboxes.org)
* [Biocontainers](biocontainers.pro)
* [Nvidia GPU Cloud (NGC)](ngc.nvidia.com)*
* [quay.io](quay.io)*

The last two require you to create an account and login to access containers.

### Best practices ###

- don't re-invent the wheel, it's worth looking to see who's done what you want already
