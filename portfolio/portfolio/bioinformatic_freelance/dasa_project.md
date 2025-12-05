## **Dasa Challenge (eng-US)**

### About
This project implements a pipeline developed to meet the challenge of the selection process for the bioinformatics position at Dasa. Snakemake was used to integrate different programs and custom scripts, enabling the annotation of genetic variants from a VCF file. The annotations include the gene, dbSNP ID, and the frequencies of reference and alternative alleles relative to the global population of the 1000 Genomes 30x project. The results were made available through an API and an interactive web interface developed with Flask, allowing the filtering of variants by frequency and depth (DP).

The scripts developed for annotation are available in `/projeto_dasa/annotation/scripts`.
The annotation results are generated in the `/projeto_dasa/annotation/results` directory.
The scripts developed for the interface are available in `/projeto_dasa/interface`.

### Instructions
#### 1. Clone the repository:
``` bash
git clone https://github.com/gbrl-mendes/projeto_dasa.git
```
#### 2. Build the project image:
``` bash
docker build -t projeto_dasa projeto_dasa/docker_dasa/
```
#### 3. Start the Container:
``` bash
docker run -it -p 8181:80 projeto_dasa
```
#### 4. Run the Snakefile:
``` bash
snakemake --cores <X> # choose a number of cores according to your machine
```
The Snakefile executes all annotation steps for the VCF file provided for the challenge. The annotation results are made available in the `/projeto_dasa/annotation/results` directory.

The last step, initiated with the `get_frequencies` rule, involves retrieving the population frequencies of the variants, and is performed through a bash script that queries the NCBI SNP database to obtain the frequencies. This process may take several hours, depending on available resources. Therefore, if you do not want to wait, you can interrupt the pipeline. Either way, the interface will be able to access the results, as the annotation has already been previously executed, and the results are available in the `/projeto_dasa/interface/data` repository. If you wish to view your own results using the interface, copy the `NIST_dbSNPid_func_annot.vcf` and `NIST_ids_frequencies.txt` files from the `/projeto_dasa/annotation/results` directory to the `/projeto_dasa/interface/data` directory.

#### 5. **Start the Flask server**:
``` bash
python app.py
```
#### 6. **Access the interface:**
	
	1. If you built the Docker image on a local machine, open the browser and type:
		http://localhost:8181
	2. If you built the Docker image on a server, open the browser on your local machine and type:
		http://<server_address>:8181

### Contact
For more information, contact me through my [e-mail](mailto:gabrielmendesbrt@outllok.com) address 😊

---

## **Desafio Dasa (pt-BR)**

### Sobre
Este projeto implementa um pipeline desenvolvido para atender ao desafio do processo seletivo para a vaga de bioinformática na Dasa. O Snakemake foi utilizado para integrar diferentes programas e scripts personalizados, permitindo a anotação de variantes genéticas a partir de um arquivo VCF. As anotações incluem o gene, o ID dbSNP e as frequências dos alelos de referência e alternativo em relação à população global do projeto 1000 Genomes 30x. Os resultados foram disponibilizados por meio de uma API e de uma interface web interativa desenvolvida com Flask, permitindo a filtragem de variantes por frequência e profundidade (DP).

Os scripts desenvolvidos para a anotação estão disponíveis em `/projeto_dasa/annotation/scripts`.
Os resultados da anotação são gerados no diretório `/projeto_dasa/annotation/results`.
Os scripts desenvolvidos para a interface estão disponíveis em `/projeto_dasa/interface`.

### Instruções
#### 1. Clone o repositório:
``` bash
git clone https://github.com/gbrl-mendes/projeto_dasa.git
```
#### 2. Construa a imagem do projeto:
``` bash
docker build -t projeto_dasa projeto_dasa/docker_dasa/
```
#### 3. Inicie o Container:
``` bash
docker run -it -p 8181:80 projeto_dasa
```
#### 4. Execute o Snakefile:
``` bash
snakemake --cores <X> # escolha um numero de cores de acordo com sua maquina
```
O Snakefile executa todas as etapas de anotação do arquivo VCF fornecido para o desafio. Os resultados da anotação são disponibilizados no diretório `/projeto_dasa/annotation/results`.

A última etapa, iniciada com a regra `get_frequencies`, envolve a recuperação das frequências populacionais das variantes, e é realizada por meio de um bash script que consulta o banco de dados NCBI SNP para obter as frequências. Este processo pode levar algumas horas, dependendo dos recursos disponíveis. Portanto, caso não queira aguardar, é possível interromper o pipeline. De qualquer forma, a interface conseguirá acessar os resultados, pois a anotação já foi previamente executada, e os resultados estão disponíveis no repositório `/projeto_dasa/interface/data`. Caso deseje visualizar os próprios resultados usando a interface, copie os arquivos `NIST_dbSNPid_func_annot.vcf` e `NIST_ids_frequencies.txt` do diretório `/projeto_dasa/annotation/results` para o diretório `/projeto_dasa/interface/data`.

#### 5. **Inicie o servidor Flask**:
``` bash
python app.py
```
#### 6. **Acesse a interface:**
	
	1. Se você construiu a imagem Docker em uma máquina local, abra o navegador e digite:
		http://localhost:8181
	2. Se você construiu a imagem Docker em um servidor, abra o navegador na sua máquina local e digite:
		http://<endereço_do_servidor>:8181

### Contato 
Para mais informações, entre em contato comigo através do meu endereço de [e-mail](mailto:gabrielmendesbrt@outllok.com) 😊
