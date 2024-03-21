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
![F1screenshot.png](/MyGenome/F1Screenshot.png).
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
