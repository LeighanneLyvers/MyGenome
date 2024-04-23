# MyGenome
Analyses for ABT480/CS485G genome assembly

## 1. Analysis of sequence quality
The F1 and R1 sequence datasets were analysed using FASTQC: 
```bash
ssh -Y ukyID@ukyID.cs.uky.edu
cd MyGenome
fastqc &
```
Load F1 and R1 datasets into GUI interface.
Take screen shots of output files:
![F1screenshot.png](F1Screenshot).
![F2screenshot.png](/data/F2screenshot.png).

## 2. Trimmming of sequences
The sequences were then trimmed utilizing Trimmomatic:
```bash
java -jar ~/assembly/trimmomatic-0.38.jar PE -threads 2 -phred33 -trimlog UFVPY202_errorlog.txt UFVPY202_1.fq.gz UFVPY202_2.fq.gz UFVPY202_1_paired.fastq UFVPY202_1_unpaired.fastq UFVPY202_2_paired.fastq UFVPY202_2_unpaired.fastq ILLUMINACLIP:adaptors.fasta:2:30:10 SLIDINGWINDOW:20:20 MINLEN:120
```
These were then analysed utilizing FASTQC:
```bash
fastqc UFVPY202_1_paired.fastq UFVPY202_1_unpaired.fastq UFVPY202_2_paired.fastq UFVPY202_unpaired.fastq
```
Take screen shots of the output files:
![F1Paired.png](/F1Paired.png).
![F2Paired.png](/F2Paired.png).

## 3. Counting the number of paired reads and the total number of bases
From the paired reads we then can then count the number of paired reads and the total number of bases. 
To count the number of paired reads:
```bash
grep -c '@A00261:902' UFVPY202_1_paired.fastq
```
To count the total number of bases:
```bash
cat UFVPY202_1_paired.fastq UFVPY202_2_paired.fastq | grep -v "+"| grep -e "F" -e "#" -e ":" -e "," | wc -c
```

## 4.Assembly of MyGenome
Upload the forward and reverse trimmed paired files to the Morgan Compute Cluster
  
Copy the VelvetOptimiser script to your personal directory and add email using nano
```bash
cp ../SLURM_SCRIPTs/velvetoptimiser_noclean.sh .
nano velvetoptimiser_noclean.sh
```
Submit assemblies to the SLURM queue
```bash
sbatch velvetoptimiser_noclean.sh <genomeID> <start_kmer> <end_kmer> <step_size>
```
  For this SLURM queue the start k-mer was 61, the end k-mer was 131, and the step size was 10.
  
Inspect assembly file
```bash
nano /path to log file
```
  Upon inspection the range of kmer values to use for the next VelvetOptimiser run was determined to be 91 to 111.

Re-run VelvetOptimiser witha  narrower k-mer range and step size of 2
```bash
sbatch velvetoptimiser_noclean.sh <genomeID> <start_kmer> <end_kmer> <step_size>
```
 For this SLURM queue the start k-mer was 91, the end k-mer was 111, and the step size was 2.
 
Rename optimized assembly according to the format GenomeID.fasta
```bash
mv contigs.fa genomeID.fasta
```

Use SimpleFastaHeaders.pl perl script to rename sequence headers to a standard format
```bash
//copy script to current directory
cp ../SCRIPTs/SimpleFastaHeaders.pl genomeID.fasta
//add email to script
nano SimpleFastaHeaders.pl
//run script
perl /path to SimpleFastaHeaders.pl /path to genomeID.fasta genomeID
```

## 5. Check genome completeness using BUSCO
Within the MCC supercomputer, copy the BuscoSingularity.sh script to your working directory
```bash
cd //path to working directory
cp ../SLURM_SCRIPTS/BuscoSingularity.sh .
```

Run BUSCO
```bash
sbatch /path to BuscoSingularity.sh genomeID.fasta
```
Inspect the BUSCO log
```bash
cat busco_logID.log
```

## 6. BLASTing MyGenome
Copy the CullShortContigs.pl script from the MCC to VM
```bash
scp ukyID@mcc.uky.edu:../SCRIPTs/CullShortContigs.pl .
```
Remove any contigs in MyGenome assembly that are less than 200 bp in length
```bash
perl CullShortContigs.pl ~/MyGenome/genomeID_nh.fasta
```
Check that it worked using the SequencesLength.pl
```bash
//copy SequencesLength.pl from MCC to VM
scp ukyID@mcc.uky.edu:..//SCRIPTs/SequencesLength.pl .
//run the script
perl SequenceLengths.pl ~/MyGenome/genomeID_final.fasta
```
Determine which contigs in the assembly correspond to the mitochondrial genome
  BLAST MoMitochondrion.fasta sequence against final genome assembly using output format 6 with specific column sections
```bash
//enter blast directory within VM
cd blast
//download MoMitochondrion.fasta
scp -r AppliedBioInfo@address:~/Desktop/MoMitochondrion.fasta .
//run the blast
blastn -query MoMitochondrion.fasta -subject MyGenome_nh.fasta -evalue 1e-50 -max_target_seqs 20000 -outfmt '6 qseqid sseqid slen length qstart qend sstart send btop' > B71v2sh.MyGenome.BLAST
//export a list of contigs that mostly comprise the mitochondrial sequence
awk '$3/$4 > 0.9 {print $2 ",mitochondrion"}' B71v2sh.MyGenome.BLAST > MyGenome_mitochondrion.csv
```
[UFVPY202_mitochondrion.csv](https://github.com/LeighanneLyvers/MyGenome/blob/main/UFVPY202_mitochondrion.csv)


