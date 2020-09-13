# IBDcalculation
In this repository you can find the pipeline to calculate identity by descent (IBD) from paper: Genomic and chemical evidence for local adaptation in resistance to different herbivores in *Datura stramonium* by *De-la-Cruz et al. 2020* 

   ddRad-seq data to calculate IBD is available in GenBank under the Bioproject "comming soon"

## Programs
   1. Lep-MAP3
   2. Samtools
   3. BWA
   4. awk
   
### The first step after trimming bad sequences is align all the individuals to reference genomes. This can be achieved using BWA program

1.- Create a folder and stored all the sequences (PE) of each individual

2.- Is important to note that for hundreds of PE reads the name of the files should be like teo1_1.fq and teo1_2.fq. This script considers to eliminate the column of the PE direction. At the end you will have all the aligned sequences per each individual in a .bam file. Also the original name of the individuals files are mantained

    for i in *.fq; do yolo=$(echo "$i" | rev | cut -c 6- | rev | uniq); bwa mem index_tic_geno ${yolo}_1.fq ${yolo}_2.fq -t 20 | samtools view -Sb -@ 20 - > ${yolo}.bam; done &> log.txt &
    
 ### Then sort all the bam files
 
    for f in *.bam; do samtools sort -@ 30 $f > ${f%\.*}_sorted.bam; done &
    
## Calculating Identity by Descent values between all individuals and each parent. 

### The sequencing data processing pipeline for Lep-MAP3 is based on "SAMtools mpileup" and custom scripts. This scripts can be found in https://sourceforge.net/p/lep-map3/wiki/LM3%20Home/#ibd YOU CAN ALSO FIND DOCUMENTATION TO INSTALL Lep-MAP3 there

pileupParser2.awk and pileup2posterior.awk, provided in scripts.zip (also provided in this tutorial). The usage is

All the stored -bam files for each individual are de imput to obtain GENOTYPE POSTERIOR PROBABILITIES 

    samtools mpileup -q 10 -Q 10 -s $(cat sorted_bams)|awk -f pileupParser2.awk|awk -f pileup2posterior.awk|gzip >post.gz

 This command requires two files, sorted_bams and mapping.txt, both containing exactly one line listing the file names for sorted bams and individual names, respectively and in the same order. If the data of each individual is in its own bam, then the files can be same (but it is more clear to remove the bam suffix from the individual names). Please note that this pipeline does not work with the old version of samtools (0.X). An example is provided in this tutorial. 

For example (3 individuals in 4 bams):

sorted_bams: 1.bam 1a.bam 2.bam 3.bam

mapping.txt: 1 1 2 3

 ### The next step is calculate the IBD values between all pair-wise individuals. Input from above script
  
    zcat post_from_pipeline_above.gz|awk '{if (NR==1) {print;print;print;print;print}; print}'|java IBD data=- >ibd.txt

 ### One program has finished jus use this command line to obtain relatedness to a potential parent
 
    sort -n -r -k 3,3  ibd.txt|grep -w possible_parent|less

That's all and Enjoy

imda@ecologia.unam.mx
