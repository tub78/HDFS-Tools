#############################################################################
Command-line utilities for interacting with a remote Hadoop filesystem (HDFS)
#############################################################################

The following HDFS tools are found in $(ICB_HADOOP_HOME)/scripts/all 

    hdfs.sh   - Toggles state of HDFS tunnel
    hls.sh    - Subtree listing from remote HDFS
    hget.sh   - Get files or subtree from HDFS

With appropriate environment variables and network connectivity, the HDFS
command toggles a ssh tunnel required to communicate with the remote HDFS.

HLS and HGET are used to manage a local copy of the remote HDFS.  The user 
first selects a location to hold the "root directory" of the local copy, and 
sets the HDFSPREFIX environment variable to this location.  Paths to files in
the local copy are the same as in the remote HDFS, except with the prefix
added.  This enables the use relative paths, which are converted for the
remote HDFS appropriately.  Calls to HLS and HGET only work if the current
working directory is located under the local root.  

Type <CMD> -h for specific usage information.  

-------------------------------
Setting up HDFS tools:  See below for setup examples.
-------------------------------

1. Install (i.e. unzip) hadoop package into HADOOP_HOME

2. Set environment variables in .bash_profile, e.g:

    JAVA_HOME=/System/Library/Frameworks/JavaVM.framework/Versions/1.6.0/Home
    HADOOP_HOME=~/Research/Hadoop/hadoop
    HDFSPREFIX=~/Data
    ICBUSER=sta2013

3. Add the following to your PATH

    $(ICB_HADOOP_HOME)/scripts/all 
    $(HADOOP_HOME)/hadoop/bin

4. Download & install ICB client configuration files under $(HADOOP_HOME)/hadoop/conf/ 

http://icb.med.cornell.edu/wiki/uploads/4/41/Hadoop-client-conf.zip

5. Set ICBUSER name in hadoop.job.ugi field in $(HADOOP_HOME)/hadoop/conf/core-site.xml 

6. Set JAVA_HOME in $(HADOOP_HOME)/hadoop/conf/hadoop-env.sh

7. Verify tunnel creating using these commands:

ssh -ND 2600 ICBUSER@rodin.med.cornell.edu
ps -eo user,pid,command |grep ssh

 
 TODO: Figure out how to support wildcards etc.
 
   Currently, wildcards cause a problem when passing 
   arguments to "hadoop fs" commands
 

-------------------------------
Examples (assuming HDFSPREFIX=~/Data).
-------------------------------
mac133990 ~ -> 
mac133990 ~ -> hdfs -c
    ENABLED:  0
    RUNNING PROCESS: 


mac133990 ~ -> 
mac133990 ~ -> hdfs -t
    ENABLED:  0
    PID: 
    + ssh -ND 2600 sta2013@rodin.med.cornell.edu
    Started HDFS tunnel with PID: '7647'


mac133990 ~ -> 
mac133990 ~ -> hdfs -c
    ENABLED:  1
    RUNNING PROCESS:  7647 ssh -ND 2600 sta2013@rodin.med.cornell.edu


mac133990 ~ -> 
mac133990 ~ -> hls
    hls.sh > CWD must be under HDFSPREFIX=~/Data


mac133990 ~ -> 
mac133990 ~ -> cd ~/Data/
mac133990 ~/Data ->


mac133990 ~/Data ->
mac133990 ~/Data -> hls
    Found 2 items
    drwxr-xr-x   - hadoop supergroup          0 2009-07-10 13:22 /scratchLocal
    drwxrwxrwx   - hadoop supergroup          0 2009-08-03 14:13 /user


mac133990 ~/Data ->
mac133990 ~/Data -> hls -v user/maqc3/input-data/reads-maqc3/Illumina/
    HDFSPREFIX=~/Data
    PWD=~/Data
    OVERLAP=6
    HDFSPWD=
    HDFSURL=/user/maqc3/input-data/reads-maqc3/Illumina/
    LOCALURL=~/Data/user/maqc3/input-data/reads-maqc3/Illumina/
    Found 4 items
    drwxr-xr-x   - maqc3 supergroup          0 2009-06-04 22:31 /user/maqc3/input-data/reads-maqc3/Illumina/100bp
    drwxr-xr-x   - maqc3 supergroup          0 2009-06-22 17:18 /user/maqc3/input-data/reads-maqc3/Illumina/Brain
    drwxr-xr-x   - maqc3 supergroup          0 2009-07-13 19:37 /user/maqc3/input-data/reads-maqc3/Illumina/NorthShore
    drwxr-xr-x   - maqc3 supergroup          0 2009-07-06 11:43 /user/maqc3/input-data/reads-maqc3/Illumina/uhr35


mac133990 ~/Data ->
mac133990 ~/Data -> cd user/maqc3/input-data/reads-maqc3/Illumina/Brain
    -bash: cd: user/maqc3/input-data/reads-maqc3/Illumina/Brain: No such file or directory


