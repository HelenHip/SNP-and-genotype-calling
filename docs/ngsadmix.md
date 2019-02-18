## Population structure with NGSadmix

[NGSadmix](http://www.popgen.dk/software/index.php/NgsAdmix) is a method for inferring admixture from genotype likelihoods, thus taking into account the uncertainty inherent to NGS data. NGSadmix is part of the package ANGSD, it uses genotype likelihood information rather than absolute genotype calls. 

We first need to conver the bcf file to beagle format with angsd:
```bash
mkdir ngsadmix
angsd -vcf-gl <(bcftools view -O v filtering/snps.bcf) -fai genome/Hmel2.fa.fai -doMaf 3 -nInd 32 -domajorminor 1 -doglf 2 -out ngsadmix/snps
```

Now prepare the script to run NGSadmix for K=2 to K=4 in parallel:

```bash
#!/bin/bash
#$ -l h_rt=1:00:00
#$ -l mem=2G
#$ -t 1-4
#$ -m bea
#$ -M myemail@mail.com
#$ -j y
#$ -o ngsadmix.log
#$ -N ngsadmix

# internal SGE task index
K=$(($SGE_TASK_ID+1))


hostname
date
echo "=============================================================================="

NGSadmix -likes ngsadmix/snps.beagle.gz -K $K -o ngsadmix/snps_$K >& ngsadmix/snps_$K.sclog

echo "=============================================================================="
date
```
When you have finished editing the bash script, save it as ngsadmix.sh, make it executable with chmod and submit it to the job queue with qsub:
```bash
chmod +x ngsadmix.sh
qsub ngsadmix.sh
```
After running, you should have the following files: 
```bash
ls -lh ngsadmix
```
The important ones here are the .qopt files that contains the admixture proportions:
```bash
head ngsadmix/snps_K2.qopt
```
should show something like:
>``0.00000000100000000000 0.99999999900000002828 ``<br>
>``0.08651065200327273663 0.91348934799672731888 ``<br>
>``0.00000000100000000000 0.99999999900000002828 ``<br>
>``0.00000000100000000000 0.99999999900000002828 ``<br>
>``0.49425155246735569259 0.50574844753264425190 ``<br>
>``0.48612707587065617787 0.51387292412934382213 ``<br>
>``0.99999999900000002828 0.00000000100000000000 ``<br>
>``0.99999999900000002828 0.00000000100000000000 ``<br>
>``0.00000000100000000000 0.99999999900000002828 ``<br>
>``0.00000000100000000000 0.99999999900000002828 ``<br>


Now we are going to visualize the admixture estimates using ´R´. We will need some more information about the samples to produce meaningful plots. A file with information about the race and the sex of each sample can be copied from a shared directory on ShARC:
```bash
cp /usr/local/extras/Genomics/workshops/NGS_AdvSta_2019/SNPgenocall/sample_race_sex.tsv ngsadmix/ 
head sample_race_sex.tsv
```
Now, let's launch an interactive session of ``R`` and generate some plots for the admixture inferences when K=2:

```R
# Load admixture proportions
admix<-t(as.matrix(read.table("ngsadmix/snps_K2.qopt")))

# Load info about samples
id.info<-read.table("ngsadmix/sample_race_sex.tsv", sep="\t", header=T)
# add category combining race and sex
id.info['race.sex']<-paste(id.info$race,id.info$sex,sep="-")

# Palette for plotting
mypal<-c("#E41A1C","#377EB8","#4DAF4A","#984EA3")

# sort by race and plot
admix<-admix[,order(id.info$race)]
id.info<-id.info[order(id.info$race),]
pdf(file="ngsadmix/snps_K2_byrace.pdf")
h<-barplot(admix,col=mypal,space=0,border=NA,xlab="Individuals",ylab="admixture")
text(tapply(1:nrow(id.info),id.info$race,mean),-0.05,unique(id.info$race),xpd=T)
dev.off()

# sort by sex
admix<-admix[,order(id.info$sex)]
id.info<-id.info[order(id.info$sex),]
pdf(file="ngsadmix/snps_K2_bysex.pdf")
h<-barplot(admix,col=mypal,space=0,border=NA,xlab="Individuals",ylab="admixture")
text(tapply(1:nrow(id.info),id.info$sex,mean),-0.05,unique(id.info$sex),xpd=T)
dev.off()

# sort by race and sex
admix<-admix[,order(id.info$race.sex)]
id.info<-id.info[order(id.info$race.sex),]
pdf(file="ngsadmix/snps_K2_byracebysex.pdf")
h<-barplot(admix,col=mypal,space=0,border=NA,xlab="Individuals",ylab="admixture")
text(tapply(1:nrow(id.info),id.info$race.sex,mean),-0.05,unique(id.info$race.sex),xpd=T)
dev.off()
```

This generates 3 pdf files that can be downloaded for visualization. Further analysis with K=3:

```R
# Load admixture proportions
admix<-t(as.matrix(read.table("ngsadmix/snps_K3.qopt")))
# sort by race and plot
admix<-admix[,order(id.info$race)]
id.info<-id.info[order(id.info$race),]
pdf(file="ngsadmix/snps_K3_byrace.pdf")
h<-barplot(admix,col=mypal,space=0,border=NA,xlab="Individuals",ylab="admixture")
text(tapply(1:nrow(id.info),id.info$race,mean),-0.05,unique(id.info$race),xpd=T)
dev.off()

# sort by sex
admix<-admix[,order(id.info$sex)]
id.info<-id.info[order(id.info$sex),]
pdf(file="ngsadmix/snps_K3_bysex.pdf")
h<-barplot(admix,col=mypal,space=0,border=NA,xlab="Individuals",ylab="admixture")
text(tapply(1:nrow(id.info),id.info$sex,mean),-0.05,unique(id.info$sex),xpd=T)
dev.off()

# sort by race and sex
admix<-admix[,order(id.info$race.sex)]
id.info<-id.info[order(id.info$race.sex),]
pdf(file="ngsadmix/snps_K3_byracebysex.pdf")
h<-barplot(admix,col=mypal,space=0,border=NA,xlab="Individuals",ylab="admixture")
text(tapply(1:nrow(id.info),id.info$race.sex,mean),-0.05,unique(id.info$race.sex),xpd=T)
dev.off()
```