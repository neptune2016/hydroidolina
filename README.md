# Hydroidolina Phylogeny Assignment

In this assignment you will reanalyze the data from the following paper:

> Cartwright, P., Evans, N. M., Dunn, C. W., Marques, A. C., Miglietta, M. P., 
Schuchert, P., & Collins, A. G. (2008). Phylogenetics of Hydroidolina 
(Hydrozoa: Cnidaria). Journal of the Marine Biological Association of the UK, 
88(08), 1663-1672. 
[doi:10.1017/S0025315408002257](http://dx.doi.org/10.1017/S0025315408002257)

Below are a series of analyses to perform with these data. As with many analyses, most of the hands-on work is data wrangling (getting files organized and in the right format). New ground that we cover here includes:

- Batch download of data from NCBI
- Reformatting sequence names
- Multiple sequence alignment
- Combining data from multiple genes into a single alignment
- Running multiple analyses

## Steps

Some of the steps below build on things you already did in the [siphonophore16s](https://github.com/Phylogenetics-Brown-BIOL1425/siphonophore16s) analysis. Check that repository if you are unsure about details here.


### Fork the repository on github and clone it

Browse to the GitHub repository for this exercise. If you are reading this at GitHub, you are already [there](https://github.com/neptune2016/hydroidolina).

Click the "Fork" button on the top right of the page. This creates your own independent copy of the repository to work with, so we can all make changes without affecting other peoples' analyses. It also takes you to the web page for the new repository. The url will be something like:

    https://github.com/github_username/hydroidolina

where `github_username` is your github username.

Clone the repository to the computer where you will run the analyses.

### The data

The data we will analyze are available in three fasta files:

    28s.raw.fasta
    18s.raw.fasta
    16s.raw.fasta

#### How the data were obtained

This section describes how the data were obtained. You don't need to go through it, and is included here as a reference.

Table 1 of the paper presents the [NCBI](http://www.ncbi.nlm.nih.gov) accession 
numbers for all the data included in the analyses. I have extracted the data 
from Table 1 and are presented here in three files (I also cleaned up a couple 
of typos in the accession numbers):

    28s.txt
    18s.txt
    16s.txt
    
Use the 
[NCBI Batch Entrez interface](http://www.ncbi.nlm.nih.gov/sites/batchentrez) 
to download a fasta file for the sequences listed in each of these three files. 
From the top of the interface, select the 'Nucleotide' Database and chose a file 
of accession numbers. Click retrieve, and then follow the link that is generated. 
This will take you to a page with the sequence summaries. Click 'Send to', 
choose 'File' as the destination, 'FASTA' as the format, and then click 
'Create File'.

Repeat this for each of the three gene files, rename them `28s.raw.fasta` etc. 

### Reformat the headers

The fasta header rows (which start with `>`) have lots of information. To combine data across genes we will need the header to just have the species name. Use the python script in this repository to reformat the headers with regular expressions:

    python shorten_headers.py 16s.raw.fasta > 16s.fasta
    python shorten_headers.py 18s.raw.fasta > 18s.fasta
    python shorten_headers.py 28s.raw.fasta > 28s.fasta

Then add the fasta files to the repository, commit, and push them:

   git add *.fasta
   git commit -am "added raw fasta files"
   git push


### Generate alignments

In the raw fasta files, homologous sites are not in the same columns because the sequences start in different places, and there have been insertions and deletions in the course of sequence evolution. Aligners attempt to place homologous sites in the same columns by inserting gaps (usually denoted with the `-` character) to create a data matrix. We'll use the aligner `mafft`. 
    
    mafft 28s.fasta > 28s.aligned.fasta
    mafft 18s.fasta > 18s.aligned.fasta
    mafft 16s.fasta > 16s.aligned.fasta

    git add *aligned.fasta
    git commit -am "added alignments"
    git push


### Create concatenated alignments

Now you have alignments for three genes. We will combine the data by creating concatenated alignments that include data for all genes. There are a few ways to do this, we'll use the tool [phyutility](https://github.com/blackrim/phyutility). If you don't have java installed already, you will need to install it (if you try the command below but don't have java, it may describe how to install java on your system). Rather than install java on your laptop, you could just run the command on oscar (which has java installed already)

Execute the following line to create the combined analysis:

    java -jar phyutility.jar -concat -in 16s.aligned.fasta 18s.aligned.fasta 28s.aligned.fasta -out combined.aln

This will create a new fasta file that has data from all three genes. Data are combined in the same row for sequences with the same species names.

You need to convert the file as a phylip file. That can be done as follows:

    python fasta2phylip.py combined.aln > combined.phy

Now that you have all your alignment files, go ahead and commit them:

    git add combined.aln combined.phy
    git commit -am "generated nexus and phylip alignment files"
    git push

### Maximum likelihood analyses

Finally, perform the analysis with the phylogenetic following command:

    raxmlHPC-MPI  -f a -x 12345 -p 12345 -N 20 -m GTRGAMMA -s combined.phy -n combined_boot

We are running only 20 bootstraps. Typically you would do a lot more (100-1000), but this will give an idea of how the analysis is done in a shorter time.

