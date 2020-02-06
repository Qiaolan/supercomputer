I am going write a short guide of ASC unity cluster and I hope this would be helpful to new users.

Basic application

First of all, what is ASC unity cluster?

The Unity cluster is a local high-performance computing (HPC) environment maintained by Arts and Sciences Technology Services (ASCTech). Unity is designed to accommodate researchers with their intense computational and storage needs. That is, if your program runs for hours or days on your own device, you should turn to the unity cluster.

How to get access to unity cluster? As a student of college of arts and sciences, you can send an request here. After you obtain the access, you can log in unity cluster via terminal in Mac.

Now, let's start to show how to run your R program in unity cluster. It would be very helpful if you've learnt some basic linux commands.

Open terminal and input

ssh -l deng.295 unity.asc.ohio-state.edu (here you should use your own lastname.#)

then you will be asked to input your password. By the way, your password is invisible.

When you see "Welcome to Unity", you have successfully logged in. Next, let's run R in unity cluster.

module load intel

module load R/3.5.1-test

R

You can see familiar environment as in your own laptop. However, actually we don't run R program here. The reason why we do this is that you can install required R packages by your program here. Otherwise, you will fail to run your program, because only basic R packages are installed in unity cluster and you have to install many extra packages.

Now, I am going to show how to run your R program step by step.

Step 1: upload rscript file.

scp ~/test.R deng.295@unity.asc.ohio-state.edu:/home/deng.295 

scp command will upload your rscript file to the home directory in unity cluster. By the way, you'd better add

#! /apps/R/intel/18.0/3.5.1/bin/Rscript 

at the head of your rscript. This line of code asks the system to use Rscript to run this file. Moreover, since you will not be able to see output when programs are running in unity cluster, you should always save your output or the whole workspace by write.csv() or save.img().

Step 2: upload or write pbs file

A Portable Batch System (PBS) file defines the commands and cluster resources used for the job. In unity cluster, you can create a new pbs file by

vi test.pbs 

and below is an example of pbs file:

#PBS -l walltime=08:00:00

#PBS -l nodes=1:ppn=16,mem=64GB

#PBS -N Example_job

#PBS -j oe

#PBS -m abe

#PBS -M supercomputer@gmail.com


module load intel

module load R/3.5.1-test

Rscript test.R

#PBS -l walltime=08:00:00, this defines the maximum running time you can use in cluster

#PBS -l nodes=1:ppn=16,mem=64GB, we will use 16 processors and each processor has 4GB memory.

#PBS -N Example_job, the name of your job

#PBS -j oe

#PBS -m abe

#PBS -M supercomputer@gmail.com, unity cluster will send message upon start and end of your job


After defining these parameters, we start to write commands to run our program.

module load intel, this is necessary to load R

module load R/3.5.1-test, load R

Rscript test.R, run R script

More information can be found here.

Step 3: submit your job

Save the pbs file and input:

qsub test.pbs

Then it will show your job ID which is the unique identification of your job.

It is common that your job will not start immediately and be in queue when lots of users are using it. To check the status of your job, input:

qstat -u lastname.# 

You can see if your job is running (R), in queue (Q), or complete (C).

showq -u lastname.# 

is similar and

showq

gives a full view of current status of unity cluster.

qdel all or qdel jobid

help you delete all or specific job.

Step 4: download outputs

After your job is finished, you should see outputs of the program. To download them, we still use scp but in reverse way:

scp lastname.#@unity.asc.ohio-state.edu:/home/lastname.#/example.csv ~/somefileinhomedirectory

Then the outputs will be downloaded to your local directory.



Now, you should know how to run your R script file in unity cluster.



Parallel Computing

The power of unity cluster is shown by parallel computing. For example, a large-scale simulation in R may take you several days to complete, while by parallel computing, this can be done in hours.

To initiate parallel computing, we need to add some commands to our pbs file. Here is an example:



#PBS -l walltime=24:00:00

#PBS -l nodes=1:ppn=16,mem=224GB

#PBS -N Z_AF

#PBS -j oe

#PBS -m abe

#PBS -M supercomputerdeng@gmail.com

#COMMANDS TO RUN

module load intel

module load R/3.5.1-test

module load cxx17

#There are multiple versions of R and do check whether the version of R is compatible with the code you are going to execute.

for i in `seq 1 16`

do

(./z_sim_code.R $i) &

done

wait