mac133990 ~/Data ->
mac133990 ~/Data -> hls user/maqc3/input-data/reads-maqc3/Illumina/Brain/
    Found 16 items
    -rw-r--r--   3 maqc3 supergroup   83230056 2009-06-22 17:15 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_1_sequence.compact-reads
    -rw-r--r--   3 maqc3 supergroup  198350968 2009-06-22 17:15 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_1_sequence.fq.gz
    -rw-r--r--   3 maqc3 supergroup       1210 2009-06-22 17:15 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_1_sequence_short.compact-reads
    -rw-r--r--   3 maqc3 supergroup       9175 2009-06-22 17:15 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_1_sequence_short.fq
    -rw-r--r--   3 maqc3 supergroup  114895300 2009-06-22 17:15 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_2_sequence.compact-reads
    -rw-r--r--   3 maqc3 supergroup  255804498 2009-06-22 17:15 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_2_sequence.fq.gz
    -rw-r--r--   3 maqc3 supergroup  112672171 2009-06-22 17:16 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_3_sequence.compact-reads
    -rw-r--r--   3 maqc3 supergroup  259357908 2009-06-22 17:16 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_3_sequence.fq.gz
    -rw-r--r--   3 maqc3 supergroup  122025101 2009-06-22 17:16 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_4_sequence.compact-reads
    -rw-r--r--   3 maqc3 supergroup  266334414 2009-06-22 17:16 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_4_sequence.fq.gz
    -rw-r--r--   3 maqc3 supergroup  118826217 2009-06-22 17:17 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_6_sequence.compact-reads
    -rw-r--r--   3 maqc3 supergroup  272331720 2009-06-22 17:17 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_6_sequence.fq.gz
    -rw-r--r--   3 maqc3 supergroup  117544429 2009-06-22 17:17 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_7_sequence.compact-reads
    -rw-r--r--   3 maqc3 supergroup  262340995 2009-06-22 17:17 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_7_sequence.fq.gz
    -rw-r--r--   3 maqc3 supergroup  100760514 2009-06-22 17:18 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_8_sequence.compact-reads
    -rw-r--r--   3 maqc3 supergroup  233632305 2009-06-22 17:18 /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_8_sequence.fq.gz


mac133990 ~/Data -> 
mac133990 ~/Data -> hget user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_1_sequence.compact-reads
    hget.sh > Local directory does not exist, or is not writable: ~/Data/user/maqc3/input-data/reads-maqc3/Illumina/Brain
    hget.sh > Try -p flag


mac133990 ~/Data -> 
mac133990 ~/Data -> hget -pv user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_1_sequence.compact-reads
    HDFSPREFIX=~/Data
    PWD=~/Data
    OVERLAP=6
    HDFSPWD=
    HDFSURL=/user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_1_sequence.compact-reads
    LOCALURL=~/Data/user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_1_sequence.compact-reads
    ISFILE=1
    ISDIR=0
    LOCALDIR=~/Data/user/maqc3/input-data/reads-maqc3/Illumina/Brain
    CMD=~/Research/Hadoop/hadoop/bin/hadoop fs -get /user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_1_sequence.compact-reads ~/Data/user/maqc3/input-data/reads-maqc3/Illumina/Brain/s_1_sequence.compact-reads
    ...


mac133990 ~/Data -> 
mac133990 ~/Data -> ls user/maqc3/input-data/reads-maqc3/Illumina/Brain/
    s_1_sequence.compact-reads


mac133990 ~/Data -> 
mac133990 ~/Data -> hls -r user/maqc3/input-data/reads-maqc3/Illumina/100bp
    drwxr-xr-x   - maqc3 supergroup          0 2009-06-04 22:31 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split
    -rw-r--r--   3 maqc3 supergroup  325464704 2009-06-04 22:29 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_Brain_100bp_MAR2009_fastq-1.compact-reads
    -rw-r--r--   3 maqc3 supergroup  323904091 2009-06-04 22:29 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_Brain_100bp_MAR2009_fastq-2.compact-reads
    -rw-r--r--   3 maqc3 supergroup  324001625 2009-06-04 22:30 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_Brain_100bp_MAR2009_fastq-3.compact-reads
    -rw-r--r--   3 maqc3 supergroup  324651749 2009-06-04 22:30 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_Brain_100bp_MAR2009_fastq-4.compact-reads
    -rw-r--r--   3 maqc3 supergroup  324276468 2009-06-04 22:30 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_Brain_100bp_MAR2009_fastq-5.compact-reads
    -rw-r--r--   3 maqc3 supergroup  278561961 2009-06-04 22:30 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_Brain_100bp_MAR2009_fastq-6.compact-reads
    -rw-r--r--   3 maqc3 supergroup  336044333 2009-06-04 22:30 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_UHR_100bp_MAR2009_fastq-1.compact-reads
    -rw-r--r--   3 maqc3 supergroup  335763176 2009-06-04 22:30 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_UHR_100bp_MAR2009_fastq-2.compact-reads
    -rw-r--r--   3 maqc3 supergroup  335438175 2009-06-04 22:30 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_UHR_100bp_MAR2009_fastq-3.compact-reads
    -rw-r--r--   3 maqc3 supergroup  335688144 2009-06-04 22:31 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_UHR_100bp_MAR2009_fastq-4.compact-reads
    -rw-r--r--   3 maqc3 supergroup  335992001 2009-06-04 22:31 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_UHR_100bp_MAR2009_fastq-5.compact-reads
    -rw-r--r--   3 maqc3 supergroup  335541696 2009-06-04 22:31 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_UHR_100bp_MAR2009_fastq-6.compact-reads
    -rw-r--r--   3 maqc3 supergroup  336026291 2009-06-04 22:31 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_UHR_100bp_MAR2009_fastq-7.compact-reads
    -rw-r--r--   3 maqc3 supergroup    3523258 2009-06-04 22:31 /user/maqc3/input-data/reads-maqc3/Illumina/100bp/compact-split/SEQC_Inhouse_ILM_UHR_100bp_MAR2009_fastq-8.compact-reads


mac133990 ~/Data -> 
mac133990 ~/Data -> hdfs
    ENABLED:  1
    PID:  7647
    Stopping HDFS tunnel with PID: '7647'
    + kill -9 7647


