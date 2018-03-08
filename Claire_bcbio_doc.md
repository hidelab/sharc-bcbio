Claire's Example

73 samples from Illumina HiSeq 2000 experiment

Downloaded from NCBI using sra-toolkit

1) Download appropriate sra-toolkit file from https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/
2) Before you download any sra files, you have to configure the download location. Navigate to the sratoolkit folder and run /bin/vdb-config -i
3) This brings up a slightly clunky user interface where you can set the download location ("Default Import Path"). Use the directional keys to highlight this box, or press 5. Then either navigate through the directories in the "directories:" box or left-click "Goto" to input your own path name. Press OK and then press 6, Enter, and 7 to save and quit.
4) Navigate to folder you have just set to download files to and write small script to run sra-toolkit. 

```
#!/bin/bash
#$ -l rmem=4G -l mem=4G
#$ -pe openmp 1
#$ -j y
#$ -l h_rt=100:00:00
#$ -m bea
#$ -M youremailaddress@sheffield.ac.uk

path/to/sratoolkit.2.8.2-1-centos_linux64/bin/prefetch.2.8.2 thing.sra thing2.sra thingn.sra


path/to/sratoolkit.2.8.2-1-centos_linux64/bin/fastq-dump.2.8.2 --gzip thing.sra thing2.sra thingn.sra
```

The first line downloads the sra files from NCBI, and the second line converts these to fastq and zips them in one go. Obviously edit this to the correct file name for the version you are using.

NB It is particularly important to include "--gzip" if you will be using up a lot of space. Unzipped Fastq files can be very large and can fill you system's memory very quickly. These 73 sra files and 73 fastq.gz files took up 803GB of space in total.

5) Once you have your fastq files, you can start setting up your bcbio input. The easiest thing to start off with is the .csv file of the metadata. This can be created in anything that can save as .csv, I use Excel because it's easy and I'm a heathen.

The .csv file requires four columns of information. The headers must be as follows "samplename", "description", "phenotype", "batch". Sample name can be the file name (the SRR code in this example). Description has to be different for every file, so I just put in the GSM number. Phenotype can be grouped terms (e.g. "patient" and "control" or "treated" and "untreated"). Batch is to signify the grouping of samples based on batch - e.g. if samples were run in 3 batches the value can be either 1, 2 or 3. In this example there are no batches known so all samples get a value of "1".

The next bit is important - YOU MUST SAVE YOUR FILE AS AN MS-DOS CSV OTHERWISE IT CAN'T BE READ PROPERLY AND YOU GET RANDOM SYMBOLS ALL OVER THE SHOP. 

NB - pick an analysis name to name all of your files. It will make it easier to keep track.

6) Now you have your csv file you need to put it into the right folder. I won't go into how to upload files into your HPC storage but you can find what you need here https://www.sheffield.ac.uk/wrgrid/using/access. 

Set up a folder for your project that ISN'T named the same as the analysis name you are using for your files. E.g. my Project Folder is called "PD_RNAseq" therefore will name my files "PD_bcbio.whatever". This is because bcbio will create a new folder with the same name as the file names later on, and it will get confusing if it's the same as the project folder. 

Put your csv file in your project folder.

7) Not many words rhyme with camel. EXCEPT FOR YAML! You need a .yaml template file to tell bcbio what to do. There are two ways to get a yaml template - either download one or make your own. I decided to download it once and edit it for my preferences, and now I use that for everything.


type: nano myanalysisname-template.yaml (remember, name it the same as the csv)

and paste:

# Template for human RNA-seq using Illumina prepared samples
---
details:
  - analysis: RNA-seq
    genome_build: GRCh37
    algorithm:
      aligner: star
      strandedness: unstranded
upload:
  dir: ../final 

We removed adaptor trimming because it wasn't necessary and actually took a lot of time.

8) Now you need to set up your SGE file that will direct bcbio to the samples and what to do with them. 

staying in your project folder, type nano youranalysisname.SGE, and paste:

                               

#!/bin/bash
#$ -l rmem=40G -l mem=40G
#$ -pe openmp 8
#$ -j y
#$ -l h_rt=48:00:00
#$ -m bea
#$ -M cmgreen1@sheffield.ac.uk
#$ -P rse

work_dir='/path/to/your/project/folder'

#Seq.Reads file directories
r1_files=$work_dir/locationofsrafiles
#If you have paired reads remove hash
#r2_files=$work_dir/locationofsrafiles

#Read in seq reads
r1=($(find $r1_files -type f -name "*.gz"|sort -n))
#r2=($(find $r2_files -type f -name "*.gz"|sort -n))


#Initialise the main analysis
echo "INITIALISING ANALYSIS"
/usr/local/community/bcbio-nextgen/2017-08/tool/bin/bcbio_nextgen.py -w template $work_dir/youranalysisname-template.yaml $work_dir/youranalysisname.csv ${r1[@]} 

#add ${r2[@]} to the end of the above line if paired 

#Perform the analysis
echo "PERFOMING ANALYSIS"
cd $work_dir/youranalysisname/work
/usr/local/community/bcbio-nextgen/2017-08/tool/bin/bcbio_nextgen.py -n 8 ../config/youranalysisname.yaml

9) When you have "qsub"ed this file it will do a number of things. First it will make another yaml file which is a combination of the yaml template and the csv file - essentially saying "do template to every file". This is also when it creates a new folder in your project folder called "youranalysisname" where it will create a config, work, and final folder. Config holds your yamls and csv, work is where it does the processing, and final is where it spits out the output files