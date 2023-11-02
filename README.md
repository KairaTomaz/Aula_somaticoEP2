# Aula_somaticoEP1
OBS.: Para todo esse papeline, utilizar o GitPod no terminal Bash
## Referências:
(https://github.com/renatopuga/somaticoEP1/blob/main/README.md)

(https://github.com/renatopuga/somaticoEP2)
### Instalando ferramentas
```
brew install sratoolkit
```

```
pip install parallel-fastq-dump
```

### Download do arquivo a ser utilizado
```
wget -c https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/3.0.0/sratoolkit.3.0.0-ubuntu64.tar.gz
tar -zxvf sratoolkit.3.0.0-ubuntu64.tar.gz
export PATH=$PATH://workspace/somaticoEP1/sratoolkit.3.0.0-ubuntu64/bin/
echo "Aexyo" | sratoolkit.3.0.0-ubuntu64/bin/vdb-config
```

```
time parallel-fastq-dump --sra-id SRR8856724 --threads 4 --outdir ./ --split-files --gzip
```

### Fazendo o download das Referências do Genoma hg38 (FASTA, VCFs)

```
wget -c https://storage.googleapis.com/gatk-best-practices/somatic-hg38/af-only-gnomad.hg38.vcf.gz```

```wget -c https://storage.googleapis.com/gatk-best-practices/somatic-hg38/af-only-gnomad.hg38.vcf.gz.tbi```

```wget -c https://storage.googleapis.com/gatk-best-practices/somatic-hg38/1000g_pon.hg38.vcf.gz```

```wget -c https://storage.googleapis.com/gatk-best-practices/somatic-hg38/1000g_pon.hg38.vcf.gz.tbi```

```wget -c https://storage.googleapis.com/gatk-best-practices/somatic-hg38/small_exac_common_3.hg38.vcf.gz```

```wget -c https://storage.googleapis.com/gatk-best-practices/somatic-hg38/small_exac_common_3.hg38.vcf.gz.tbi
```
### Download do exoma
```
wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg38/chromosomes/chr9.fa.gz
```
### Instalando o BWA no gitpod
```
brew install bwa
```

### Descompactando arquivo e indexando
```
gunzip chr9.fa.gz
```

```
bwa index chr9.fa
```

### Instalando Samtools faidx e gerando arquivo
```
brew install samtools
```

```
samtools faidx chr9.fa
```

### Convertendo FASTQ para BAM
```
NOME=WP312; Biblioteca=Nextera; Plataforma=illumina;

bwa mem -t 16 -M -R "@RG\tID:$NOME\tSM:$NOME\tLB:$Biblioteca\tPL:$Plataforma" \
chr9.fa \
SRR8856724_1.fastq.gz \
SRR8856724_2.fastq.gz > WP312.sam
```

## Ao final desta etapa vc deverá ter esses aquivos e diretório:
```
ls
```
![1 EP1](https://github.com/KairaTomaz/Aula_somaticoEP2/assets/149710213/578bef40-bd83-446e-b60a-73cde378f461)

### organizando os arquivos com samtools: fixmate, sort e index
```
time samtools fixmate -@10 WP312.sam WP312.bam
time samtools sort -O bam -@6 -m2G -o WP312_sorted.bam WP312.bam
time samtools index WP312_sorted.bam
time samtools rmdup WP312_sorted.bam WP312_sorted_rmdup_F4.bam
time samtools index WP312_sorted_rmdup_F4.bam
```
### Instalando a ferramenta bedtools
```
brew install bedtools
```

### Gerando o arquivo BED
```
bedtools bamtobed -i WP312_sorted_rmdup_F4.bam > WP312_sorted_rmdup.bed
bedtools merge -i WP312_sorted_rmdup.bed > WP312_sorted_rmdup_merged.bed
bedtools sort -i WP312_sorted_rmdup_merged.bed > WP312_sorted_rmdup_merged_sorted.bed
```
## Ao final desta etapa vc deverá ter esses aquivos e diretório:
```
ls
```
![2 EP1](https://github.com/KairaTomaz/Aula_somaticoEP2/assets/149710213/8f49fca1-4755-4217-8d27-28358a8bfc63)

### Cobertura Média
```
git clone https://github.com/circulosmeos/gdown.pl.git
./gdown.pl/gdown.pl  https://drive.google.com/file/d/1pTMpZ2eIboPHpiLf22gFIQbXU2Ow26_E/view?usp=drive_link WP312_sorted_rmdup_F4.bam
./gdown.pl/gdown.pl  https://drive.google.com/file/d/10utrBVW-cyoFPt5g95z1gQYQYTfXM4S7/view?usp=drive_link WP312_sorted_rmdup_F4.bam.bai
```
```
bedtools coverage -a WP312_sorted_rmdup_merged_sorted.bed \
-b WP312_sorted_rmdup_F4.bam -mean \
> WP312_coverageBed.bed
```
### Realizando filtro no arquivo gerado (total de reads >=20)
```
cat WP312_coverageBed.bed | \
awk -F "\t" '{if($4>=20){print}}' \
> WP312_coverageBed20x.bed
```
### Instalando GATK4
```
wget -c https://github.com/broadinstitute/gatk/releases/download/4.2.2.0/gatk-4.2.2.0.zip
unzip gatk-4.2.2.0.zip
```

### Gerando o arquivo .dict
```
./gatk-4.2.2.0/gatk CreateSequenceDictionary -R chr9.fa -O chr9.dict
```
### Gerando o arquivo interval_list
```
./gatk-4.2.2.0/gatk BedToIntervalList -I WP312_coverageBed20x.bed \
-O WP312_coverageBed20x.interval_list -SD chr9.dict
```
### Convertendo Bed para Interval_list
```./gatk-4.2.2.0/gatk BedToIntervalList -I WP312_coverageBed20x.bed \
-O WP312_coverageBed20x.interval_list -SD chr9.dict
```
## Ao final desta etapa vc deverá ter esses aquivos e diretórios:
```
ls
```
![3 EP1](https://github.com/KairaTomaz/Aula_somaticoEP2/assets/149710213/019e0272-140b-412e-8e4a-aa82666d5c00)

### Calculando contaminação
```
./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I WP312_sorted_rmdup_F4.bam  \
	-V af-only-gnomad.hg38.vcf.gz \
	-L WP312_coverageBed20x.interval_list \
	-O WP312.table
```
```
./gatk-4.2.2.0/gatk CalculateContamination \
-I WP312.table \
-O WP312.contamination.table
```
### Visualizando resultado
```
cat WP312.contamination.table
```
### MuTect2 Call
```
./gatk-4.2.2.0/gatk Mutect2 \
  -R chr9.fa \
  -I WP312_sorted_rmdup_F4.bam \
  --germline-resource af-only-gnomad.hg38.vcf.gz  \
  --panel-of-normals 1000g_pon.hg38.vcf.gz \
  -L WP312_coverageBed20x.interval_list \
  -O WP312.somatic.pon.vcf.gz
```
### MuTect2 FilterMutectCalls
```
./gatk-4.2.2.0/gatk FilterMutectCalls \
	-R chr9.fa \
	-V WP312.somatic.pon.vcf.gz \
	--contamination-table WP312.contamination.table \
	-O WP312.filtered.pon.vcf.gz
```

