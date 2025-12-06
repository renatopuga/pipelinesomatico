# pipelinesomatico
Pipeline Somático  - Do VCF (anotado) até o CGI Classicação

**1. Clonar o git lmabrasil-hg38**

```bash
!git clone https://github.com/renatopuga/lmabrasil-hg38.git
```

output:
```
Cloning into 'lmabrasil-hg38'...
remote: Enumerating objects: 226, done.
remote: Counting objects: 100% (168/168), done.
remote: Compressing objects: 100% (108/108), done.
remote: Total 226 (delta 90), reused 114 (delta 56), pack-reused 58 (from 1)
Receiving objects: 100% (226/226), 8.63 MiB | 17.44 MiB/s, done.
Resolving deltas: 100% (106/106), done.
```

Agora, vá até o github lmabrasil-hg38 na sessão Usando CGI via API Rest no Google Colab.

```bash
%%bash
# cortar pelas colunas de 1 a 4 e criar um novo arquivo chamado df_WP048-cgi.txt
# 1: CHROM (converte CHROM para CHR) formato que o CGI gosta
# 2: POS
# 3: REF
# 4. ALT
cut -f1-4 /content/lmabrasil-hg38/vep_output/liftOver_WP048_hg19ToHg38.vep.filter.tsv | sed -e "s/CHROM/CHR/g"  > df_WP048-cgi.txt

# listar as 10 primeiras linhas
head df_WP048-cgi.txt
```

output:
```
CHR	POS	REF	ALT
chr1	114716123	C	T
chr9	5073770	G	T
```

**Visualizar a tabela `alterations.tsv`**

Instalar a lib pandas
```bash
!pip install pandas
```

```python
import pandas as pd
pd.read_csv('/content/results/WP048/alterations.tsv',sep='\t',index_col=False, engine= 'python')
```

output:

| Input ID | CHROMOSOME | POSITION |       REF | ALT | CHR |  POS |  ALT_TYPE | STRAND | CGI-Sample ID |     ... | CGI-Oncogenic Prediction |           CGI-External oncogenic annotation |             CGI-Mutation |    CGI-Consequence |   CGI-Transcript |      CGI-STRAND | CGI-Type | CGI-HGVS |                                         CGI-HGVSc |                   CGI-HGVSp |                               |
|---------:|-----------:|---------:|----------:|----:|----:|-----:|----------:|-------:|--------------:|--------:|-------------------------:|--------------------------------------------:|-------------------------:|-------------------:|-----------------:|----------------:|---------:|---------:|--------------------------------------------------:|----------------------------:|-------------------------------|
|     0    |  input01_1 |        1 | 114716123 |   C |   T | chr1 | 114716123 |    snp |             + | input01 |                      ... | driver (boostDM: non-tissue-specific model) | cgi,oncokb,clinvar:13901 | chr1:114716123 C>T | missense_variant | ENST00000369535 |        + |      SNV |     ENST00000369535:c.38G>A;p.(Gly13Asp);p.(G13D) |   ENST00000369535.5:c.38G>A |  ENSP00000358548.4:p.Gly13Asp |
|     1    |  input01_2 |        9 |   5073770 |   G |   T | chr9 |   5073770 |    snp |             + | input01 |                      ... |                    passenger (oncodriveMUT) | cgi,oncokb,clinvar:14662 |   chr9:5073770 G>T | missense_variant | ENST00000381652 |        + |      SNV | ENST00000381652:c.1849G>T;p.(Val617Phe);p.(V617F) | ENST00000381652.4:c.1849G>T | ENSP00000371067.4:p.Val617Phe |
|          |            |          |           |     |     |      |           |        |               |         |                          |                                             |                          |                    |                  |                 |          |          |                                                   |                             |                               |

## Anexo

Como criar uma tabela mais complexa em MarkDown

- https://www.tablesgenerator.com/markdown_tables




