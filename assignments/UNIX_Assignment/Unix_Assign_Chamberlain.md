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
 

### By inspecting this file I learned that:
1. There is 1 row of header that contains Sample_ID, JG_OTU, Group, and columns that include multiple SNP IDs. Within those columns are SNP data. 
2. There are 2783 lines, 2744038 words, and 11051939 characters/bytes
3. The file is 6.5M
4. There are 986 columns after the header is removed.


### Attributes of snp_position.txt

I started inspecting the file as before with fang_et_al_genotypes.txt

    (head -n 3; tail -n 3) < snp_position.txt
 This command looks at the first 3 and last 3 lines of the file.
    
    less snp_position.txt
This command allows for a slow scroll through the file.

    wc snp_position.txt
This command provides the line, word, and character/byte count.
    
    du -h snp_position.txt
This command provides human-readable information about file size.

    tail -n +6 snp_position.txt | awk -F "\t" '{print NF; exit}' snp_position.txt
This command starts by looking at the last 6 lines in the data in order to remove the header and print the number of columns.

By inspecting the file I learned that:
1. There is 1 line of heading (SNP_ID, cdv_marker_id, Chromosome, Position, alt_pos, mult_positions, amplicon, cdv_map_feature.name, gene,   candidate/random, Genaissance_daa_id,   Sequenom_daa_id, count_amplicons, count_cmf, count_gene) and the content is detailed information about SNPs.
2. Using less, I figured out the layout of the column headers do not line up with the SNP data and why it looked so messy intitially.
3. There are 984 lines, 13198 words, and 82763 characters/bytes
4. The size is 41K *Is the difference because of number of lines?*
5. There are 15 columns after the header is removed

## Data Processing

### Transposing Data
To identify how to extract maize and teosinte from tripsacum, we used the uniq feature to find where tripsacum is via the group column.

    $ cut -f3 fang_et_al_genotypes.txt | sort | uniq
    
This command returned all unique Group ID's in column 3, where many groups other than the targeted 6 from maize and teosinte were present.

Next, we extracted just the groups we needed for the maize and teosinte files. What is new here is that we added "Sample_ID" to ensure we keep that column. I used ">" to overwrite previous txt. output files.

    $ grep -E "(ZMMIL|ZMMLR|ZMMMR|Sample_ID)" fang_et_al_genotypes.txt >     maize_genotypes.txt
    
To ensure Tripsacum data was not also added to the new file, I searched for TRIPS in maize_genotypes.txt

    $ grep "TRIPS" maize_genotypes.txt
    
To ensure "Sample_ID" properly pulled in and file re-written, I grep'ed again.

    $ grep "Sample_ID" maize_genotypes.txt
Once satisfied, I continued the sampe process for the teosinte file:

    $ grep -E "(ZMPBA|ZMPIL|ZMPJA|Sample_ID)" fang_et_al_genotypes.txt > teosinte_genotypes.txt

I used head to ensure the file was laid out correctly.

    $ head teosinte_genotypes.txt
Time to transpose data for each genotype file to make columns into rows using the provided code.

    $ awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt

We used head to see if it worked.

    $ head -n 3 transposed_teosinte_genotypes.txt

Then repeated for maize.

    $ awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt | head -n 3 transposed_maize_genotypes.txt

To make sure the columns read "SNP_ID, Chromsome, Position", in the SNP_positions file before joining, we cut just the columns we needed.

    $ cut -f 1,3,4 snp_position.txt > cut_snp_positions.txt

Next, we need to sort the SNP_position, and transposed maize and teosinte files by SNP_ID (first column) so they join correctly.

    $ sort -k1,1 cut_snp_positions.txt > sorted_snp_position.txt

We used -c to check that is is ordered before moving on.

    $ sort -c k1,1 sorted_snp_position.txt

We then sorted the transposed maize and teosinte files, checking as we went.

    $ sort -k1,1 transposed_maize_genotypes.txt > sort_transposed_maize_genotypes.txt
    $ sort -c -k1,1 sort_transposed_maize_genotypes.txt
    $ sort -k1,1 transposed_teosinte_genotypes.txt > sort_transposed_teosinte_genotypes.txt
    $ sort -c -k1,1 sort_transposed_teosinte_genotypes.txt
    
