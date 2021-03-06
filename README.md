
### NGS-Read-Simulator-and-Mapping ####

Genome file Source: ftp://ftp.ncbi.nih.gov/genomes/Homo_sapiens/CHR_01/ (Download hs_ref_GRCh38.p2_chr1.fa.gz)

Fastq Quality Source: http://www.somewhereville.com/?p=1508 (Used Scores from 4th Column: Sanger "Q + 33" Shift)

    Used subset quality values by selecting them from pre-existing fastq file.

    Code and all the neccessary files are saved in the repository.

Python Code for generating FastQ file:
!/usr/bin/python3

author = "Rajesh Shinde" credits = ["Rajesh Shinde", "Aditya Pathak"] version = "1.0" email = "rajesh27071992@gmail.com" status = "Development"
Imports

import random
File Read - Writes

try: fh = open("hs_ref_GRCh38.p2_chr1.fa", "r") except FileNotFoundError: print("Sequence file is not present in the working directory!")

try: fh2 = open("qual_values_2","r") except FileNotFoundError: print("Dummy quality value file is not present in the working directory!")

try: fout = open("RandomSeq.fastq","w") except FileExistsError: print("Simulated Fastq file is already present!")

Thanks..
Global Variables

seq_string = '' qual_string = '' No_of_seq = 100000 No_of_iterations = No_of_seq/2 read_length = 50
## File data handlers

for line in fh: if (line.startswith(">")): continue else: seq_string = seq_string + line.strip()

for l in fh2: qual_string = qual_string + l.strip()
### Functions

''' This function introduces 1 base error in the read provided as input '''

def IntroduceError(seq1): list1 = ["A","G","T","C","N"] random_pos = random.randint(0,len(seq1)-1) new_nucleotide = random.choice(list1)

if (new_nucleotide != seq1[random_pos]):
    seq1 = seq1[:random_pos] + new_nucleotide + seq1[random_pos + 1:]
return seq1

''' This function select 2 random reads of 50bp from Genome and input one of the two read to the IntroduceError function. This is because the required error rate is 0.01 and hence, only one of 2 reads can have base with error '''

def random_seq(sequence): pos1 = random.randint(0,(len(sequence)-read_length)) return_seq1 = sequence[pos1:pos1+read_length] pos2 = random.randint(0,(len(sequence)-read_length)) return_seq2 = sequence[pos2:pos2+read_length]

read_with_error = random.randint(1,2)
if (read_with_error == 1):
    return_seq1 = IntroduceError(return_seq1)
else:
    return_seq2 = IntroduceError(return_seq2)
return (return_seq1,return_seq2)

''' This function return 50 randomly selected quality values qualities string ''' def random_qual(num, qualities): return_qual= '' for x in range(num):
return_qual = return_qual + random.choice(qualities) return return_qual

j = 1 ## Variable to append read_number at the end of each read

''' This function writes 2 reads at a time to the output fastq file. One of these reads has a base with error '''

def NGSFastqSimulator(sequence1, qualities): global j for i in range(int(No_of_iterations)): (read_sequence1 , read_sequence2) = random_seq(sequence1)

    ### Writing 1st Read ###
    fout.write("@RandomSequence_"+str(j)+"\n")
    fout.write(read_sequence1 + "\n")
    fout.write("+"+"\n")
    fout.write(random_qual(read_length,qual_string)+"\n")

    ### Writing 2nd Read ######
    fout.write("@RandomSequence_"+str(j+1)+"\n")
    fout.write(read_sequence2 + "\n")
    fout.write("+"+"\n")
    fout.write(random_qual(read_length,qual_string)+"\n")
    j = j+2

###### Function call

NGSFastqSimulator(seq_string, qual_string)
Message to print on successful generation of file

print("File generation completed")
Fastqc of Generated FastQ file:

Filename RandomSeq.fastq File type Conventional base calls Encoding Sanger / Illumina 1.9 Total Sequences 99956 Sequences flagged as poor quality 0 Sequence length 50 %GC 41
Alignemt Steps:

BWA-SW is used for alignemt

    2 different aligents performed with different options

Alignement 1: -n (Maximum Edit Distance): 0.01 Command used: Indexing Genome: bwa index -a bwtsw hs_ref_GRCh38.p2_chr1.fa Alignement: bwa aln -n 0.01 hs_ref_GRCh38.p2_chr1.fa RandomSeq.fastq > without\ -N\ option/alignment.sai SAM generation: bwa samse hs_ref_GRCh38.p2_chr1.fa with_\ -N\ option/alignment.sai RandomSeq.fastq > with_\ -N\ option/alignment.sai.sam SAM to BAM: samtools view -bS without\ -N\ option/alignment.sai.sam > without\ -N\ option/alignment.sai.bam Reads with one mismatch in alignment: 39602 Reads with 0 mismatches: 60253

Alignement 2: -n (Maximum Edit Distance): 0.01 -N : after disabling iterative search

Alignment: bwa aln -n 0.01 -N hs_ref_GRCh38.p2_chr1.fa RandomSeq.fastq > with_\ -N\ option/alignment.sai SAM generation: bwa samse hs_ref_GRCh38.p2_chr1.fa with_\ -N\ option/alignment.sai RandomSeq.fastq > with_\ -N\ option/alignment.sai.sam SAM to BAM: samtools view -Sb with_\ -N\ option/alignment.sai.sam > with_\ -N\ option/alignment.sai.bam Reads with one mismatch in alignment: samtools view with_\ -N\ option/alignment.sai.bam | grep -E "XM:i:(1)"| wc -l , result = 39602 Reads with 0 mismatches: samtools view with_\ -N\ option/alignment.sai.bam | grep -E "XM:i:(0)"| wc -l, result = 60253


- Error rate 0.01.


Thanks.

