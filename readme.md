# Drift Effects

The computer program NeBias.cpp estimates the degree to which a residual egg/seed bank influences the apparent magnitude of genetic drift from generation to generation. It is assumed that each diapausing embryo produced in a particular year has probability of hatching (phatch) in each subsequent year, with (1 – phatch) surviving to a subsequent year. Consecutive years of samples are drawn from the pool of hatchlings, and then used to estimate the Ne required to account for the observed allele-frequency change in adjacent years. To look at the pure effects of drift, it is assumed that sampling variance on the part of the investigator is absent. Many independent runs of this process are performed starting from a particular allele frequency and allowing sufficient numbers of generations to proceed so that essentially all eggs laid in a particular generation have hatched before statistics are accumulated (i.e., approximating a steady-state process).

Underlying details of the population-genetic model can be found at the top of the code.

On lines 24-33 of the code, the user enters the actual effective population size, the probability of annual hatching, the number of independent runs to make, and the number of runs in between printouts of the cumulative statistics to the slurm files.

Lines 70, 76, and 143 can be edited to name the output files.

The code is written so that runs are made in parallel for 9 different starting allele frequencies, which are entered on lines 148-156. If it is desired to alter the number of runs, edit line 75; and if  >19 are to be run, the allele-frequency array on lines 148-156 needs to be expanded.

There will be two sets of output files: “slurm” files containing the cumulative statistics allow the user to determine whether the runs have proceeded for sufficient time to equilibrate; “dataout” files give the final results for each of the 9 allele frequencies. 


To run the program, enter on the unix command line:

module load intel/2019.4
icc -o NeBias NeBias.cpp -lm -lgsl
sbatch –array=1-9 NeBias.sh

(The first line may need to be modified, depending on the system involved. The 9 in the final line needs to be edited if a different number of runs is being made).

A batch shell file (NeBias.sh) must be provided in the local file space to set up the series of runs:

#! /bin/bash

#SBATCH -A mlynch11
#SBATCH -p cmecpu1
#SBATCH -q cmeqos
#SBATCH -n 1
#SBATCH -t 11-4:00

echo "Running the script in parallel"
./NeBias $SLURM_ARRAY_TASK_ID


Here, the #SBATCH lines will need to be modified to the user’s specifications, NeBias is the folder within which the .pp and .sh files sit.

If all is operating properly, upon submission of the job, the slurm and dataout files should immediately appear, and the slurm files will begin to be periodically updated with the cumulative statistics, with the dataout files becoming populated after each run completes. The completion times will vary depending on the user-defined run lengths.  


Summary file of results:

Upon completion of all runs, an sbatch file called concatenate.sh (for example; lines below) can be submitted, which will create an output called summary.txt that contains the comma-delimited stacked set of final results, which can be imported to a spreadsheet. “dataoutxxx” needs to be edited to give the prefix of the output file names, which will come out in parallel as dataoutxxx_1.txt to dataoutxxx_9.txt.

To run this file, type on the command line: sbatch concatenate.sh


#! /bin/bash

#SBATCH -A mlynch11

#SBATCH -n 1
#SBATCH -t 0-4:00

myFiles1=`ls dataoutxxx_?.txt`
myFiles2=`ls dataoutxxx_??.txt`
cat $myFiles1 $myFiles2 > summary.txt

 
