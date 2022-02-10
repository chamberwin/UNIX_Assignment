# Chamberlain UNIX Assignment

## Data Inspection

### Attributes of fang_et_al_genotypes

I started my inspection by looking at the first the last lines of code to understand the headings and layout, then went on to examine size.
    

    (head -n 3; tail -n 3) < fang_et_al_genotypes.txt
This command looks at the first 3 and last 3 lines of the file.

    wc fang_et_al_genotypes.txt
This command provides the line, word, and character/byte count.
  
    du -h fang_et_al_genotypes.txt
This command provides human-readable information about file size.
  
    tail -n +6 fang_et_al_genotypes | awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
This command starts by looking at the last 6 lines in the data in order to remove the header and print the number of columns.
  =

By inspecting this file I learned that:
1. There is 1 row of header that contains Sample_ID, JG_OTU, Group, and SNP ID. Within those columns are SNP data. 
2. There are 2783 lines, 2744038 words, and 11051939 characters/bytes
3. The file is 6.5M
4. There are 986 columns after the header is removed.


###Attributes of snp_position.txt

I started inspecting the file as before with fang_et_al_genotypes.txt

    (head -n 3; tail -n 3) < snp_position.txt
    less snp_position.txt
    wc snp_position.txt
    ls -lh snp_position.txt
    du -h snp_position.txt
    tail -n +6 snp_position.txt | awk -F "\t" '{print NF; exit}' snp_position.txt

By inspecting the file I learned that:
1. There is 1 line of heading (SNP_ID, cdv_marker_id, Chromosome, Position, alt_pos, mult_positions, amplicon, cdv_map_feature.name, gene,   candidate/random, Genaissance_daa_id,   Sequenom_daa_id, count_amplicons, count_cmf, count_gene) and the content is detailed information about SNPs.
2. Using less, I figured out the layout of the column headers do not line up with the SNP data and why it looked so messy intitially.
3. There are 984 lines, 13198 words, and 82763 characters/bytes
4. The size is 81K
5. The size is 41K *Is the difference because of number of lines?*
6. There are 15 columns after the header is removed


##Data Processing

###Maize Data

here is my snippet of code used for data processing

Here is my brief description of what this code does

###Teosinte Data

here is my snippet of code used for data processing

Here is my brief description of what this code does
