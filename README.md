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

**Enviar job para CGI API**

```python
import requests
headers = {'Authorization': 'renatopuga@gmail.com f3c4d7630ac204bf0b86'}
payload = {'cancer_type': 'HEMATO', 'title': 'Somatic MF WP048', 'reference': 'hg38'}
r = requests.post('https://www.cancergenomeinterpreter.org/api/v1',
                headers=headers,
                files={
                        'mutations': open('/content/df_WP048-cgi.txt', 'rb')
                        },
                data=payload)
r.json()
```

output:
```
ea324a9c29a5e6ef4d55
```

**Status do JobID**

```python
import requests
job_id ="ea324a9c29a5e6ef4d55"

headers = {'Authorization': 'renatopuga@gmail.com f3c4d7630ac204bf0b86'}
r = requests.get('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers)
r.json()
```

output:
```
{'status': 'Done',
 'metadata': {'id': 'ea324a9c29a5e6ef4d55',
  'user': 'renatopuga@gmail.com',
  'title': 'Somatic MF WP048',
  'cancertype': 'HEMATO',
  'reference': 'hg38',
  'dataset': 'input.tsv',
  'date': '2025-12-06 14:05:15'}}
```

**Log Completo do JobID**

```python
import requests
job_id ="ea324a9c29a5e6ef4d55"

headers = {'Authorization': 'renatopuga@gmail.com f3c4d7630ac204bf0b86'}
payload={'action':'logs'}
r = requests.get('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers, params=payload)
r.json()
```

output:
```
{'status': 'Done',
 'logs': ['# cgi analyze input.tsv -c HEMATO -g hg38',
  '2025-12-06 15:05:19,322 INFO     Parsing input01.tsv\n',
  '2025-12-06 15:05:23,170 INFO     Running VEP\n',
  '2025-12-06 15:05:24,089 INFO     Check cancer genes and consensus roles\n',
  '2025-12-06 15:05:24,177 INFO     Annotate BoostDM mutations\n',
  '2025-12-06 15:05:24,218 INFO     Annotate OncodriveMUT mutations\n',
  '2025-12-06 15:05:26,668 INFO     Annotate validated oncogenic mutations\n',
  '2025-12-06 15:05:26,826 INFO     Check oncogenic classification\n',
  '2025-12-06 15:05:26,892 INFO     Matching biomarkers\n',
  '2025-12-06 15:05:26,989 INFO     Prescription finished\n',
  '2025-12-06 15:05:27,003 INFO     Aggregate metrics\n',
  '2025-12-06 15:05:29,952 INFO     Compress output files\n',
  '2025-12-06 15:05:30,013 INFO     Analysis done\n']}

```

**Download do arquivo `.zip`**

Total de 4 arquivos de resultado:

> A definição de cada arquivo pelo CGI (ver no site)
1. alterations.tsv: ...
2. biomarkers.tsv: ...
3. input01.tsv: ...
4. summary.txt: ...

```bash
%%bash
# criar o diretorio com o ID da amostra dentro de results
mkdir -p results/WP048
```

```python
import requests
job_id ="ea324a9c29a5e6ef4d55"

headers = {'Authorization': 'renatopuga@gmail.com f3c4d7630ac204bf0b86'}
payload={'action':'download'}
r = requests.get('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers, params=payload)
with open('/content/results/WP048/WP048-cgi.zip', 'wb') as fd:
    fd.write(r._content)
```

```bash
!unzip /content/results/WP048/WP048-cgi.zip -d /content/results/WP048/
```

```
Archive:  /content/results/WP048/WP048-cgi.zip
  inflating: /content/results/WP048/alterations.tsv  
  inflating: /content/results/WP048/biomarkers.tsv  
  inflating: /content/results/WP048/input01.tsv  
  inflating: /content/results/WP048/summary.txt  
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




