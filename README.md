### SANGER AMPLICON PIPELINE

Author:         Tom van Wijk (NIOO-KNAW)<br/>
Contact:        t.vanwijk@nioo.knaw.nl / bioinformatics-support@nioo.knaw.nl

### DESCRIPTION

This pipeline processes, aligns and classifies paired- or single-end Sanger reads
to a Silva or UNITE reference database and creates a taxonomy file. Reference database
options for 16S, 18S, 23S, 28S and ITS amplicons are included.
The pipeline is developed to run on the internal NIOO computing servers.

### INSTALLATION

It is recommended to install the pipeline in your home directory: `/home/nioo/{username}`<br/>
This is done by following these steps:

-	Clone the sanger-amplicon-pipeline repository to your home directory by logging on to the server,<br/>
	navigating to your home directory if not already there and executing the command:<br/>
	`git clone https://gitlab.bioinf.nioo.knaw.nl/pipelines/sanger-amplicon-pipeline.git`

-	To be able to run the python scripts for any location it is recommended to add the location<br/>
	of the repository to your PATH variable:<br/>
	`PATH=$PATH:/home/nioo/{username}/sanger-amplicon-pipeline/` (replace {username} with your actual username)<br/>
	Note: The PATH variable resets after every new login.<br/>
	Therefore it is recommended to add the command to your `~/.bashrc file`.<br/>
	This way, the PATH varibale is automatically configurated when you log-in on the server.

To run the pipeline on the NIOO servers it is required to create a conda environment<br/>
that contains the required dependencies. You can do this by running the following commands:

`mamba create --name sanger-amplicon-environment`<br/>
`conda activate sanger-amplicon-environment`<br/>
`mamba install biopython fastqc python=3.9.4 multiqc pear rdptools`<br/>
`conda deactivate`<br/>

Note: It is recommended to only run one command at the time. Conda might require
input during multiple of these steps. Simply follow the instructions on the screen.

### UPDATE INSTALLATION

When you already have the sanger-amplicon-pipeline installed and want to check if there are updates<br/>
and/or install updates, login on one of the nioo servers, navigate to the repository:<br/>
`cd sanger-amplicon-pipeline`

and run the following command to check for updates:<br/>
`git pull`

### USAGE

The pipeline can be started with the following command:<br/>
`sanger-amplicon-pipeline -f {FORDIR} -r {REVDIR} -n {PROJNAME} -d {REFDB} -q {QTRIM} -t {DELTEMP} -x {NAMETRIM}`

Required parameters:
-	**{FORDIR}:**<br/>
Location of directory with .ab1 files containing forward reads (required)

Optional parameters: (can simply be left out when running the command)

-	**{REVDIR}:**<br/>
Location of directory with .ab1 files containing reverse reads.<br/>
Note: Both the forward and reverse .ab1 files need to have an identical flag and the end
of both filenames separated by an underscore for the software to be able to match the filepairs.
For example: `20E6SAB000_A10.ab1` and `20E6SAB001_A10.ab1`.<br/>
When left at default (parameter not defined), pipeline starts in single-end mode.

-	**{PROJNAME}:**<br/>
Name of the project that will be included in output directory name.<br/>
This parameter is optional but it is highly recommended to use this since it will help you organise
your data and identify the differect output directories.<br/>
When left empty, the output directory will be named with a timestamp.

-	**{REFDB}:**<br/>
Reference database to be used for taxonomic classification. Default = 'silva-ssu'.<br/>
Available reference database options are:<br/>
'silva-ssu': Silva SSU 16S/18S small subunit risosomal RNA database<br/>
'silva-lsu': Silva LSU 23S/28S large subunit ribosomal RNA database<br/>
'unite': UNITE ITS ribosomal RNA database

-	**{QTRIM}:**<br/>
Quality trimming of reads, set to 'no' to deactivate. Default = yes.<br/>
This parameter is optional, not recommended to change except for experienced users.

