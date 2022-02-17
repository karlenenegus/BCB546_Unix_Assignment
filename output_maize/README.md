## output_maize directory

This directory should contain 22 files containing maize SNP data which has been divided into files based on chromosome. Data has been duplicated into files ascending and descending physical coordinates for chromosomes 1 through 10.

For example a file named `maize_chr_1_asc.txt` contains maize sample genotypes of SNP markers located on chromosome 1 and which have been organized in ascending physical order.

Markers mapped to multiple positions are included in `maize_chr_multiple.txt`files and markers with unknown positions are included in `maize_chr_unknown.txt` files. Markers mapped to multiple positions located on a single chromosome have been diverted to `maize_chr_multiple.txt` rather than their chromosome number specific file.

Each file is tab-delimitated and contains Sample_ID information in the first row, SNP_IDs in the first column, chromosome number in the second column, physical position in the third column and genotypes in subsequent rows and columns.

These files have been created using the code provided in the `README.md` file located in the parent directory.
