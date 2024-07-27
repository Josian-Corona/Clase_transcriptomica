
# Alineamiento de lecturas

## Objetivos de aprendizaje

- Alinear datos de secuenciación a una referencia (genomas o transcriptomas)
- Entender cómo interpretar datos de alineamiento de secuenciación masiva
- Primer acercamiento a los formatos de coordenadas SAM y BAM

## Datos requeridos

Para proceder con esta práctica, se requieren los resultados de tarea de ensamble de transcriptoma de la clase pasada:

- Archivos fastq de lecturas filtradas por calidad usando Trimmomatic via Trinity
- Archivo fasta de ensamble de transcriptoma completo

Usaremos los archivos fastq que utilizamos la clase pasada.

También usaremos el genoma de referencia de nuestro organismo, *Saccharomyces pombe*:

```bash
 wget https://liz-fernandez.github.io/PBI_transcriptomics/datasets/genome/Sp_genome.fa
```

Los datos ya se encuentran en su directorio de datos en Docker.

Vamos a alinear las lecturas y transcritos al genoma utilizando HISAT2

Pueden encontrar el manual en el siguiente [link](https://daehwankimlab.github.io/hisat2/manual/).

Creemos un directorio para guardar nuestros análisis:

```bash
 cd /usr/local/
 mkdir ANALYSIS
 cd ANALYSIS
 mkdir MAPPING
 cd MAPPING
```

Copiemos los archivos de las lecturas y el genoma:

```bash
cp /usr/local/data/Sp*fq.gz .
cp /usr/local/data/Sp_genome.fa.gz .
```

Y descomprimimos el genoma:

```bash
gunzip Sp_genome.fa.gz
```

### Alineando las lecturas filtradas al genoma

Primero generaremos un índice de hisat2 para el genoma:

```bash
hisat2-build Sp_genome.fa Sp_genome 
```

El índice se guarda en archivos con extensión .ht2:

```bash
ls *ht2
```
```plaintext
Sp_genome.1.ht2
Sp_genome.2.ht2
Sp_genome.3.ht2
Sp_genome.4.ht2
Sp_genome.5.ht2
Sp_genome.6.ht2
Sp_genome.7.ht2
Sp_genome.8.ht2
```

Usamos hisat2 para alinear las lecturas. Este programa nos permite dividir lecturas que atraviesan sitios de splicing:

```bash
 hisat2 --max-intronlen 300 --min-intronlen 20 -x ./Sp_genome -1 Sp_log.left.fq.gz  -2 Sp_log.right.fq.gz -S Sp_log.sam
```

```plaintext
37915 reads; of these:
  37915 (100.00%) were paired; of these:
    2945 (7.77%) aligned concordantly 0 times
    34969 (92.23%) aligned concordantly exactly 1 time
    1 (0.00%) aligned concordantly >1 times
    ----
    2945 pairs aligned concordantly 0 times; of these:
      278 (9.44%) aligned discordantly 1 time
    ----
    2667 pairs aligned 0 times concordantly or discordantly; of these:
      5334 mates make up the pairs; of these:
        3354 (62.88%) aligned 0 times
        1980 (37.12%) aligned exactly 1 time
        0 (0.00%) aligned >1 times
95.58% overall alignment rate
```

Exploramos el resultado, el cuál es un archivo tipo SAM.

```bash
$ head Sp_log.sam
```

### El formato SAM

Veamos un ejemplo más pequeño de este formato. Supongamos que tenemos el siguiente alineamiento:

```plaintext
Coor    12345678901234 5678901234567890123456789012345
ref AGCATGTTAGATAA**GATAGCTGTGCTAGTAGGCAGTCAGCGCCAT
+r001/1       TTAGATAAAGGATA*CTG
+r002        aaaAGATAA*GGATA
+r003      gcctaAGCTAA
+r004                    ATAGCT..............TCAGC
-r003                           ttagctTAGGC
-r001/2                                       CAGCGGCAT
```

El formato SAM correspondiente será el siguiente:

```plaintext
@HD VN:1.5  SO:coordinate
@SQ SN:ref  LN:45
r001    99  ref 7   30  8M2I4M1D3M  =   37  39  TTAGATAAAGGATACTG   *
r002    0   ref 9   30  3S6M1P1I4M  *   0   0   AAAAGATAAGGATA  *
r003    0   ref 9   30  5S6M    *   0   0   GCCTAAGCTAA *   SA:Z:ref,29,-,6H5M,17,0;
r004    0   ref 16  30  6M14N5M *   0   0   ATAGCTTCAGC *
r003    2064    ref 29  17  6H5M    *   0   0   TAGGC   *   SA:Z:ref,9,+,5S6M,30,1;
r001    147 ref 37  30  9M  =   7   -39 CAGCGGCAT   *   NM:i:1
```

El formato SAM es un formato de texto plano que nos permite guardar datos de secuenciación en formato ASCII delimitado por tabulaciones.

Está compuesto de dos secciones principales:

- El encabezado
- El alineamiento

La sección del encabezado comienza con el caracter @ seguido por uno de los códigos de dos letras que sirven para características de los alineamientos en este archivo. Cada línea está delimitada por tabulaciones y, además de las líneas que comienzan con @CO, cada campo de datos tiene el formato TAG:VALUE, en donde TAG es una cadena de dos caracteres que define el formato y contenido de VALUE.

El encabezado no es indispensable pero contiene información acerca de la versión del archivo así como si está ordenado o no. Por ello es recomendable incluirlo.

La sección de alineamiento contiene la siguiente información:

- **QNAME**: Nombre de la referencia, QNAME (SAM)/Nombre de la lectura (BAM). Se utiliza para agrupar alineamientos que están juntos, como es el caso de alineamientos de lecturas por pares o una lectura que aparece en alineamientos múltiples.
- **FLAG**: Set de información describiendo el alineamiento. Provee la siguiente información:
  - ¿Hay múltiples fragmentos?
  - ¿Todos los fragmentos están bien alineados?
  - ¿Está alineado este fragmento?
  - ¿No ha sido alineado el siguiente fragmento?
  - ¿Es esta referencia la cadena inversa?
  - ¿El siguiente fragmento es la cadena reversa?
  - ¿Es este el primer fragmento?
  - ¿Es este el último fragmento?
  - ¿Es este un alineamiento secundario?
  - ¿Esta lectura falló los filtros de calidad?
  - ¿Es esta lectura un duplicado por PCR o óptico?
- **RNAME**: Nombre de la secuencia de referencia.
- **POS**: Posición de alineamiento izquierda (base 1).
- **MAPQ**: Calidad del alineamiento.
- **CIGAR**: Cadena CIGAR.
- **RNEXT**: Nombre de referencia del par (mate) o la siguiente lectura.
- **PNEXT**: Posición del par (mate) o la siguiente lectura.
- **TLEN**: Longitud del alineamiento.
- **SEQ**: La secuencia de prueba de este alineamiento (en este caso la secuencia de la lectura).
- **QUAL**: La calidad de la lectura.
- **TAGs**: Información adicional.

### Cadenas CIGAR

La secuencia alineada a la referencia puede tener bases adicionales que no están en la referencia o puede no tener bases en la lectura que sí están en la referencia. La cadena CIGAR es una cadena que codifica cada base y la característica de cada una en el alineamiento.

Por ejemplo, la cadena CIGAR:

```plaintext
CIGAR: 3M1I3M1D5M
```

indica que las primeras 3 bases de la lectura alinean con la referencia (3M), la siguiente base no existe en la referencia (1I), las siguientes 3 bases alinean con la referencia (3M), la siguiente base no existe en la lectura (1D), y 5 bases más alinean con la referencia (5M).

Como pueden ver, estos archivos contienen muchísima información que puede ser analizada usando scripts que arrojen estadísticas del alineamiento. Programas como Picard realizan este tipo de análisis.

La versión comprimida de los archivos tipo SAM se conoce como BAM (binary sam). Convirtamos el archivo SAM a BAM usando samtools:

```bash
 samtools view -S -b Sp_log.sam > Sp_log.bam
```

No abrimos los archivos BAM ya que, dado que están en formato binario, son ilegibles. Para poder leerlos necesitamos usar samtools con el comando:

```bash
 sam

tools view Sp_log.bam | head
```

### Tarea 1 - Alineando las muestras restantes al genoma

Alineamos una muestra al genoma pero queremos alinear todas las muestras.
Alinea el resto de las muestras y guarda los resultados (en formato bam), ya que los necesitarás en prácticas subsecuentes.

### Tarea 2 (Opcional) - Alineando las lecturas filtradas al transcriptoma

Hemos alineado las lecturas al genoma pero queremos alinearlas también directamente al transcriptoma. Revisa el manual de HISAT2 y usa las opciones que nos permiten mapear lecturas directamente a transcriptomas. Deberás alinear cada muestra de manera independiente. Alinea el resto de las muestras y guarda los resultados (en formato bam), ya que los necesitarás en prácticas subsecuentes.

Pista: No podrán utilizar el índice generado previamente.