-	**{DELTEMP}:**<br/>
Delete temporary output files, set to 'no' to deactivate. Default = 'yes'.<br/>

-	**{NAMETRIM}:**<br/>
Trim sequence names in MEGA output file. set to 'no' to deactivate. Default is 'yes'.<br/>
This might be usefull when using files with sequence headers that are deviant from the standard.
When on, everything before the first `_` and after the second `_` in the sequence headers will be trimmed off.
For example: `>21F0SAA000_A01_premix` will be trimmed to `>A01`.<br/>
Note: Only affects MEGA output file.<br/>
Note: The script that created the MEGA output file can be run separately.
Instructions on how to do so are shown by running:<br/>
`make-mega-format -h` or `make-mega-format --help`

Additional help on how to run the pipeline is shown by running:<br/>
`sanger-amplicon-pipeline -h` or `sanger-amplicon-pipeline --help`

Note: The pipeline automatically engages and closes the conda environment so it is not
required to engage the conda environment manually at any moment.

Note: It is recommended to run the pipeline in a TMUX or SCREEN session so it does not
terminate when your ssh client crashes, your connection to the server is broken or any
issues in that regard.

### OUTPUT

When you run the pipeline for the first time, it will create a directory in your home directory called<br/>
`/home/nioo/{username}/sanger-amplicon-pipeline-output/`.<br/>

In this directory, the output of each run you perform with the pipeline will
be stored in their own subdirectory. The name of that subdirectory will be a
combination of the name parameter given when running the pipeline and a randomly
generated id. In the output directory you find the following data:

-	**{random-ID}\_{PROJNAME}*\_sanger-amplicon-pipeline.log**:<br/>
*if provided<br/>
A logfile generated by the pipeline.

-	**{random-ID}\_{PROJNAME}*\_{classification-db}\_RDP-out.txt**:<br/>
*if provided<br/>
Tab-delimited output classification file from RPD classifier containing the taxonomic classification
off all reads in de input dataset.

-	**{random-ID}\_{PROJNAME}*\_{classification-db}\_RDP-hier.txt**:<br/>
*if provided<br/>
Hier output file from RDP classifier.

-	**{random-ID}\_{PROJNAME}*\_{classification-db}\_RDP-out\_MEGA-format.txt**:<br/>
*if provided<br/>
Additional output file that combines data from the tab-delimited output classification file from RPD classifier
and processed .fastq files into a format that can be directly imported to MEGA.

-	**{random-ID}\_{PROJNAME}*\_{classification-db}\_RDP-out\_make-mega-format.log**:<br/>
*if provided<br/>
A logfile generated by the make-mega-format subscript.

-	**quality_control_post-trim/fastQC/**:<br/>
A quality report of the individual .fastq files	that are generated by the pipeline after
quality trimming (if not deactivated) and conversion of .ab files to .fastq format.

-	**quality_control_post-trim/multiqc_report.html**:<br/>
An .html report that gives an overview of the quality of the full dataset.

-	**quality_control_post-trim/multiqc_data/**:<br/>
Data that the .html quality report file uses.

-	**temp/01_fastq_convert/**:<br/>
Converted and trimmed .fastq files of bothforward and reverse reads in their own directories.<br/>
Note: The temp directory is automatically removed when leaving parameter {DELTEMP} at default or 'yes'.

-	**temp/02_fastq_assembled/**:<br/>
.fastq files of the joined and unjoined reads after attempted assembly<br/>
Note: The temp directory is automatically removed when leaving parameter {DELTEMP} at default or 'yes'.<br/>
Note: This directory is only created in paired-end mode.

-	**temp/03_fastq_concatenated/**:<br/>
.fastq files where reads belonging to a	single sample are placed in one file again.<br/>
Note: The temp directory is automatically removed when leaving parameter {DELTEMP} at default or 'yes'.<br/>
Note: This directory is only created in paired-end mode.

-	**processed_fasta_files/**:<br/>
Fully processed input data converted to .fasta format.<br>
