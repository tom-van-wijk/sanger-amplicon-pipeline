#!/usr/bin/env python

# Name:         make-mega-format
# Author:       Tom van Wijk

# Consult the README.md in this git repository for detailed instructions
# on how to install and use this tool


# import python libraries
from argparse import ArgumentParser
import os
import sys
import logging
import random


# Function to parse the command-line arguments
# Returns a namespace with argument keys and values
def parse_arguments(args):
	parser = ArgumentParser(prog="make-mega-format")
	parser.add_argument("-f", "--fasta_dir", dest = "fasta_files",
		action = "store", default = None, type = str,
		help = """Location of directory with processed .fasta files created by the
		sanger amplicon pipeline. (required)""", required = True)
	parser.add_argument("-i", "--rdp_file", dest = "rdp_file",
		action = "store", default = None, type = str,
		help = """Location of '_RDP-out.txt' classification file created by the
		sanger amplicon pipeline. (required)""", required = True)
	parser.add_argument("-o", "--output_prefix", dest = "output_prefix",
		action = "store", default = None, type = str,
		help = """Location of output files. If left at Default, the output and log
		files will be written to directory in which the rdp file is located.
		NOTE: this parameter is intended to be used by the sanger pipeline script,
		it is not recommended to modify when just running make-mega-format.""")
	parser.add_argument("-x", "--name_trim", dest = "name_trim",
		action = "store", default = "yes", type = str,
		help = """Trim the sequence names in MEGA output file. Default is 'yes',
		set to 'no' to turn name trimming off. This might be usefull when using
		files with sequence headers that are deviant from the standard.
		When on, everything before the first '_' and after the second '_' in
		the sequence headers will be trimmed off. For example: '>21F0SAA000_A01_premix'
		will be changed to '>A01'.""")
	return parser.parse_args()


# Function creates logger with handlers for both logfile and console output
# Returns logger
def create_logger(logpath):
	# create logger
	log = logging.getLogger()
	log.setLevel(logging.INFO)
	# create file handler
	fh = logging.FileHandler(logpath)
	fh.setLevel(logging.DEBUG)
	fh.setFormatter(logging.Formatter('%(message)s'))
	log.addHandler(fh)
	# create console handler
	ch = logging.StreamHandler()
	ch.setLevel(logging.INFO)
	ch.setFormatter(logging.Formatter('%(message)s'))
	log.addHandler(ch)
	return log


# Function creates a list of files or directories in <inputdir>
# on the specified directory depth
def list_directory(input_dir, obj_type, depth):
	dir_depth = 1
	for root, dirs, files in os.walk(input_dir):
		if dir_depth == depth:
			if obj_type ==  'files':
				return files
			elif obj_type == 'dirs':
				return dirs
		dir_depth += 1


# Function closes logger handlers
def close_logger(log):
	for handler in log.handlers:
		handler.close()
		log.removeFilter(handler)


# function to iterate over all sequences in each file of a directory of .fasta files and returns
# a dictionary with the sequence headers as keys and corresponding sequences as values
def parse_fasta_files(fasta_dir, log):
	fasta_dict = {}
	merged_sequences = 0
	unmerged_sequence_pairs = 0
	for file in list_directory(fasta_dir, 'files', 1):
		path = fasta_dir+"/"+file
		#print("path: \t"+path)
		with open(path, "r") as fasta:
			header = ""
			sequence = ""
			for line in fasta:
				if line.startswith('>'):
					if header == "":
						merged_sequences += 1
						header=line.replace('\n','').replace('>','')
					else:
						#add header and sequence to dict
						fasta_dict[header] = sequence
						#print(header+" first of file with second sequence!")
						#print(sequence)
						unmerged_sequence_pairs += 1
						merged_sequences -= 1
						header=line.replace('\n','').replace('>','')
						sequence = ""
				elif line.startswith(('A','a','T','t','C','c','G','g')):
					sequence += line.replace('\n','')
			#add header and sequence to dict
			fasta_dict[header] = sequence
			#print(header)
			#print(sequence)
	log.info("\nMerged sequence pairs handled: \t"+str(merged_sequences))
	log.info("Unmerged sequence pairs handled: \t"+str(unmerged_sequence_pairs))
	return fasta_dict


# MAIN function
def main():
	# parse command line arguments
	args = parse_arguments(sys.argv)
	# generate run_id
	run_id = random.randint(1000000, 9999999)

	# determining output file location
	if args.output_prefix == None:
		outpath = os.path.dirname(os.path.abspath(args.rdp_file))+"/"+str(run_id)+"_MEGA-format.txt"
		logpath = os.path.dirname(os.path.abspath(args.rdp_file))+"/"+str(run_id)+"_make-mega-format.log"
	else:
		outpath = os.path.abspath(args.output_prefix)+"_MEGA-format.txt"
		logpath = os.path.abspath(args.output_prefix)+"_make-mega-format.log"

	# create logger
	log = create_logger(logpath)
	log.info("Running python in env: "+sys.executable)
	log.info("Starting MEGA file creation with id: "+str(run_id)+"...")
	log.info("\nOutput file: \t"+outpath)
	log.info("Logfile path: \t"+logpath)

	# create dictionary of fasta records
	fasta_dict = parse_fasta_files(args.fasta_files, log)
	#for header in fasta_dict:
	#	print(header,": \t"+fasta_dict[header])

	# Iterate over entries in rpd file and write to output file the
	# sequence identifier, last known taxonomic classification and sequence from fasta dictionary
	log.info("\nWriting output file...")
	with open(args.rdp_file, 'r') as infile, open(outpath, 'w') as outfile:
		for line in infile:
			header = line.split('\t')[0]
			taxonomic_classification = line.split('\t')[17]
			sequence = fasta_dict[header]
			if args.name_trim.lower() in ['yes', 'y']:
				header = header.split("_")[1]
			#print(header+" - "+taxonomic_classification)
			#print(sequence)
			outfile.write(">"+header+"_"+taxonomic_classification+"\n"+sequence+"\n")
		if args.name_trim.lower() in ['yes', 'y']:
			log.info("trimming sequence headers: \tyes")
		else:
			log.info("trimming sequence headers: \tno")

	infile.close()
	outfile.close()

	# close logger handlers
	log.info("\nClosing logger and exit script...")
	close_logger(log)


# calling MAIN function
main()

