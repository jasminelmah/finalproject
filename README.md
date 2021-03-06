---
title: "compgen final project"
csl: plos.csl
bibliography: Bib.bib
---


# Final Project

## Repo Map  
A quick description of the folders in my repo:  
* **agalma**: scripts and output of Agalma runs  
* **class_presentation**: slides and materials for my CompGen class presentation  
* **code**: all the code used during this project. Note that neurogenes.rmd are my rough notes.  
* **code_clean.rmd**: contains a runnable version of the code in the Methods section of this readme (see last section). 
* **data**: contains genome data files (GFF3 and aa fastas), sample data for testing code, and a folder that contains some preliminary data for the distribution of "neuronal" genes across non-bilaterians. The latter is here for a potential future project. Note that the actual genome scaffold fastas are not included - they were too big.  
* **output**: output from running the code.  
* **readme_figs**: the figures I've embedded in this readme are found here.  

## Introduction

This is a final project for the [Comparative Genomics](https://github.com/Yale-EEB723/syllabus) seminar in the spring of 2019. I aim to establish gene loss across broad evolutionary distances by synteny analysis. I will take a macrosynteny approach, establishing synteny and subsequently searching for gene loss between distantly related taxa at the whole-scaffold level.  

## Background  
While biologists characterize most animals by what they have, non-bilaterians are sometimes characterized by what they don't have. In sponges, traditionally these absences have been considered ancestral, an assumption contributing to their primitive image.  

Loss is trivial to establish phylogenetically, but is less so when the phylogeny itself possesses a degree of uncertainty. Synteny analysis provides one method of determining loss. In particular, the ghost locus hypothesis suggests that in the case of gene loss, the synteny surrounding the locus of the gene may be preserved even in the absence of the gene itself (Ramos et al., 2012). In this project, I will identify candidate 'ghost loci' across broad evolutionary distances.  

Many studies examining synteny across long evolutionary distances choose a microsynteny approach. Is it possible to identify synteny at a macro, whole-scaffold level?  

## Goals

This project can be broken up into 3 goals:  
1. Characterize the genomes. Is synteny analysis possible?  
* How contiguous are the genomes?  
* How many scaffolds have at least three genes?  
2. Identify homologous genes on scaffolds. This requires running Agalma and learning how to parse GFF3 files.  
3. Combine the data in (2) to identify candidates for gene loss ('ghost loci').  


## The data  

**Data source**: All genomes and GFF3 files were downloaded from Ensembl. In general, only the toplevel assembly was available, so all synteny analysis was run on either toplevel or primary assemblies. GFF3 statistics were run on chromosomal level assemblies for Drosophila, Danio, Homo and Taeniopygia since the presence of many short scaffolds lacking annotated genes produced misleading results.    

**The genome versions:**    
Amphimedon queenslandica: Aqu1  
Capitella telata: Capitella telata v1.0  
Danio rerio: GRCz11  
Drosophila melanogaster: BDGP6.22.96   
Helobdella robusta: Helro1  
Homo sapiens: GRCh38  
Lottia gigantea: Lotgi1  
Mnemiopsis leidyi: MneLei_Aug2011  
Nematostella vectensis: ASM20922v1  
Strongylocentrotus pupuratus: Spur_3.1  
Taeniopygia guttata: taeGut3.2.4  
Trichoplax adhaerens:  ASM15027v1  

**Data structure**:  
- genome FASTA files: nt scaffolds, peptide  
- genome GFF3 files  


## Results
### Broad Sampling Across the Animal Tree  
These are the animals chosen for this project. They were selected to represent a broad swath of diversity across Metazoa.  

![phylogeny](readme_figs/phylogeny.png)  

### Is Synteny Analysis Possible?   
Synteny analysis, which analyzes the spatial structure of a genome, greatly depends on the use of high quality assemblies. How does assembly contiguity vary across the animal tree?  

#### Most Genomes Have Short Scaffolds   
There is a momentous difference in median scaffold length between assemblies from model organisms (Homo sapiens, Danio rerio, Drosophila melanogaster, and Taeniopygia guttata) and other animals. The human genome possesses the highest median scaffold length (approximately 133 million base pairs). In contrast, Strongylocentrotus posseses the shortest median scaffold length (just 1006 bp). 

![med.scaff.len](readme_figs/rm.med.scaff.len.png)  

 Some examples of the scaffold length distribution for various species is given below. I have limited the x-axis to 10,000 bp since median scaffold length of all species was below 10,000 bp. The exceptions are the model organisms, including Taeniopygia, which is displayed here. For non-model animals, distributions were right-skewed with several small peaks as scaffold length increased. These smaller peaks were seen in Amphimedon, Trichoplax, Nematostella Capitella, Helobdella, Lottia.  

![scaff.len.per.species](readme_figs/rm.scaff.len.per.species.examples.png)  

#### Most Genomes Are Highly Fragmented   
A similar divide between model organisms and all other animals is seen in the degree to which assemblies are fragmented. Chromosome-level assemblies are available for only the model organisms (Homo sapiens, Danio rerio, Drosophila melanogaster, Taeniopygia guttata), and thus were used here. All other assemblies were primary or top-level assemblies, and possessed over 1,000 scaffolds each. In particular, Strongylocentrotus, again, possessed over 32,000 scaffolds. Danio rerio possessed the fewest number of chromosomes: 8.  

![chr.no.](readme_figs/rm.chr.no.png)  

#### Scaffolds With At Least 3 Genes Per Scaffold  
Synteny analysis requires at least 3 genes on a scaffold - two on either side of the gene of interest to serve as anchors delineating the synteny block, and the gene of interest itself. To what degree do my genomes satisfy this requirement?  

Excluding the model organisms, the proportion of scaffolds that possess at least 3 genes is less than 10% in most animals. Two exceptions are Mnemiopsis leidyi (20%) and Amphimedon queenslandica (17%).  

![prop.contig.>3](readme_figs/rm.prop.contig.morethan3.png)  

However, because the assemblies of non-model animals are highly fragmented, this low proportion still translates into a high number of scaffolds with three or more genes.  

![no.contig.>3](readme_figs/rm.no.contig.morethan3.png)  

Showing only the proportion *or* number of scaffolds alone is misleading. Having substantially more scaffolds will simulataneously decrease the proportion with > 3 genes but increase the count of scaffolds with > 3 genes. For the latter metric, highly contiguous genomes (Homo, Danio, Taeniopygia, Drosophila) will possess many fewer scaffolds. A more ideal metric may be a normalized ratio of the number:proportion of scaffolds with > 3 genes.  

### Ghost Loci Analysis   

#### Workflow  
I pursued the strategy of aggregating data at each step into a master table. Although most analyses were performed on subsets of this table, the master table remains available for use in future analyses (eg. I included start and end gene coordinates for collinearity analysis).  

Below is an overview of my workflow, elaborated from my presentation. The steps are:  
1. Parse the GFF3 files.  
2. Run Agalma and parse Agalma output files.    
3. Aggregate the data to identify homologous genes on scaffold.  
4. Input this aggregated data into a contingency table, make a distance matrix, and perform cluster analysis.  
5. Combine the GFF3, homolog ID, and cluster ID data to form a master table.  
6. Use this master table to find ghost loci. I take a phylogenetic approach.     

Greater detail can be found in the Methods section.   

![workflow](readme_figs/rm.workflow.png)  

For the following analysis, Agalma was run on the entire 12 species dataset. However, only Homo sapiens, Danio rerio and Strongylocentrotus purpuratus were selected to form the contingency table, which was subsequently downsampled to a 100x100 matrix. Thus, clustering was performed only on Homo, Danio and Strongylocentrotus.  

#### Clustering Result

I chose to use the Cross-clustering algorithm. Cross-clustering is robust to outliers and does not require a priori knowledge of k, the 'true' number of clusters (Tellaroli et al., 2016). It achieves this by combining principles from 2 clustering algorithms: Ward's minimum variance and complete linkage. As Ward's builds clusters by minimizing the squared distances of points from the cluster centroid, it is good for shaping clusters and estimating the optimal number of clusters. Complete linkage is used to identify outliers based on the fact that it uses the maximum distance between two points in different clusters as an estimate of the overall proximity between those clusters.  

I visualized clustering with a tSNE. In the plot below, each dot represents a scaffold, and dots are colored according to the cluster it was assigned to by the Cross-clustering algorithm.  

There is a single large cluster (cluster 2) containing the majority of the points. The remaining "clusters" have extremely low membership, most often consisting of a single scaffold.  

![HsaDreSpu100 p=1 main cluster](readme_figs/rm.HsDreSpu100_cluster_main.png)  

It is possible this pattern arises from the clustering algorithm. Cross-cluster deals with outliers by excluding them from clustering - singleton clusters may represent outliers. Tellaroli et al. (2016) suggest that obtaining a single large cluster from cross-clustering suggests that clustering the data is not appropriate. However, for my previous rotation project, where I attempted to cluster different animal cell types, a different clustering algorithm (DBScan) produced the same clustering pattern. This pattern is further repeated using several other clustering algorithms and data sets (see Clustering Troubleshooting).  

#### Ghost Loci Analysis Output  
By default, genes that arose after the emergence of a particular clade will not be present in that clade. As seen in my presentation, I had initially pursued a 'nesting' approach: I assigned each tip a number, which increased in value the more successively nested that tip was. However, following your advice I've pursued a phylogenetic approach. For each homolog, I found its most recent common ancestor and all the taxa within the clade defined by this common ancestor. Instances where a tip is a member of this clade yet lacks the homolog is flagged as a candidate gene loss event.  

The final output looks like this:  
![absence analysis output](readme_figs/rm.absence.analysis.output.png)  

Where:  
* **species_scaffold**: The scaffold in which the gene is found. Species name is concatenated to scaffold ID to create a unique scaffold identifier.   
* **gene**: ID of gene on scaffold.  
* **homology_id**: homology ID of that gene (from Agalma).  
* **absent**: The taxon from which the gene is absent. Currently these are represented by the tip numbers of the tree but I will eventually include the species name instead. The above code is really new and hasn't been perfected yet or as thoroughly debugged as I would like.   

## Clustering Troubleshooting  
There are at least three elements to clustering: the data, the algorithm used to create the distance matrix, and the clustering algorithm.  

### Is the contingency matrix underlying the distance matrix too sparse?  
Is it possible that all scaffolds appear similar because most scaffolds share few genes? In other words, do many scaffolds share the absence of genes? Is the contingency matrix mainly occupied by zeros?  

Using the entire data set (not subsetted or downsampled) with the default Agalma bitscore threshold, I counted the number of scaffolds each homolog was found on. This produced the below histogram. 13,685 homologs were shared across 9,183 scaffolds. The median number of scaffolds a homolog was found on was 9. The minimum was 1 and the maximum was 306.  

![scaffold occupancy](readme_figs/rm.scaffold.occupancy_nohydraALL.png)

Thus, most frequently, a homolog is shared across ~0.1% of the scaffolds. This is indeed low.  

### Testing different subsets of data   
The same clustering algorithm (cross-clustering) was used on different subsets of data.  

#### relaxedbit data set  
My project compares sequences of very distantly related taxa. As such, lowering the BLAST bit score threshold to 100 when running Agalma's Homologize script may increase the number of identified homologs. All animal genomes were run through Homologize with this relaxed threshold. A contingency matrix of Homo, Danio, and Strongylocentrotus sequences was made and subsequently downsampled into a 100x100 matrix.  

The scaffold occupancy profile improved, but not by much. The number of scaffolds with homologs increased to 17,353, as did the number of homologs (19,016). However, the median number of scaffolds occupied per homolog was 11, and the maximum 600.  

![scaffold occupancy relaxed bit](readme_figs/rm.scaffold.occupancy_relaxedbit.png)  

Clustering with cross-clustering produced a familiar pattern: one big cluster plus many small clusters with a low number of members per cluster.   

![relaxedbit cluster](readme_figs/rm.relaxedbit_cluster.png)  

One anomaly in the Agalma output is that the same gene can have multiple homology_ids. As far as I can tell, this does not occur in any of the other Agalma runs that used the higher bitscore threshold.  

#### 5taxa dataset:  
Another way to increase the number of 'shared' genes is to limit our comparison to a smaller subset of closely related, high quality genomes. Sequences from Strongylocentrotus, Danio, Homo, Drosophila, and Taeniopygia were run through Agalma Homologize (at normal bitscore threshold). Homo, Danio, and Strongylocentrotus were then used to create a contingency matrix, which was ultimately downsampled into a 100x100 matrix.  

7,248 homologs were shared across 3,466 scaffolds. The median number of scaffolds each homolog occupied was 5; the maximum occupied was 218. 

![scaffold occupancy 5taxa](readme_figs/rm.scaffold.occupancy_5taxa.png)  

Again, the familiar clustering pattern is produced by cross-clustering:  

![5 taxa cluster](readme_figs/rm.5taxa_cluster.png)   

Agalma won't run on data from only 3 taxa, which is why I tried 5. Strongylocentrotus was included because it is closely related to Homo and Danio, but its genome is in fact highly fragmented. It may be useful to try this again, but excluding Strongylocentrotus. Using chromosome-level assemblies (where possible) might also improve this outcome.    

### Testing different clustering algorithms   
I tested if basic clustering algorithms that lack cross-clustering's outlier approach produce the same pattern. The data set was kept constant.  

**Data**: The 5taxa dataset was used. Only Homo and Danio were selected for clustering, the reasoning being that clustering only two high very high quality genomes would give the highest likelihood of success. This contingency table was then downsampled to a 100x100 matrix.  

#### agnes: aggregative hierarchical clustering  
*Algorithm*: Initially, each data point is declared a cluster. Each sucessive step the nearest clusters are combined to form a larger cluster, until all points form a single cluster. [R documentation](https://www.rdocumentation.org/packages/cluster/versions/2.0.7-1/topics/agnes)  

![agnes](readme_figs/rm.agnes.dendrogram.png)  

#### diana: divisive hierarchical clustering  
*Algorithm*: All data points are initially members of a single large cluster. At each step, diana searches for the two most distant points, then reassigns all other points based on whether they are closer to the "splinter group" than the "old party" (as worded in the R documentation). This continues until every point is relegated to its own cluster. [R documentation](https://astrostatistics.psu.edu/su07/R/html/cluster/html/diana.html)      

![diana](readme_figs/rm.diana.dendrogram.png)  

Both agnes and diana produce a dendrogram. The general pattern I've received from all other clustering attempts is reiterated in the dendrogram structure. There is one large cluster, following by many little clusters of low membership consecutively splitting off.  

#### k-means: clustering around centroids  
*Algorithm*: Start by choosing k, the number of clusters. K random data points are selected to serve as the initial centroids of k clusters, then each observation is grouped into the cluster with the closest centroid. Centroids are re-calculated based on the new members. Re-arrangment of clusters continue with a goal of minimizing the total within sum of squares until the number of different clusters do not change.  

For the data I am testing, the elbow plot suggests that the optimal k is 2.  

![kmeans elbow plot](readme_figs/rm.kmeans_elbowplot.png)  

The resulting clustering appears below:  

![kmeans cluster](readme_figs/rm.kmeans.cluster.png)  

## Testing the null hypothesis  
Given that different subsets of data and clustering algorithms seem to return the same general clustering structure, is it possible that this structure is inherent to this type of data itself? Is this clustering structure the 'true' result given the resolution afforded by this work-flow, or is it a random pattern?  

*Null hypothesis*: The clustering pattern I have been observing so far (one big cluster + many small singleton clusters) is due to random chance. If so, the clustering pattern created by a random contingency matrix should match the pattern I've received in previous clustering attempts.    

The randomized data was based off the original data from the Ghost Loci Analysis and the same clustering algorithm was used. The below tSNE is designed to be compared to the original tSNE in the Ghost Loci Analysis section.  

**Data**: Genomes from all animals were homologized, then gene presence/absence data was used to create a contingency matrix of gene residency per scaffold. Only Homo, Danio and Strongylocentrotus sequences were included in the contingency matrix, which was subsequently downsampled into a 100x100 matrix.   

For randomization, I found the proportion of 1's (gene present on scaffold) vs 0's (gene absent) in the 100x100 contingency matrix. Then, a random 100x100 matrix of 1's  and 0's was created, with proportions matching the original proportions of 1's and 0's seen in the real data. Simply randomizing the order of the rows or columns would change the labels of the clusters, but would not test the overall configuration of the clusters themselves.  

**Clustering algorithm**: Cross-clustering.  

![null cluster](readme_figs/rm.null.cluster.png)  
 
Scaffolds do *not* cluster into a single large cluster! I have assumed that the clustering pattern I've been receiving is incorrect, but perhaps it is a true reflection of the structure of the data. For example, perhaps the scaffolds are clustering into two categories: high vs low quality. I am currently investigating the identity of the scaffolds that are members of the large cluster. Alternatively, while this pattern may be true at a high level, clustering methods may lack the resolution to identify relationships at finer levels. It is interesting that I receive similar clustering profiles from clustering scaffolds by gene presence/absence and clustering cell types by gene presence/absence across broad distances.       


## Assessment  
*Was it successful in achieving the initial goal?*  
Yes, this project completed all three goals it set out to achieve. However, due to the difficulty in interpreting the results of the clustering step, the results of the ghost loci analysis is also less clear.  

*What are the main obstacles encountered?*  
* Computation: My laptop did not have enough computing power to process the dataset in its entirety. The next step is to run R on the cluster.  
* Clustering: As discussed, I could not resolve tight clusters.  

*What would you have done differently?*  
* Include animal outgroups (choanoflagellates etc.)  
* I've received advice on other clustering strategies:  
  * Antonio: Perform local clustering and link them together like a daisy chain  
  * Zack: Go through genes on a scaffold using a sliding 10-gene window. If two homologs within this window are also on the same scaffold in another organism, this suggests a possible syntenic region.  Quantify how many of these pairs you can find in the window.  
  * Casey: Instead of making a homolog x scaffold contingency matrix, make a homolog x homolog matrix. (*I'm still not fully clear about this4 - can we go over this again, Casey?*)  

*What are future directions this could go in?*  
* Ancestral state reconstruction - it is difficult to study what is absent, but perhaps this can be achieved by reconstructing the gene that was lost.    
* Study system-specific questions eg. Have sponges lost a nervous system? This is the question that started the project.  
* Study the characteristics of the genes that have been lost eg. are they orthologs or paralogs? Genes that arose earlier or later?  
* WAAAAAAAAY MORE!!! I would like to write a more in depth document should we decide this is a promising direction to go.  


## Methods  

All code was written in R, except where indicated.  
Load libraries:    

```
library( tidyverse )
library(ggplot2)
library(seqinr)
library(ape)
library(Biostrings)
library(dplyr)

#clustering
library(cluster)
library(CrossClustering)
library(vegan)
library(Rtsne)
library(factoextra)

#tree manipulation
library(ape)
library(hutan)
```

### Genome Statistics  

#### What is the distribution of scaffold lengths?   
For each genome, the below code plots a histogram of scaffold lengths and finds the median, maximum, and minimum scaffold lengths.  

The human and zebrafish fastas were too large to run through this code, but median scaffold length was found from online database resources. The many small unplaced scaffolds in the Drosophila toplevel assembly reduced median scaffold length, so only chromosomes were used.  

```
{r distr scaffold length}
fasta_dir<-"~/Desktop/compgen/repo/ensembl/genome_fasta/"
fasta<-c("Amphimedon_queenslandica.Aqu1.dna.toplevel.fa","Capitella_teleta.Capitella_teleta_v1.0.dna.toplevel.fa","Drosophila_chr.fa", "Helobdella_robusta.Helro1.dna.toplevel.fa","Lottia_gigantea.Lotgi1.dna.toplevel.fa","Mnemiopsis_leidyi.MneLei_Aug2011.dna.toplevel.fa","Nematostella_vectensis.ASM20922v1.dna.toplevel.fa","Strongylocentrotus_purpuratus.Spur_3.1.dna.toplevel.fa","Taeniopygia_guttata.taeGut3.2.4.dna.toplevel.fa","Trichoplax_adhaerens.ASM15027v1.dna.toplevel.fa")

for (blah in fasta){
fasta_path<-paste0(fasta_dir,blah)  

#histogram of call contig lengths,
scaffold.length.plot<-read.fasta(fasta_path)%>%getLength(.)%>%as.data.frame()%>%ggplot()+geom_histogram(aes(x=.), fill="skyblue3", color="black",binwidth=100)+xlab("Scaffold Length")+labs(title=blah)
scaffold.length.plot

#histogram of contig lengths, restricted to scaffolds < 10,000 bp. 
restricted.plot<-read.fasta(fasta_path)%>%getLength(.)%>%as.data.frame()%>%filter(.<10000)%>%ggplot()+geom_histogram(aes(x=.), fill="skyblue3", color="black",binwidth=100)+xlab("Scaffold Length")+labs(title=blah)
restricted.plot

#stats
fasta.len=read.fasta(fasta_path)%>%getLength()
print(paste("Median scaffold/contig length (bp):",median(as.numeric(fasta.len))))
print(paste("Max scaffold/contig length (bp):",max(as.numeric(fasta.len))))
print(paste("Minimum scaffold/contig length (bp):",min(as.numeric(fasta.len))))
  
}

```

#### How many scaffolds have at least 3 genes?  
For a given genome, this code plots a frequency histogram of the number of genes per scaffold. For each genome, it also finds the proportion of contigs with at least 3 genes, the median number of genes per contig, and the maximum number of genes per contig.   
```
{r count no genes per contig/scaffold}

gff3_dir<-"data/genomes/gff3/"
gff3<-c("Amphimedon_queenslandica.Aqu1.42.gff3","Capitella_teleta.Capitella_teleta_v1.0.42.gff3","Danio_rerio.GRCz11.96.chr.gff3","Drosophila_melanogaster.BDGP6.22.96.chr.gff3","Helobdella_robusta.Helro1.42.gff3","Homo_sapiens.GRCh38.96.chr.gff3","Lottia_gigantea.Lotgi1.42.gff3","Mnemiopsis_leidyi.MneLei_Aug2011.42.gff3","Nematostella_vectensis.ASM20922v1.42.gff3","Strongylocentrotus_purpuratus.Spur_3.1.42.gff3","Taeniopygia_guttata.taeGut3.2.4.96.chr.gff3","Trichoplax_adhaerens.ASM15027v1.42.gff3")

for (blah in gff3){
  
gff3_path<-paste0(gff3_dir,blah)
contig.gene.df <- ape::read.gff(file=gff3_path, GFF3=TRUE) %>% select(.,"seqid","type","start","end") %>% filter(.,type == "gene")

#for histogram, take just the contig names and gene counts
gene.count<- as.data.frame(table(contig.gene.df$seqid))

#plot frequency of frequencies
gff3.plot<-ggplot(data = gene.count, aes(x=gene.count$Freq)) + geom_histogram(binwidth=1) + geom_vline(xintercept = 3,color="red")+ggtitle(blah)
gff3.plot

#use for custom axes
#gff3.plot<-ggplot(data = gene.count, aes(x=gene.count$Freq)) + geom_histogram(binwidth=1) + geom_vline(xintercept = 3,color="red") +xlim(0,10)+ylim(0,3000)

#stats
print(blah)

print(paste("proportion contigs with gene count >= 3:",nrow(subset(gene.count, gene.count$Freq >= 3))/nrow(gene.count)*100))

print(paste("Median no. genes/contig:",median(gene.count$Freq)))

print(paste("Max no. genes/contig:",max(gene.count$Freq))) 

}

```
### Prep sequences for Agalma  
#### Simplify sequence names  
Download genome protein sequences from Ensembl. 

Agalma cuts off sequence names after the first space. In addition to this, some genomes possess gene, peptide and transcript IDs that don't follow a simple pattern (i.e. cannot "translate" a peptide ID into its transcript ID - the names are seemingly random).

Simplify sequence names while retaining information on the scaffold, gene, transcript and peptide IDs. I used the following format:   
`>p:protID:s:scaffoldID:g:geneID:t:transID`  

To simpify, use Regex "Find":  
Amphimedon:  
>`>(.*?)@pep@scaffold:Aqu1:(Contig\w+):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`  

Capitella:  
>`>(.*?)@pep@supercontig:Capitella_teleta_v1.0:(CAPTEscaffold_\w+?):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*?)@.*?$`  

Danio:    
>`>(.*?)@pep@chromosome:GRCz11:(\w+):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotype.*`  

Drosophila:    
>`>(.*?)@pep@chromosome:BDGP6:(\w+):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.* `  
  
Helobdella:  
>`>(.*?)@pep@supercontig:Helro1:(\w+):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`  

Homo:   
-need chromosome version AND scaffold version    
>`>(.*?)@pep@scaffold:GRCh38:(.*):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`    
`>(.*?)@pep@chromosome:GRCh38:(.*):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`   
 
Danio:    
>`>(.*?)@pep@chromosome:GRCz11:(\w+):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`    
`>(.*?)@pep@scaffold:GRCz11:(.*):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`    

Lottia:  
>`>(.*?)@pep@supercontig:Lotgi1:(.*):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`  
  
Mnemiopsis:     
>`>(.*?)@pep@supercontig:MneLei_Aug2011:(.*):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`    
`>(.*?)@pep@chromosome:MneLei_Aug2011:(.*):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`      
Nematostella:  
>`>(.*?)@pep@supercontig:ASM20922v1:(.*):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`    

Strongylocentrotus:    
>`>(.*?)@pep@supercontig:Spur_3.1:(.*):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`    

Taeniopygia:  
>`>(.*?)@pep@chromosome:taeGut3.2.4:(.*):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`    

Trichoplax:  
>`>(.*?)@pep@scaffold:ASM15027v1:(.*):\w+?:\w+?:.*?@gene:(.*?)@transcript:(.*)@gene_biotyp.*`  


Then, Replace with `>p:\1:s:\2:g:\3:t:\4`.  

#### Select longest peptide only  
Comparing the number of genes in the gff3 file vs the number of sequences in the peptide fasta, it becomes clear that many genes have more than one peptide associated with it:    
no. genes: `grep "gene:" gff3.gff3|wc -l`  
no. peptides: `grep ">" pepfile.fasta|wc -l`  

Create a fasta where each gene has only 1 peptide sequence. Choose the longest peptide per gene.  

The below code was adapted from code by [story (Jan 9'17')](https://bioinformatics.stackexchange.com/questions/595/how-can-longest-isoforms-per-gene-be-extracted-from-a-fasta-file) and [nassimhddd (Jul 18 '16)](https://stackoverflow.com/questions/24237399/how-to-select-the-rows-with-maximum-values-in-each-group-with-dplyr).  

**IMPORTANT NOTE**: Apr28: just noticed a bug. The group_by() doesn't work correctly if you also load the plyr library.  

```{r longest pep only}
#install bioconductor 3.8 https://www.bioconductor.org/install/  
#install Biostrings: BiocManager::install("Biostrings", version = "3.8") ; do not update mclust because it will have a warning and then biostrings module not installed.  

#~~~~~ Beginning of 'story''s code.~~~~~

## read your fasta in as Biostrings object
fasta.s <- readDNAStringSet("./data/sample_data/longest_pep_sample.fasta")

## get the read names (in your case it has the isoform info)
names.fasta <- names(fasta.s)

## extract only the relevant gene and isoform (JM: aka pep) id (split name by the period symbol)
#JM: modify line to fit my name format
gene.iso <- sapply(names.fasta,function(j) cbind(unlist(strsplit(j,'\\:'))[1:8]))

## convert to good data.frame = transpose result from previous step and add relevant column names
gene.iso.df <- data.frame(t(gene.iso))
#JM: alter to put your own column names in:
colnames(gene.iso.df) <- c('p','pepID','s','scaffID','g','geneID','t','transID')
gene.iso.df<-select(gene.iso.df, geneID, pepID)

## and length of isoforms
gene.iso.df$width <- width(fasta.s)

#Did not use rest of code.

#~~~~~ End of 'story''s code.~~~~~

#JM:
gene.iso.df<-rownames_to_column(gene.iso.df)

#~~~~~beginning of code by nassimhddd (Jul 18 '16) Stackoverflow~~~~~
longest.pep.df <-gene.iso.df%>%dplyr::group_by(geneID)%>%mutate(the_rank = rank(-width,ties.method="random")) %>%filter(the_rank==1)%>%select(-the_rank)
#~~~~~End of nassimhddd's code.~~~~~

list<-longest.pep.df[,1,drop=FALSE]
write.csv(list,"longestpep.txt",quote=FALSE,row.names=FALSE)
```  

The code above produces a list of all of the longest peptides per gene. Use the below Python script I wrote to pull out each sequence from the fasta file by name. The fasta headers in this list must perfectly match those in the fasta file, and names must be in a single column with no column header. This means that you have to regex the desired ID (in this case, protein ID), from the list of fasta headers.  

```
#! /usr/bin/env python
import sys
import re

#def
identity = 0.0
Namefile = []
keep = False

#pull out sequences from a fasta file by name
#usage: fasta_puller.py <listofseqnames> <fastafile.fasta>  
#seq names should be listed in a single column and must exactly match fasta headers.

#open the file with list of sequence names 
#put each name into the list Namefile
File = sys.argv[1]
File = open(File, 'rU')
for Line in File:
	Line = Line.strip('\n')
	Namefile.append(Line)
File.close()

#open the transcriptome file from which sequences are to be pulled from
#since names must match exactly, if you have >TR_blah_blah|cblah_gblah_iblah as name only must alter fastsa names to match 

File = sys.argv[2]
File = open(File, 'rU')
for Line in File:
	Line = Line.strip('\n')
	
	if Line[0]==">":				#for name lines
		keep = False
		for Name in Namefile:
			if Name == Line:
				print Name
				keep = True
				#Namefile.remove(Name)	#hash out if working with gene names only, but want to keep multiple isoforms
	if Line[0] != ">":				#for non-name, ATCG lines
		if keep == True:
			print Line

``` 

### Run Agalma
A single run-through of Agalma consists of three scripts:    
**1. Catalog the fasta files**  
```
#!/bin/bash
#SBATCH --partition=general
#SBATCH --job-name=agalma_catalog_compgen_noNA_nohydra
#SBATCH -c 1
#SBATCH --mem-per-cpu=6G
#SBATCH --time=0-02:00:00
#SBATCH --mail-type=ALL

export AGALMA_DB="./compgen_homologs_nohydra.sqlite"

source activate /gpfs/ysm/project/jlm329/conda_envs/agalma

set -e

agalma catalog insert -p ../pep/Amphimedon_longestpep.fasta -s 'Amphimedon queenslandica' --id 'Aqu_7c2ad7b'
agalma catalog insert -p ../pep/Capitella_longestpep.fasta -s 'Capitella telata' --id 'Cte_aef8c97'
agalma catalog insert -p ../pep/Danio_longestpep.fasta -s 'Danio rerio' --id 'Dre_a1c97d2'
agalma catalog insert -p ../pep/Drosophila_longestpep.fasta -s 'Drosophila melanogaster' --id 'Dme_05eee07'
agalma catalog insert -p ../pep/Helobdella_longestpep.fasta -s 'Helobdella robusta' --id 'Hro_793004c'
agalma catalog insert -p ../pep/Homo_longestpep.fasta -s 'Homo sapiens' --id 'Hsa_f36e9f9'
agalma catalog insert -p ../pep/Lottia_longestpep.fasta -s 'Lottia gigantea' --id 'Lgi_e03f004'
agalma catalog insert -p ../pep/Mnemiopsis_longestpep.fasta -s 'Mnemiopsis leidyi' --id 'Mle_309f7a5'
agalma catalog insert -p ../pep/Nematostella_longestpep.fasta -s 'Nematostella vectensis' --id 'Nve_2427268'
agalma catalog insert -p ../pep/Strongylocentrotus_longestpep.fasta -s 'Strongylocentrotus purpuratus' --id 'Spu_a9c5697'
agalma catalog insert -p ../pep/Taeniopygia_longestpep.fasta -s 'Taeniopygia guttata' --id 'Tgu_290e6b7'
agalma catalog insert -p ../pep/Trichoplax_longestpep.fasta -s 'Trichoplax adhaerens' --id 'Tad_bdc7fb0'
```

**2. Import**
```
#!/bin/bash
#SBATCH --partition=general
#SBATCH --job-name=agalma_import_compgen_nohydra
#SBATCH -c 16
#SBATCH --mem-per-cpu=6G
#SBATCH --time=2-00:00:00
#SBATCH --mail-type=ALL

export AGALMA_DB="./compgen_homologs_nohydra.sqlite"

source activate /gpfs/ysm/project/jlm329/conda_envs/agalma


set -e


export BIOLITE_RESOURCES="threads=16,memory=6G"


agalma import --id Aqu_7c2ad7b --seq_type aa
agalma annotate --id Aqu_7c2ad7b

agalma import --id Cte_aef8c97 --seq_type aa
agalma annotate --id Cte_aef8c97

agalma import --id Dre_a1c97d2 --seq_type aa
agalma annotate --id Dre_a1c97d2

agalma import --id Dme_05eee07 --seq_type aa
agalma annotate --id Dme_05eee07

agalma import --id Hro_793004c --seq_type aa
agalma annotate --id Hro_793004c

agalma import --id Hsa_f36e9f9 --seq_type aa
agalma annotate --id Hsa_f36e9f9

agalma import --id Lgi_e03f004 --seq_type aa
agalma annotate --id Lgi_e03f004

agalma import --id Mle_309f7a5 --seq_type aa
agalma annotate --id Mle_309f7a5

agalma import --id Nve_2427268 --seq_type aa
agalma annotate --id Nve_2427268

agalma import --id Spu_a9c5697 --seq_type aa
agalma annotate --id Spu_a9c5697

agalma import --id Tgu_290e6b7 --seq_type aa
agalma annotate --id Tgu_290e6b7

agalma import --id Tad_bdc7fb0 --seq_type aa
agalma annotate --id Tad_bdc7fb0
```

**3. Homologize, align, and make gene trees**
```
#!/bin/bash
#SBATCH --partition=general
#SBATCH --job-name=agalma_genetrees_compgen_nohydra
#SBATCH -c 16
#SBATCH --mem-per-cpu=6G
#SBATCH --time=6-00:00:00
#SBATCH --mail-type=ALL

# Script must be run from same directory as existing sqlite database
export AGALMA_DB=$(pwd)"/compgen_homologs_nohydra.sqlite"

source activate /gpfs/ysm/project/jlm329/conda_envs/agalma

set -e


export BIOLITE_RESOURCES="threads=16,memory=6G"

ID=CompGen_nohydra

mkdir -p $ID
cd $ID

agalma homologize --id $ID
agalma multalign --id $ID
agalma genetree --id $ID
```
If Agalma fails at any step, delete all files and start fresh (RE-search!).  

Parse the Agalma output.
```
SELECT DISTINCT
       homology.id          AS homology_id,
       genes.gene           AS gene,
       models.catalog_id    AS catalog_id,
       models.id            AS id,
       homology_models.homology_id,
       homology_models.model_id,
       sequences.model_id,
       genes.model_id
FROM agalma_homology        AS homology        JOIN
     agalma_homology_models AS homology_models JOIN
     agalma_models          AS models          JOIN
     agalma_genes           AS genes          JOIN
     agalma_sequences       AS sequences
     ON homology.id=homology_models.homology_id AND
        homology_models.model_id=models.id AND
        models.id=sequences.model_id AND
        models.id=genes.model_id
ORDER BY homology.id ASC;
```
To run the database query:
`sqlite3 -csv -header homologs.sqlite < query_homology.sql > homology_results.csv`

### Agalma Output  
A few things will make life easier down the road when parsing the Agalma output.  

Although the simplified fasta header names are useful for downstream analyses, when merging tables the gene names must exactly match those in the gff3 files. Use Regex Find: `(.*,)p:.*?:s:.*?:g:(.*?):t:.*?(,.*)` and Replace: `$1$2$3`.  

Some of the gene names in the peptide files do not match the gene names in the gff3 files - they have an extra decimal point at the end of their names (I think indicating assembly version). This only occurred in Danio, Homo and Taeniopygia. Remove the decimal point wiht Regex Find:    
Danio:    
>`(\d+,ENSDARG.*).\d+(,Dre_.*)`  
>`(.*,ENSDARG\d+)\.(,Dre_.*)` -> seemed to do the trick better - needed two rounds

Homo:  
>`(\d+,ENSG.*?)\.\d+(,Hsa_.*)`  

Taeniopygia:  
>`(\d+,ENSTGUG.*?).\d+(,Tgu_.*)`  

### Parse the GFF3 and Agalma files  
#### Parse GFF3 files: 
1. For each GFF3 file, filter so that only lines where type = 'gene' are extracted. It may seem more direct to filter for 'gene_id', but pseudogenes and ncRNA_genes also have a gene_id. For each species, add these lines to a single dataframe.  
2. Grep out the gene_id and create a new column, 'gene'.  
3. From this, create **gff3.table**: select only the 'seqid' (aka scaffold ID), 'start', 'end', and 'gene' columns, and add a new column called 'animal' that indicates the species name for each gene.    

```
{r Parse GFF3 and Agalma data}


#~~~ Create GFF3 Table ~~~

#set up empty df to rbind to
all.df<-data.frame(matrix(ncol=5,nrow=0))
names<-c("seqid","type","start","end","attributes")
colnames(all.df)<-names

#set up file names to loop through
gff3_dir="data/genomes/gff3/"
gff3=c("Amphimedon_queenslandica.Aqu1.42.gff3","Capitella_teleta.Capitella_teleta_v1.0.42.gff3","Danio_rerio.GRCz11.95.gff3","Drosophila_melanogaster.BDGP6.95.gff3","Helobdella_robusta.Helro1.42.gff3","Homo_sapiens.GRCh38.95.gff3","Lottia_gigantea.Lotgi1.42.gff3","Mnemiopsis_leidyi.MneLei_Aug2011.42.gff3","Nematostella_vectensis.ASM20922v1.42.gff3","Strongylocentrotus_purpuratus.Spur_3.1.42.gff3","Taeniopygia_guttata.taeGut3.2.4.95.gff3","Trichoplax_adhaerens.ASM15027v1.42.gff3")

#1. Read in each gff3 file and extract seqid, type, start, end, attributes. Filter lines by type = gene.
for (blah in gff3)
{
gff3_path<-paste0(gff3_dir,blah)
contig.gene.df <- ape::read.gff(file=gff3_path, GFF3=TRUE) %>% dplyr::select(.,"seqid","type","start","end","attributes") %>% filter(.,type == "gene")
all.df<-bind_rows(all.df,contig.gene.df)
}

#2. Grep out the gene_id from the attributes 
all.df$gene<-gsub(".*gene_id=(.*?);.*","\\1",all.df$attributes)

#3. Construct gff3 table
gff3.table<-select(all.df,"seqid","start","end","gene")

#add animal IDs to gff3 table
gff3.table$animal<-c(rep.int("Amphimedon_queenslandica",43615),rep.int("Capitella_telata",32175),rep.int("Danio_rerio",25606),rep.int("Drosophila_melanogaster",13931),rep.int("Helobdella_robusta",23432),rep.int("Homo_sapiens",21492),rep.int("Lottia_gigantea",23349),rep.int("Mnemiopsis_leidyi",16559),rep.int("Nematostella_vectensis",24773),rep.int("Strongylocentrotus_purpuratus",28987),rep.int("Taeniopygia_guttata",17488),rep.int("Trichoplax_adhaerens",11520))

#~~~ GFF3 Table Finished ~~~
```

#### Read in Agalma files   
`agalma.homologs<-read.csv("agalma_results.csv",header=TRUE,sep=",")`.   
This results in **agalma.homologs**. 

```
#Read in agalma homologs csv
agalma.homologs<-read.csv("5taxa_results_alt.csv",header=TRUE,sep=",")

#to double check that there are no duplicate genes in final.master.table:
#duplicated(final.master.table$gene)%>%table()
#~~~Final master table complete~~~

```

### Perform Cluster Analysis 
For cluster analysis, gff3.table and agalma.homologs will be inner-joined, then scaffold ID and homology ID will be selected to input into the contingency table.  

An inner join is performed for two reasons:   
1. Remove genes in gff3.table that were not assigned a homology_id by Agalma - these NAs do not add useful information for clustering.   
2. There are some human genes in the Agalma output that are missing from the filtered and unfiltered GFF3 files. Remove.

#### Cluster Analysis Steps  
1. Filter gff3.table by animal. It is not possible to run an analysis using data from all species. Here, we subsample whichever set of animals we want to cluster.  
2. Create **contingency.table**. As written above, inner join gff3.table x agalma.homologs. Scaffold IDs are not unique, so concatenate scaffold IDs to animal IDs. Select only species-scaffold ID and homology_ids.  
3. Create contingency table: column names are homology IDs, row names are species-scaffold IDs. 
* 1 = homology ID present in scaffold, 0 = homology ID absent in scaffold.  
Subsample contingency table 100x100 to allow computation on a laptop.  
4. Create distance matrix using gower. Gower is a widely used algorithm capable of working on a mix of continuous and categorical data. For categorical data, it uses Jaccard Similarty Index.  
5. Cluster.  
* cross-clustering  
* agnes    
* diana   
* kmeans  
6. Visualize clustering with tSNE or a dendrogram.

tSNE code is by [Daniel P. Martin](http://dpmartin42.github.io/posts/r/cluster-mixed-types).  

**Note**: You must run the previous chunk `{r Parse GFF3 and Agalma data}` before running this chunk.  

```
{r cluster}
#must run previous chunk prior to running this chunk


#***1. SUBSAMPLE: choose from which animals you want to cluster scaffolds. Not doing so makes computation impossible on a local machine.
filter_target=c("Homo_sapiens","Danio_rerio","Strongylocentrotus_purpuratus")
gff3.table<-filter(gff3.table, animal==filter_target)


#~~~2. Create contingency.table~~~
#Make scaffold names unique- join seqid and catalog id
contingency.table<-inner_join(gff3.table,agalma.homologs)%>%dplyr::select(.,seqid,homology_id,animal)%>%mutate(species_scaffold=str_c(seqid,animal,sep="_"))%>%dplyr::select(.,species_scaffold,homology_id)
#~~~contingency.table compelte~~~


#3. Use contingency.table to make the contingency table
contingency<-table(contingency.table$species_scaffold,contingency.table$homology_id)%>%as.matrix()
contingency<-1*(contingency>0)
#***DOWNSAMPLE contingency table for computation on a local machine.
contingency<-contingency[sample(100),sample(100)]


#4. distance matrix
dist.matrix<-daisy(contingency,metric="gower",stand=FALSE)


#5. cluster
#cross-cluster
cross.clust<-cc_crossclustering(dist.matrix,out=FALSE)
clust<-cc_get_cluster(cross.clust)
#plot w/ tSNE
#adjust perplexity if get error "perplexity is too large for the number of samples"
tsne_obj<-Rtsne(dist.matrix,is_distance=TRUE,perplexity=1)
tsne_data<-tsne_obj$Y%>%data.frame()%>%setNames(c("X","Y"))%>%mutate(cluster=factor(clust))
tsne_plot<-ggplot(aes(x = X, y = Y), data = tsne_data) + geom_point(aes(color = cluster))
tsne_plot

#agglomerative
ag<-agnes(dist.matrix,diss=TRUE)
plot(ag,labels=FALSE)

#divisive
di<-diana(dist.matrix)
plot(di,labels=FALSE)

#kmeans
kmean<-kmeans(dist.matrix,51)
kclust<-kmean$cluster
tsne_obj<-Rtsne(dist.matrix,is_distance=TRUE,perplexity=1)
tsne_data<-tsne_obj$Y%>%data.frame()%>%setNames(c("X","Y"))%>%mutate(cluster=factor(kclust))
tsn_plot_k<-ggplot(aes(x = X, y = Y), data = tsne_data) + geom_point(aes(color = cluster))
tsn_plot_k
#~~~Clustering complete~~~

```

### Create Master Table And Perform Ghost Loci Analysis    

#### Create Master Table   
The master table is formed by left-joining gff3.table x agalma.homologs. Then, unique scaffold IDs are created by concatenating scaffold IDs to animal IDs. In addition, cluster IDs from the previous step are added.   

Thus, the master table contains all information gathered so far: scaffold ID, start coordinates, end coordinates, gene ID, homology ID, and cluster ID.  

```
#~~~Create master.table.~~~

#Put together a scaffold:cluster df
clust.df<-as.data.frame(clust)
colnames(clust.df)<-"cluster_id"
clust.df<-mutate(clust.df,species_scaffold=row.names(contingency))

#Left join gff3 table x agalma homologs + scaffold ID + cluster ID
master.table<-left_join(gff3.table,agalma.homologs)%>%select(.,"seqid","start","end","gene","homology_id","catalog_id","animal")%>%mutate(species_scaffold=str_c(seqid,animal,sep="_"))%>%left_join(.,clust.df,by="species_scaffold")%>%select(.,species_scaffold,start,end,gene,homology_id,animal,cluster_id)
#~~~master.table complete~~~

```

#### Perform Ghost Loci Analysis  
I take a phylogenetic approach:    
1. Find all taxa that possess a particular homology_id.   
2. Find the most recent common ancestor of all those taxa.   
3. Find all the tips of the last common ancestor.  
4. Compare which tips do not possess the homology_id. These are candidate cases of gene loss.  

```{r tree}
#initialize
mrca.df<-data.frame()
taxa.list<-NULL


#phylogeny
tree_text = "(Mnemiopsis_leidyi,(Amphimedon_queenslandica,(Trichoplax_adhaerens,(Nematostella_vectensis,((Drosophila_melanogaster,(Lottia_gigantea,(Capitella_telata,Helobdella_robusta))),(Strongylocentrotus_purpuratus,(Danio_rerio,(Homo_sapiens,Taeniopygia_guttata))))))));"
phy = read.tree(text=tree_text)
plot(phy)


#create a list of unique homology_ids; do not include NAs
homolog.list<-unique(master.table$homology_id)
homolog.list<-homolog.list[!is.na(homolog.list)]


       
for (h in homolog.list){
#1.Find all taxa that possess a particular homology_id. 
homolog.animals<-master.table%>%filter(.,homology_id == h) %>% select(.,animal)%>%unique()

#2. Find the most recent common ancestor of all those taxa. 
#If homology_id is possessed by only 1 taxon (eg. shared among paralogs), then skip. By default, there is no opportunity for detecting absence.  
if (nrow(homolog.animals) == 1){
  next
}

if (nrow(homolog.animals) > 1){
  mrca <-getMRCA(phy,as.character(homolog.animals$animal))
  bind.df<-cbind(homology_id=h,mrca)
  mrca.df<-rbind(mrca.df,bind.df)
}

}

#Add MRCAs to master.table
new.master.table<-left_join(master.table,mrca.df, by="homology_id")


absent.df<-data.frame(h=integer(0),absent.in.taxa=integer(0))
for (h in mrca.df$homology_id){
#3. Find all the tips of the last common ancestor.  
all.tips<-mrca.df$mrca[mrca.df$homology_id==h]%>%descendants(phy,.)
all.tips<-all.tips[all.tips<=length(phy$tip.label)]

taxa<-new.master.table%>%filter(homology_id==h)%>%select(.,animal)%>%unique()


#first find the node number of all animals that taxa with that homology_id
for (t in taxa$animal){
  find.taxa<-which(phy$tip.label==t)
  taxa.list<-append(taxa.list,find.taxa)
}
#4. Compare which tips do not possess the homology_id.
#Where there are no instances in gene loss absent.in.taxa will equal integer(0). Hence need rbind.fill (not rbind)  
absent.in.taxa<-setdiff(all.tips,taxa.list)
taxa.list<-NULL

#append to table of homology_id + absent taxa
combine.df<-cbind(h,absent.in.taxa)%>%as.data.frame()
absent.df<-rbind.fill(absent.df,combine.df)


}

#prep absent.df dataframe
#remove NAs in 'absent' column - these are homologs not absent in any of the child taxa.
absent.df<-na.omit(absent.df)
names(absent.df)<-c("homology_id","absent")

#Create table of scaffold - gene - homology id - taxa homolog is absent in
absence.results<-left_join(master.table,absent.df,by="homology_id")%>%select(.,species_scaffold,gene,homology_id,absent)%>%na.omit(absent)%>%group_by(homology_id)

write.csv(absence.results,"absence.analysis.results.csv",quote=FALSE,row.names=FALSE)

#Add to master table.  
new.master.table<-left_join(new.master.table,absent.df)
```

### Measure scaffold occupancy  
The below code makes a histogram of scaffold occupancy per gene, and also finds the median, mean, minimum and maximum number of scaffolds occupied by a gene in a particular genome.  

**IMPORTANT NOTE**:  The below code runs on whatever contingency table was produced in the `{r cluster}` chunk.  

```{r contingency stats}
# The `Parse GFF3 and Agalma data` chunk and the `cluster` chunk must be run first before this chunk. *Whatever you have set as the contingency table there will be the input for this chunk.*  

#number of scaffolds
nrow(contingency)
#number of homologs shared across scaffolds
ncol(contingency)
#sums of each column
colsum<-colSums(contingency)
#histogram of number of sacffolds occupied per homolog
hist<-ggplot() + aes(colsum)+geom_histogram(fill="skyblue3",color="black")+xlab("Number of Scaffolds Occupied") + ylab("Count")+ggtitle("Number of Scaffolds Occupied Per Homolog (5taxa)")

#median number of scaffolds occupied
median(colsum)
#average number of sacffolds occupied
mean(colsum)
#min number of scaffolds occupied
min(colsum)
#max number of scaffolds occupied.
max(colsum)
```

### Test the Null Hypothesis  

Create a random contingency matrix with the same dimensions and proportion of 1s and 0s as from your real data. Cluster this random matrix using cross-clustering, agnes, diana, and kmeans.  

**IMPORTANT NOTE**: The below code bases the random matrix on whatever contingency table was produced in the `{r cluster}` chunk.  

```{r test null model}

#mixing up the columns or rows will not affect overall clustering pattern (i.e. all clustering)
#make a data frame with the same proportion of 1's and 0's in random order
dim(contingency)
n_entries<-ncol(contingency)*nrow(contingency)
proportion_1s<-sum(contingency)/n_entries
proportion_0s<-1-proportion_1s

random.matrix<-c(rep("1",proportion_1s*n_entries),rep("0",proportion_0s*n_entries))%>%sample(.,n_entries)%>%matrix(.,ncol=ncol(contingency),nrow=nrow(contingency))%>%as.data.frame()
rownames(random.matrix)<-row.names(contingency)
colnames(random.matrix)<-colnames(contingency)

#distance matrix
rand.dist.matrix<-daisy(random.matrix,metric="gower",stand=FALSE)

#cross-clustering
rand.cross.clust<-cc_crossclustering(rand.dist.matrix,out=FALSE)
rand.clust<-cc_get_cluster(rand.cross.clust)
rand.tsne_obj<-Rtsne(rand.dist.matrix,is_distance=TRUE,perplexity=1)
rand.tsne_data<-rand.tsne_obj$Y%>%data.frame()%>%setNames(c("X","Y"))%>%mutate(rand.cluster=factor(rand.clust))
rand.tsne_plot<-ggplot(aes(x = X, y = Y), data = rand.tsne_data) + geom_point(aes(color = rand.cluster))
rand.tsne_plot

#agglomerative
rand.ag<-agnes(rand.dist.matrix,diss=TRUE)
plot(rand.ag,labels=FALSE)

#divisive
rand.di<-diana(rand.dist.matrix)
plot(rand.di,labels=FALSE)

#kmeans
rand.dist.matrix.coerced<-as.matrix(rand.dist.matrix)
fviz_nbclust(rand.dist.matrix.coerced,kmeans,method="wss")
rand.kmean<-kmeans(rand.dist.matrix,2)
rand.kclust<-rand.kmean$cluster
rand.tsne_obj<-Rtsne(rand.dist.matrix,is_distance=TRUE,perplexity=1)
rand.tsne_data<-rand.tsne_obj$Y%>%data.frame()%>%setNames(c("X","Y"))%>%mutate(rand.cluster=factor(rand.kclust))
rand.tsne_plot<-ggplot(aes(x = X, y = Y), data = rand.tsne_data) + geom_point(aes(color = rand.cluster))
rand.tsne_plot

```

# References  
Ramos, O., Barker, D., and Ferrier, D. 2012. Ghost Loci Imply Hox and ParaHox Existence in the Last Common Ancestor of Animals. Current Biology, 22(20):1951-1956.  

Tellaroli, P., Bazzi, M., Donato, M., Brazzale, C., and Draghici, S. 2016. Cross-Clustering: A Partial Clustering Algorithm with Automatic Estimation of the Number of Clusters. PLOS ONE, 11(3): e0152333, https://doi.org/10.1371/journal.pone.0152333.  