Satisfied, we joined each sorted transposed file to the sorted SNP file, as the SNP file had the desired column arrangement, using less in between to make sure it joined correctly.

    $ join -1 1 -2 1 sorted_snp_position.txt sort_transposed_maize_genotypes.txt > maize_with_snps.txt
    $ less maize_with_snps.txt
    $ join -1 1 -2 1 sorted_snp_position.txt sort_transposed_teosinte_genotypes.txt > teosinte_with_snps.txt
    $ less teosinte_with_snps.txt
    
### Maize Data
Maize data was first sorted by chromosome (column 2), then position (column 3) in increasing order. To check if it sorted correctly, -c was used.

    $ sort -k2,2n -k3,3n maize_with_snps.txt > all_chrom_maize.txt
    $ sort -k2,2n -k3,3n -c all_chrom_maize.txt

To send each chromsome to its own file, we used "awk" with regular expression, otherwise /1/ will return chromosome 1 and 10. Head and tail were used to check.

    awk '$2 ~ /^1$/' all_chrom_maize.txt > chr1_maize.txt
    head tail chr1_maize.txt

Confident the method worked, we used a pipeline to finish the file creation, using head  n- 3 to double check.

    awk '$2 ~ /^2$/' all_chrom_maize.txt > chr2_maize.txt | head -n 3 | awk '$2 ~ /^3$/' all_chrom_maize.txt > chr3_maize.txt | head -n 3 | awk '$2 ~ /^4$/' all_chrom_maize.txt > chr4_maize.txt | head -n 3 | awk '$2 ~ /^5$/' all_chrom_maize.txt > chr5_maize.txt | head -n 3 | awk '$2 ~ /^6$/' all_chrom_maize.txt > chr6_maize.txt | head -n 3 | awk '$2 ~ /^7$/' all_chrom_maize.txt > chr7_maize.txt | head -n 3 | awk '$2 ~ /^8$/' all_chrom_maize.txt > chr8_maize.txt | head -n 3 | awk '$2 ~ /^9$/' all_chrom_maize.txt > chr9_maize.txt | head -n 3 | awk '$2 ~ /^10$/' all_chrom_maize.txt > chr10_maize.txt | head -n 3 

To put the position in descending order, we added a "r" for reverse for the numerically sorted column 3.

    $ sort -k2,2n -k3,3nr maize_with_snps.txt > rev_all_chrom_maize.txt

To replace the "?" with "-", we used sed.

    $ sed 's/?/-/g' rev_all_chrom_maize.txt > rev_all_chrom_maize_-.txt
  
 We sent the reversed version with the "-" to new files using a pipeline and used head tail and less to double check after.
 
    $ awk '$2 ~ /^1$/' rev_all_chrom_maize_-.txt > chr1_maize_-.txt | awk '$2 ~ /^2$/' rev_all_chrom_maize_-.txt > chr2_maize_-.txt | awk '$2 ~ /^3$/' rev_all_chrom_maize_-.txt > chr3_maize_-.txt | awk '$2 ~ /^4$/' rev_all_chrom_maize_-.txt > chr4_maize_-.txt | awk '$2 ~ /^5$/' rev_all_chrom_maize_-.txt > chr5_maize_-.txt | awk '$2 ~ /^6$/' rev_all_chrom_maize_-.txt > chr6_maize_-.txt | awk '$2 ~ /^7$/' rev_all_chrom_maize_-.txt > chr7_maize_-.txt | awk '$2 ~ /^8$/' rev_all_chrom_maize_-.txt > chr8_maize_-.txt | awk '$2 ~ /^9$/' rev_all_chrom_maize_-.txt > chr9_maize_-.txt | awk '$2 ~ /^10$/' rev_all_chrom_maize_-.txt > chr10_maize_-.txt
    $ head  tail chr1_maize_-txt 
    $ less chr1_maize_-txt

To create file with all uknown SNP positions, we used "awk" to find the pattern and head tail to check it worked.

    $  awk '$3 ~ /unknown/' rev_all_chrom_maize_-.txt > unknown_maize_SNPs.txt
    $ head tail unknown_maize_SNPs.txt

