## Control de calidad de datos de secuenciación masiva

### Objetivos de aprendizaje

Entender el formato FastQ.
Entender el concepto de calidad de una secuencia.
Aprender a utilizar software para medir y mejorar la calidad de datos de secuenciación masiva.



###  Descargando la imagen de Docker
**En su terminal deberán ejecutar el siguiente comando:**
```sh
	docker pull joseancorona/curso_transcriptomica:base
	```

 **Descargando los datos:**
 Lo primero que hacemos al recibir nuestros datos es descargarlos. Generalmente están en un formato comprimido llamado tar. Para su conveniencia los datos están cargados en el directorio:
 Los podemos ver usando el comando ls:
   ```sh
   cd /usr/local/data
   ls
   ```
      
   ``` sh
   FastQC_Short.tar.gz  Sp_ds.right.fq.gz  Sp_genome.fa.gz   Sp_hs.right.fq.gz  Sp_log.right.fq.gz  Sp_plat.right.fq.gz
Sp_ds.left.fq.gz     Sp_genes.gtf       Sp_hs.left.fq.gz  Sp_log.left.fq.gz  Sp_plat.left.fq.gz
  ```

### md5sum

 **Usa el comando `md5sum`:**
   ```sh
   md5sum -c MD5.txt
   ```

   **Vamos a generar un directorio de trabajo en el directorio `/usr/local`**
```sh
   $ cd ..
   $ mkdir ANALYSIS
   #Y copiaremos los datos para trabajar:

    cd ANALYSIS
    cp /usr/local/data/FastQC_Short.tar.gz .
   #Podemos descomprimir los datos usando ese mismo comando:

    tar -xvf FastQC_Short.tar.gz
   #Este comando descomprime un directorio llamado FastQC_Short. Entramos en ese directorio:

    cd FastQC_Short
   #Revisamos su contenido:

    ls
   #Partial_SRR2467141.fastq
   #Partial_SRR2467142.fastq
   #Partial_SRR2467143.fastq
   #Partial_SRR2467144.fastq
   #Partial_SRR2467145.fastq
   #Partial_SRR2467146.fastq
   #Partial_SRR2467147.fastq
   #Partial_SRR2467148.fastq
   #Partial_SRR2467149.fastq
   #Partial_SRR2467150.fastq
   #Partial_SRR2467151.fastq
   ```
