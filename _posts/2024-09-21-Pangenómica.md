---
title: "Pangenómica"
layout: page
---
## Instalación de anvio
Vamos a utilizar wsl nuevamente para instalar Anvio y poder visualizar los análisis pangenómicos. Primero preparemos un ambiente llamado anvio-8. Si no tiene wsl, baje al final y revise cómo correrlo en el cluster.
El tutorial está basado en el mismo de la página de <a href="https://merenlab.org/2016/11/08/pangenomics-v2/">Anvio</a> <br> 
```yml
#Actualizar conda
conda update conda
conda create -y --name anvio-8 python=3.10
conda activate anvio-8
pip install matplotlib==3.5.1
```
## Instalación de dependencias
```yml
conda install -y -c conda-forge -c bioconda python=3.10 \
        sqlite prodigal idba mcl muscle=3.8.1551 famsa hmmer diamond \
        blast megahit spades bowtie2 bwa graphviz "samtools>=1.9" \
        trimal iqtree trnascan-se fasttree vmatch r-base r-tidyverse \
        r-optparse r-stringi r-magrittr bioconductor-qvalue meme ghostscript
conda install -y -c bioconda fastani
#Si fastani no se instala tranquilos!
```
## Descarguemos anvio
```yml
curl -L https://github.com/merenlab/anvio/releases/download/v8/anvio-8.tar.gz \
        --output anvio-8.tar.gz
#Instalación con pip
pip install anvio-8.tar.gz
```
Si el comando anterior no sirve, hay que reinstalar una dependencia de forma manual.
```yml
wget https://files.pythonhosted.org/packages/c4/aa/94c42a2dd30c4923bffe3ab59e4416c3f7e72dbd32a89bcdd8d43ff1d5d7/datrie-0.8.2-pp373-pypy36_pp73-win32.whl
mv datrie-0.8.2-pp373-pypy36_pp73-win32.whl datrie-0.8.2-cp310-none-any.whl
pip install datrie-0.8.2-cp310-none-any.whl
#Probamos de nuevo
pip install anvio-8.tar.gz
```
Muy bien! Ahora descarguemos los fatos
```yml
wget https://ndownloader.figshare.com/files/28834476 -O Prochlorococcus_31_genomes.tar.gz
tar -zxvf Prochlorococcus_31_genomes.tar.gz
cd Prochlorococcus_31_genomes
anvi-migrate *.db --migrate-quickly
```
Los comandos anteriores descargan y descomprimen los datos que son bases de datos de anvio.
## Análisis pangenómico
Revise el archivo external-genomes con el comando cat, ustedes van a necesitar uno así para sus genomas.
El comando anvi-pan-genome puede tardar varios minutos (15-30) dependiendo de su computador. Modifique el número de threads dependiendo de los que tenga su laptop.
```yml
anvi-gen-genomes-storage -e external-genomes.txt \
                         -o PROCHLORO-GENOMES.db
anvi-pan-genome -g PROCHLORO-GENOMES.db \
                --project-name "Prochlorococcus_Pan" \
                --output-dir PROCHLORO \
                --num-threads 6 \
                --minbit 0.5 \
                --mcl-inflation 10 \
                --use-ncbi-blast
#Importar subgrupos. Puede abrir y revisar el archivo layer-additional-data
anvi-import-misc-data layer-additional-data.txt \
                      -p PROCHLORO/Prochlorococcus_Pan-PAN.db \
                      --target-data-table layers
```
## Análisis en Kabré
Para los que no tienen wsl (:C) ingresen al cluster pero ahora utilizando el siguiente comando:
```yml
#Para conectarse reemplace su usuario
ssh -p 22022 -L 8080:localhost:8080 usuario@kabre.cenat.ac.cr
#Digite su contraseña
```
Ahora copien la carpeta y ejecuten el slurm
```yml
module load miniconda/anvio-7.1
cp -r /work/bmendoza/CURSO/PROCHLORO PROCHLORO
sbatch pangenome.slurm
```
De aquí en adelante tanto los que trabajen con wsl como los que trabajen con el clúster deben ejecutar los comandos directamente en la terminal.
## Visualización del Pangenoma
```yml
anvi-display-pan -g PROCHLORO-GENOMES.db \
                 -p PROCHLORO/Prochlorococcus_Pan-PAN.db
```
Para visualizar el pangenoma, vaya a su navegador de confianza y pegue la siguiente dirección: http://localhost:8080
## Importar layers
```yml
anvi-import-state -p PROCHLORO/Prochlorococcus_Pan-PAN.db \
                  --state pan-state.json \
                  --name default
```
## Nueva visualización 
```yml
anvi-display-pan -g PROCHLORO-GENOMES.db \
                 -p PROCHLORO/Prochlorococcus_Pan-PAN.db
```