We used "awk" again to create a file for SNPs with multiple positions.

    awk '$3 ~ /multiple/' rev_all_chrom_maize_-.txt > multiple_maize_SNPs.txt


### Teosinte Data
Teosinte data was processed the same as maize data: first sorted by chromosome (column 2), then position (column 3) in increasing order. To check if it sorted correctly, -c was used.

    $ sort -k2,2n -k3,3n teosinte_with_snps.txt > all_chrom_teosinte.txt
    $ sort -k2,2n -k3,3n -c all_chrom_teosinte.txt

To send each chromsome to its own file, we used "awk" with regular expression, otherwise /1/ will return chromosome 1 and 10. Head and tail were used to check. We used a pipeline to finish the file creation, using head  tail to double check.

     awk '$2 ~ /^1$/' all_chrom_teosinte.txt > chr1_teosinte.txt | awk '$2 ~ /^2$/' all_chrom_teosinte.txt > chr2_teosinte.txt | awk '$2 ~ /^3$/' all_chrom_teosinte.txt > chr3_teosinte.txt | awk '$2 ~ /^4$/' all_chrom_teosinte.txt > chr4_teosinte.txt | awk '$2 ~ /^5$/' all_chrom_teosinte.txt > chr5_teosinte.txt | awk '$2 ~ /^6$/' all_chrom_teosinte.txt > chr6_teosinte.txt | awk '$2 ~ /^7$/' all_chrom_teosinte.txt > chr7_teosinte.txt | awk '$2 ~ /^8$/' all_chrom_teosinte.txt > chr8_teosinte.txt | awk '$2 ~ /^9$/' all_chrom_teosinte.txt > chr9_teosinte.txt | awk '$2 ~ /^10$/' all_chrom_teosinte.txt > chr10_teosinte.txt all_chrom_maize.txt > chr10_maize.txt 
    $ head tail chr1_teosinte_txt

To put the position in descending order, we added a "r" for reverse for the numerically sorted column 3.

    $ sort -k2,2n -k3,3nr teosinte_with_snps.txt > rev_all_chrom_teosinte.txt

To replace the "?" with "-", we used sed.

    $ sed 's/?/-/g' rev_all_chrom_teosinte.txt > rev_all_chrom_teosinte_-.txt
  
 We sent the reversed version with the "-" to new files using a pipeline and used head tail and less to double check after.

    $ awk '$2 ~ /^1$/' rev_all_chrom_teosinte_-.txt > chr1_teosinte_-.txt | awk '$2 ~ /^2$/' rev_all_chrom_teosinte_-.txt > chr2_teosinte_-.txt | awk '$2 ~ /^3$/' rev_all_chrom_teosinte_-.txt > chr3_teosinte_-.txt | awk '$2 ~ /^4$/' rev_all_chrom_teosinte_-.txt > chr4_teosinte_-.txt | awk '$2 ~ /^5$/' rev_all_chrom_teosinte_-.txt > chr5_teosinte_-.txt | awk '$2 ~ /^6$/' rev_all_chrom_teosinte_-.txt > chr6_teosinte_-.txt | awk '$2 ~ /^7$/' rev_all_chrom_teosinte_-.txt > chr7_teosinte_-.txt | awk '$2 ~ /^8$/' rev_all_chrom_teosinte_-.txt > chr8_teosinte_-.txt | awk '$2 ~ /^9$/' rev_all_chrom_teosinte_-.txt > chr9_teosinte_-.txt |awk '$2 ~ /^10$/' rev_all_chrom_teosinte_-.txt > chr10_teosinte_-.txt
    $ head  tail chr1_teosinte_-.txt
    $ less chr1_teosinte_-.txt

To create file with all uknown SNP positions, we used "awk" to find the pattern and head tail to check it worked.

    $  awk '$3 ~ /unknown/' rev_all_chrom_teosinte_-.txt > unknown_teosinte_SNPs.txt
    $ head tail unknown_teosinte_SNPs.txt

We used "awk" again to create a file for SNPs with multiple positions.

    awk '$3 ~ /multiple/' rev_all_chrom_teosinte_-.txt > multiple_teosinte_SNPs.txt



