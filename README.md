# Aula_somaticoEP1
OBS.: Para todo esse papeline, utilizar o GitPod no terminal Bash
## Fazendo o download dos arquivos que serão utilizados
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

```wget -c https://storage.googleapis.com/gatk-best-practices/somatic-hg38/small_exac_common_3.hg38.vcf.gz.tbi```

```wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg38/chromosomes/chr9.fa.gz
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
