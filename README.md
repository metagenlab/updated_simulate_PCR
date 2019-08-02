# updated_simulate_PCR
Corrected version of simulate_PCR (https://doi.org/10.1186/1471-2105-15-237). Now works when there is only one sequence in queried database. Tested with blast v2.9.0 

# Authors' original description

##########################  
November 2, 2014
v1.2
Bug fix in -mux 0 option. Previously, v1 and v1.1 always ran as -mux 1 even if user had specified -mux 0.

October 22, 2014
v1.1
Bug fixed in the -3prime option.

June 26, 2014
The -3prime option was added.
##############################

#Download simulate_PCR-v1.2.tar.gz
#Extract it like this:
tar -xvzf simulate_PCR-v1.2.tar.gz

#Make sure it is executable:
chmod +x simulate_PCR

## Run it without options to see what the options are
simulate_PCR
## or you might need to precede by perl
perl simulate_PCR 

This is a perl script. You must install the following bioperl libraries where they will be found in @INC, so put them where your PERL5LIB environment variable points:

LWP::Simple
Getopt::Long
Bio::SeqIO

makeblastdb, blastn, and blastdbcmd (http://blast.ncbi.nlm.nih.gov/) must be in your path. As of testing, BLAST should be version 2.2.27+ or higher, so that all options are available.

Usage: simulate_PCR -db <sequence_database> -primers <primer_probe.fasta> -minlen <min_amplicon_len> -maxlen  <max_amplicon_len> -mm <num_mm_allowed> -mux <0|1> -num_threads <#cpu> -max_target_seqs <max#_blast_matches_to_keep_per_query> -word_size <blast_word_size> -evalue <evalue> -extract_amp <0|1> -3prime <perfect_match_length_at3'end_of_primer>
        
Command line options are:
        -db       can be fasta file or blastdb of sequences to search
        -primers  fasta file with primers. If paired primers, they must have fasta deflines >idx|F and >idx|R using same idx. If mux, then defline can be anything. If there are probes, deflines must end with |IO, e.g. >probeName|IO.
        -minlen   minimum amplicon length, default 40
        -maxlen   maximum amplicon length, default 1000
        -mm       number mismatches of primer to target, including indels, and still allow priming. Does not count degenerate bases in mismatch count. default 3
        -mux      0 (paired, i.e. single plex simple rxns with F, R, IO interacting exactly as specified) or 1 (multiplex, complex primer pairings allowed), default 1
        -num_threads  number of threads (CPUs) to use in the BLAST search, default 1
        -max_target_seqs  Maximum number of aligned sequences to keep in the BLAST search, this is the value of blastn -max_target_seqs. default 1000
        -evalue   evalue of BLAST search, default 1000
        -word_size word size to use in blast search, default 4
        -genes    0 (do not find gene annotation of hits) or 1 (do find gene annotation of hits), default 0
        -extract_amp 1 (extract amplicon sequences) or 0 (do not extract amplicon sequences), default 0
        -3prime the number of bases at the 3' end of the primer that must match the database sequence in order for a hit, default 3
 
Example: simulate_PCR -db /usr/mic/bio/blastdb/kpath/all_flatfiles -primers primer.fasta -minlen 80 -maxlen 620 -mm 2 -mux 1 -num_threads  12 -max_target_seqs 10000 -genes 1 -word_size 4 -evalue 1000  -extract_amp 1  -3prime 2
        

Given a file of sequences or a blastdb and a file of primers, simulate_PCR finds the amplicons produced either from specified primer pairings (-mux 0) or complex multiplex pairings (-mux 1).
Report shows # mismatches of each primer to each hit, so you can filter the results for fewer mismatches than specified with -mm.

If -genes 1 is set, then it downloads gene annotations overlapping the amplicon. It will only annotate hits to annotated sequences in Genbank with valid gi.
        
The simulate_PCR_Example.zip contains example input and results.

The first time you run it for a given blast db from the current directory, it creates the db.headers file for blastdb's other than nt. It also saves the blast results.
Subsequent runs from this directory will use the db.headers and db.blastout blast results file already created, so they should run faster. If you  rerun and do not want to re-use previously computed blast results, you need to delete these files first.
        
Output files have prefix primerFile.(mux|pair).db  (eg primers.mux.nt.*).
        Suffixes are 
             .amplicons for the tab deliminated file with all amplified products, positions, id's, etc.
             .amplicon_distribution listing the fragment length distributions for all targets, helpful if you're doing mux and you want to predict gel band positions.
             .one_primer_but_no_amp list of targets that have one primer match but no amplicon in desired length range
             .amplicons.fasta fasta formatted amplicon sequences from each target 
	       .detection_counts_by_triplet.#mismatches list of the number of database sequences detected by each primer pair and primer/probe triplet, sorted by descending count


      Example fasta input for running with -mux 1 or 0, indicating in the case of -mux 0 (singleplex) the way that primers and probes should be paired up. :

>1|F
GGTTAGCACTTCAATGGAATGGA
>1|IO
CGTTTCCTAGTCGCTCAAACGTTATTCCACC
>1|R
CCGAAACCAATCACGTAGGC

>2|F
TTCGGGAGAATTTTCCTTGG
>2|IO
AATCATTCCTCCTTCACTTCACCAACGATGA
>2|R
TCCCAGTCAGGTTTGTAACGG

Example fasta input for running -mux 1  only. Does not work for -mux 0 since no primer pairings are specified:

>p1
TTCGGGAGAATTTTCCTTGG
>p2
CCGAAACCAATCACGTAGGC
>p3
TTCGGGAGAATTTTCCTTGG
>p4
TCCCAGTCAGGTTTGTAACGG
>p5|IO
CGTTTCCTAGTCGCTCAAACGTTATTCCACC
>p6|IO
AATCATTCCTCCTTCACTTCACCAACGATGA

        
Columns in the .amplicons output file are 

amplicon_len (amplicon length)
HitName (characters before the first space in the fasta defline
FP_ID (forward primer ID)
FP_seq (forward primer sequence)
FP_degeneracies (number of degenerate bases)
FP_mismatches (number of mismatches of forward primer to database hit, 	not counting degenerate bases)
RP_ID (reverse primer ID)
RP_seq
RP_degeneracies
RP_mismatches
RevcompRP (reverse complement of reverse primer)
ProbeID
Probe_seq
RevcompProbe
Probe_degeneracies
Probe_mismatches
Probe_startOnAmplicon (position relative to start of amplicon)
ProbeStrand (plus means it is on same strand as FP, minus means it is on 	same strand as RP)
Start (start position of amplicon in the database sequence)
End (end position of amplicon in the database sequence)
Full_Hit_ID (full fasta defline or sequence title in blast db)
SubjectFullLength (length of the database sequence)
AmpliconSeq 
Primary_tag (gene annotation primary tag information of genes that 	overlap the amplicon. Multiple annotations overlapping an amplicon 	are separated by ":::")
Gene
Product
Protein_id
Note (from gene annotation)
