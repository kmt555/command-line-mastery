# Process Substitution '<()'

Certain environments or with certain commands '<()' syntax can cause `/dev/fd/63`. To overcome such problems, one can replace <() with a named pipe (FIFO) or a temporary file.

## Example: Replacing <() with Named Pipes

```
diff <(command1) <(command2)

mkfifo fifo1 fifo2

command1 > fifo1 &
command2 > fifo2 &

diff fifo1 fifo2

rm fifo1 fifo2
```

## Example: Replacing <() with Temporary Files

```
paste <(command1) <(command2)

tempfile1=$(mktemp)
tempfile2=$(mktemp)

command1 > "$tempfile1"
command2 > "$tempfile2"

paste "$tempfile1" "$tempfile2"

rm "$tempfile1" "$tempfile2"
```


# awk one liners

Print first and last character of a pattern, and merge it with @:
```
echo "examplepattern" | awk '{print substr($0, 1, 1) "@" substr($0, length($0), 1)}'
```


## gerex


```
^starts with
$ends with
```

##

Highlight the matching pattern:

```
$ grep --color='auto' VT994904 varscan_20230928.split.vcf
```

## VCF files

List affected chromosomes in a single line, separated by ';':

`gunzip -c variants_vcfs/VT994904.variants.vcf.gz | grep -v "#" | cut -f1 | sort | uniq | tr '\n' ';' | sed 's/;$//'`

### Count the number of SNPs per chromosome, including multi-allelic sites

```
sample=file.vcf

# split multi allelic sites:
bcftools norm -m-both ${sample} | bcftools query -f '%CHROM\t%ALT\n' | awk -v s=$NAME '{if($2 ~ /,/){multi[$1]++} else {snps[$1]++}} END {for(chr in snps) {print s"\t"chr"\t"snps[chr]"\t"multi[chr]}}' 

Lines   total/split/realigned/skipped:	34/8/0/0
	HPV62	43

# count SNP and multi-allelic sites in separate:
bcftools query -f '%CHROM\t%ALT\n' ${sample} | awk -v s=$NAME '{if($2 ~ /,/){multi[$1]++} else {snps[$1]++}} END {for(chr in snps) {print s"\t"chr"\t"snps[chr]"\t"multi[chr]}}' 

	HPV62	26	8
```

## Processing ANNOVAR output

Typical ANNOVAR output will look similar to this:

```
line1	synonymous SNV	L1HPV6:HPV6:exon1:c.A942C:p.G314G,	HPV6	942	942	A	C	HPV6	942	.	A	C	.	PASS	ADP=47;WT=2;HET=51;HOM=0;NC=2109	GT:GQ:SDP:DP:RD:AD:FREQ:PVAL:RBQ:ABQ:RDF:RDR:ADF:ADR	./.:.:0 ./.:.:0 
```

To transform it into a tabular form with only select information, eg, 

```
SomeInfo1	SomeInfo2	CHROM	VARINFO	VARSITE	gDNA	.	FILTER	INFO	SomeInfo3	VARTYPE
SAMPLENAME	ASSAY	HPV124	L1HPV124:HPV124:exon1:c.T1431C:p.Y477Y	1431	c.1431T>C	.	PASS	ADP=0;WT=0;HET=2;HOM=0;NC=2160	GROUP	synonymous SNV
```

one would do:

```
fbname=SAMPLENAME_ASSAY_GROUP.bam

id1=$(echo "$fbname" | cut -d'_' -f1)
id2=$(echo "$fbname" | cut -d'_' -f2)
id3=$(echo "$fbname" | cut -d'_' -f3)
id12=${id1}_${id2}

totalreads=`samtools view ${bam} | wc -l`

cut -f2,3,4,5,7,8,14,15,16 --output-delimiter=@ annovar/${fbname}.exonic_variant_function | sed 's/,@/@/' | awk -F"@" -v var="${id12}" -v var2="${totalreads}" -v var3=${id3} '{print var3,var,$3,$2,$4
,"c."$4$5">"$6, $7, $8, $9, var2, $1}' OFS="\t" 
```




