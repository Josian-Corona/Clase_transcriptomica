# Análisis transcriptómicos

Estamos realizando un estudio transcriptómico para entender la expresion de genes en un microorganismo. Hemos logrado aislar RNA en distintas condiciones. Queremos realizar un análisis completo de la expresión diferencial de estos transcritos.


Durante esta practica realizaremos el análisis desde identificar la calidad de las secuencias hasta medir si los cambios en expresión son significativos usando métodos estadísticos.

Esta mañana recibimos un correo de nuestro colaborador:

Dear collaborator,

I have great news. All samples passed the QC and were successfully sequenced in our Hi-Seq 2000. You can download the data from the URL below. The md5sum files are also in the tar-ball.

https://drive.google.com/open?id=0B9ZVSRlHL8cIbm5EUXczdzM4a2M

Can’t wait to see if there are indeed major changes to the transcriptome!

Happy data wranggling!

Professor X

---------------

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
   Partial_SRR2467141.fastq
   Partial_SRR2467142.fastq
   Partial_SRR2467143.fastq
   Partial_SRR2467144.fastq
   Partial_SRR2467145.fastq
   Partial_SRR2467146.fastq
   Partial_SRR2467147.fastq
   Partial_SRR2467148.fastq
   Partial_SRR2467149.fastq
   Partial_SRR2467150.fastq
   Partial_SRR2467151.fastq
   ```

   ## El nombre de los archivos fastq

   En este caso el nombre de los archivos ha sido asignado de manera arbitraria pero muchas veces contienen información importante. Por ejemplo, los archivos de Illumina generalmente tienen el siguiente formato:
```
<nombre de la muestra>_<secuencia identificadora (barcode)>_L<línea (3 dígitos)>_R<número de lectura (read)>_<número del set (3 dígitos)>.fastq.gz

```
Ejemplo: NA10831_ATCACG_L002_R1_001.fastq.gz

Es ideal respetar estos nombres para evitar perder información.

### Revisemos uno de los archivos usando el comando head. 

Normalmente nos muestra las primeras 10 líneas de una archivo pero con la bandera -n nos mostrará las primeras 12:

```sh
head -n 12 Partial_SRR2467141.fastq

@SRR2467141.1 SALLY:472:C6NCDACXX:8:1101:1392:1873 length=101
NTTATTTTTTTCGTTCTTCTTGAAGAAGACGTTACCTACGGCGTATTTGCCCATCTCAGGTATGTCTAGATCAAGATCTAACTTGAATTCTCTTTTCATAA
+SRR2467141.1 SALLY:472:C6NCDACXX:8:1101:1392:1873 length=101
#1:=BDDDHDHB?<?CFGGGC9@FF@GGGG>EEEDGDHGFHGE;AEFH>AC@D;@B>C>CCC@C>>DDCC3:>AA5>CC>>CCD>@CCDCDCCCCC@C@>C
@SRR2467141.2 SALLY:472:C6NCDACXX:8:1101:1326:1950 length=101
CACCCATTGACTGGCCAAATGCCCCATTATTTTGAGTATTGTTATTTCCAAATAAACTGTTACTATTACTGCCAGCGGCAGAAGTGAATCCACAGATCGGA
+SRR2467141.2 SALLY:472:C6NCDACXX:8:1101:1326:1950 length=101
??@DFFFFHHFH?GEHEHIDEHCCFEHIE@GIGCHG<DGGIHIGIGGIGHIIFIHFFGDHGGIIHIGIIIIGGEH@EADB>=AA>3@>CCCCCCBC@C###
@SRR2467141.3 SALLY:472:C6NCDACXX:8:1101:1477:1959 length=101
CATCTTTTCTTTAGGCACATCATCCTGATAAGTGTACTTACCAGGATATATACCATCGGTATTGATGTTATCGGCATCACATAAAACTAATTCACCAGAAA
+SRR2467141.3 SALLY:472:C6NCDACXX:8:1101:1477:1959 length=101
@CCFFFFFHHHHHJJJIIJJIJJJJJJEIJJJIIJJJIJIJJIJJIIIJJJIIIHIIIJDHIJJIGJJIJJJJJIHHHFFFFFEEEEEEDDEDDDDDDDDD
```

Este archivo se encuentra en formato fastq. Este es el formato que general la mayoría de los secuenciadores actuales. Este formato es una modificación del formato de secuencias Fasta. Se pueden reconocer porque terminan con la extensión .fq o .fastq.

Los archivos FastQ son archivos planos de texto ASCII que contienen tanto las secuencias detectadas por el secuenciador así como información acerca de la calidad de cada uno de estos nucleótidos.

**¡No cambies archivos fastq!**

Los archivos fastq que recibas después de un experimento de secuenciación serán los que te soliciten cuando quieras publicar un artículo. Idealmente, realiza un para de copias en distintos dispositivos en cuanto los recibas y no los modifiques manualmente.

También es recomendable que, de tener la información a la mano, crees un pequeño archivo README (de texto plano) que esté en el mismo directorio que tus archivos fastq y describa como se realizó el experimento, cuantas réplicas hay de cada condición, que índices y controles se utilizaron, etc. Esto te ayudará enormemente al tratar de interpretar tus resultados.

Y vemos que cada secuencia esta representada por 4 líneas.

**Nombre de la secuencia** - Esta línea comienza con el símbolo @, seguido por el nombre de la lectura.

**La secuencia nucleotídica.**

**Segundo título** - Esta línea comienza con el símbolo +. Generalmente la información es la misma que en la primera línea pero también puede estar en blanco siempre y cuando contenga el símbolo de +.

**Información de calidad** - Contiene una cadena de caracteres ASCII. Cada uno de estos caracteres corresponde a un nucleótido de la secuencia y representa la calidad del mismo. El puntaje de cada nucleótido indica el nivel de confianza que se tiene en esa base. Niveles altos indican que la base se ha reportado correctamente mientras que niveles bajos sugieren que hay incertidumbre acerca de la secuencia real en esta posición.
El nombre de la secuencia también contiene información importante. En este caso, los datos de Illumina proporcionan la siguiente información:



El nombre de la secuencia también contiene información importante. En este caso, los datos de Illumina proporcionan la siguiente información:
```
@<instrumento>:<corrida>:<identificador de celda de flujo>:<línea>:<cuadro>:<x-coord>:<y-coord> <lectura>:<si fue filtrada>:<número de control>:<índice de la secuencia> length=<tamaño de la secuencia>

```

Al nivel de calidad representado en la cuarta línea se le denomina Phred score. En su formato original un Phred score es simplemente el logaritmo de las probabilidades de error:

```
Phred score = - 10 * log10(probabilidad de error)
```


Puntaje de calidad (quality score)	Probabilidad de error
Q40 0.0001	(1 en 10,000)
Q30 0.001	(1 en 1,000)
Q20 0.01	(1 en 100)
Q10 0.1	(1 en 10)

Espera, si los puntajes de Phred son números, ¿por qué los puntajes de calidad mostrados en la línea cuatro son una mezcla de símbolos alfanuméricos?

```
#1:=BDDDHDHB?<?CFGGGC9@FF@GGGG>EEEDGDHGFHGE;AEFH>AC@D;@B>C>CCC@C>>DDCC3:>AA5>CC>>CCD>@CCDCDCCCCC@C@>C
```

Esto es debido a que estos puntajes están codificados en ASCII (American Standard Code for Informational Interchange). En breve, ASCII es un sistema de codificación que equipara un número a un caracter alfanumérico. Por ejemplo, el carácter ‘A’ se representa por el número 65 en la tabla de código ASCII, mientras que ‘%’ se representa por el número 37. Este sistema nos permite representar así un total de 256 caracteres distintos.

¿Por qué usar ASCII? Dada la enorme cantidad de datos producidos durante secuenciación masiva, se trata de reducir los datos al máximo. El sistema ASCII nos permite representar números de dos dígitos en un solo bit (8 bites), lo cual reduce el espacio que ocupan estos archivos. Si te interesa el tema puedes leer más acerca de codificación binaria.

Finalmente, desde que se inventó este sistema de codificación de calidad, el cuál ya se utilizaba con la secuenciación tipo Sanger, distintas compañías han “deslizado” las escalas de conversión de ASCII porque lo que es importante verificar con la compañía o laboratorio en que versión de esta escala se han codificado para usar la conversión correcta. Algunas de las herramientas de control de calidad que utilizaremos infieren el tipo de codificación utilizado a partir de los datos.

## Verificando la calidad de las secuencias

Verificaremos la calidad de las secuencias usando el programa FastQC. Este es uno de los programa más utilizados para este tipo de análisis, ya que provee una visión global de como se estructuran los datos, así como ser bastante rápido.

FastQC lee una serie de archivos de secuenciación y produce un reporte de control de calidad para cada uno, el cual consiste de un número de módulos diferentes, cada uno de los cuales nos ayuda a identificar distintos problemas en nuestros datos.

Analicemos nuestro primer archivo de secuenciación, primero creamos un directorio para almacenar nuestros resultados:

```sh
mkdir QUAL
```
Y realizamos nuestro análisis de calidad usando FastQC:
```sh
fastqc -O ./QUAL/ Partial_SRR2467141.fastq
```
```
Started analysis of Partial_SRR2467141.fastq
Approx 20% complete for Partial_SRR2467141.fastq
Approx 40% complete for Partial_SRR2467141.fastq
Approx 60% complete for Partial_SRR2467141.fastq
Approx 80% complete for Partial_SRR2467141.fastq
Analysis complete for Partial_SRR2467141.fastq
```

Una vez terminado revisemos el resultado:
```sh
cd QUAL
ls
```
```
Partial_SRR2467141_fastqc.html
Partial_SRR2467141_fastqc.zip
```

------

# Tutorial de Bioinformática

## Exploración de Resultados

La manera más sencilla de explorar estos resultados es abriendo el archivo HTML en su navegador. Lo puedes hacer dando doble clic en el archivo.

Para hacerlo deberás descargar el archivo a tu computadora desde el contenedor de Docker. Para esto, primero debemos abrir otra terminal para no cerrar nuestra imagen de Docker. En esta nueva terminal averiguamos el ID de nuestro contenedor usando el comando:

```bash
$ docker ps
```

```plaintext
CONTAINER ID        IMAGE                           COMMAND             CREATED             STATUS              PORTS               NAMES
89510b0fd8b6        pbi_cinvestav/transcriptomics   "/bin/bash"         About an hour ago   Up About an hour                        sad_stonebraker
```

Lo que necesitamos es el número listado bajo “CONTAINER ID” y usaremos el comando `docker cp`, reemplazando el número de contenedor y el path con los correspondientes a tu computadora:

```bash
 docker cp 89510b0fd8b6:/usr/local/ANALYSIS/FastQC_Short/QUAL/Partial_SRR2467141_fastqc.html <my-computer-path>
```

Tip: Lo más sencillo es entrar vía la terminal al directorio de trabajo en tu computadora y reemplazar la ruta (path) con un punto.

Una vez descargado, podrás dar doble clic en el archivo. El resultado obtenido deberá ser similar a este.

## Secciones del Reporte

Esta página contiene mucha información desglosada en las siguientes secciones:

- **Basic Statistics**: Las estadísticas básicas del experimento.
- **Per base sequence quality**: Diagramas de caja mostrando la calidad de cada base.
- **Per tile sequence quality**: Contiene el cuadro (tile) del que proviene cada secuencia. Solo aparece si los análisis se realizan en una librería de Illumina que retiene sus identificadores originales.
- **Per sequence quality scores**: Permite ver si un subgrupo de secuencias tiene mala calidad.
- **Per base sequence content**: Muestra la proporción de cada base en cada posición.
- **Per sequence GC content**: Compara la distribución de GC observada con una distribución modelada de GC.
- **Per base N content**: Indica cuántos nucleótidos no pudieron ser interpretados y se indican con una N.
- **Sequence Length Distribution**: Muestra la distribución de la longitud de secuencias.
- **Sequence Duplication Levels**: Indica cuántas veces se repite cada secuencia. Se realiza solo en las primeras 100,000 secuencias.
- **Overrepresented sequences**: Muestra secuencias sobre representadas.
- **Adapter Content**: Muestra el contenido de adaptadores en la librería.
- **Kmer Content**: Muestra la distribución de kmers (subcadenas de tamaño específico) en cada posición de las lecturas. Solo se realiza en el 2% de la librería.

Revisemos los resultados una vez más dando clic en el link.

En general, vemos que la mayoría de los criterios tienen una palomita verde o certificado de pase de control de calidad, con la excepción del contenido de GC por base.

A pesar de que no proporcionamos esta información, el programa ha inferido la codificación ASCII (Sanger / Illumina 1.9) así como contado el número de secuencias, su tamaño y su contenido de GC.

## Calidad de las Bases

Cuando observamos la calidad por base, vemos que la mayoría de las bases están en la zona verde o tienen buena calidad. Cada base está representada por un diagrama de caja que proporciona una buena idea acerca de la distribución de los puntajes de calidad en todas las secuencias. Es evidente que la calidad decrece levemente al inicio y más marcadamente al final de la secuencia. Esto es conocido ya que pequeños errores se acumulan mientras más larga sea la secuencia, esta es una de las razones por la que los secuenciadores basados en incorporación de nucleótidos tienen un límite en la longitud máxima de sus lecturas.

## Otros Resultados del Reporte

- La gráfica mostrando la calidad por cuadro está completamente azul mostrando que la distribución promedio de errores en el cuadro (tile) es muy similar.
- La gráfica de puntaje de calidad por secuencia muestra que la mayoría de las secuencias tienen puntajes altos.
- La distribución de nucleótidos por base muestra que, al principio de la secuencia, la distribución es muy disparatada. Esto es uno de los errores comunes en RNA-Seq hecho por Illumina. Generalmente se soluciona cortando algunos de los nucleótidos en el 5’.

También observamos que:

- La distribución de GC por secuencia es muy similar a la distribución hipotética.
- Hay pocas bases ambiguas (Ns).
- Todas las secuencias tienen el mismo tamaño.
- Los niveles de duplicación de secuencias son bajos.
- No hay secuencias sobre representadas.
- Hay pocos adaptadores.
- No hay kmers sobre representados.

## Archivo Comprimido de Resultados

Al ejecutar FastQC también se generó un archivo comprimido llamado `Partial_SRR2467141_fastqc.zip`. Dentro del contenedor de Docker, desempacamos este archivo usando el comando:

```bash
$ unzip Partial_SRR2467141_fastqc.zip
```

```plaintext
Archive:  Partial_SRR2467141_fastqc.zip
   creating: Partial_SRR2467141_fastqc/
   creating: Partial_SRR2467141_fastqc/Icons/
   creating: Partial_SRR2467141_fastqc/Images/
  inflating: Partial_SRR2467141_fastqc/Icons/fastqc_icon.png
  inflating: Partial_SRR2467141_fastqc/Icons/warning.png
  inflating: Partial_SRR2467141_fastqc/Icons/error.png
  inflating: Partial_SRR2467141_fastqc/Icons/tick.png
  inflating: Partial_SRR2467141_fastqc/summary.txt
  inflating: Partial_SRR2467141_fastqc/Images/per_base_quality.png
  inflating: Partial_SRR2467141_fastqc/Images/per_tile_quality.png
  inflating: Partial_SRR2467141_fastqc/Images/per_sequence_quality.png
  inflating: Partial_SRR2467141_fastqc/Images/per_base_sequence_content.png
  inflating: Partial_SRR2467141_fastqc/Images/per_sequence_gc_content.png
  inflating: Partial_SRR2467141_fastqc/Images/per_base_n_content.png
  inflating: Partial_SRR2467141_fastqc/Images/sequence_length_distribution.png
  inflating: Partial_SRR2467141_fastqc/Images/duplication_levels.png
  inflating: Partial_SRR2467141_fastqc/Images/adapter_content.png
  inflating: Partial_SRR2467141_fastqc/fastqc_report.html
  inflating: Partial_SRR2467141_fastqc/fastqc_data.txt
  inflating: Partial_SRR2467141_fastqc/fastqc.fo
```

Si entramos al directorio creado, podemos ver el reporte en formato de texto plano, tanto el resumen:

```bash
cd Partial_SRR2467141_fastqc
more summary.txt
```

```plaintext
PASS    Basic Statistics        Partial_SRR2467141.fastq
PASS    Per base sequence quality       Partial_SRR2467141.fastq
PASS    Per tile sequence quality       Partial_SRR2467141.fastq
PASS    Per sequence quality scores     Partial_SRR2467141.fastq
FAIL    Per base sequence content       Partial_SRR2467141.fastq
PASS    Per sequence GC content Partial_SRR2467141.fastq
PASS    Per base N content      Partial_SRR2467141.fastq
PASS    Sequence Length Distribution    Partial_SRR2467141.fastq
PASS    Sequence Duplication Levels     Partial_SRR2467141.fastq
PASS    Overrepresented sequences       Partial_SRR2467141.fastq
PASS    Adapter Content Partial_SRR2467141.fastq
PASS    Kmer Content    Partial_SRR2467141.fastq
```

Como el archivo completo:

```bash
$ more fastqc_data.txt
```

Hemos omitido el contenido ya que es muy largo. El poder revisar los resultados en archivos planos es extremadamente útil cuando los resultados se encuentren en un servidor que no cuente con interfaz gráfica.

## Ejemplos de Datos de Secuenciación

Los programadores de FastQC han compilado ejemplos de datos de secuenciación de calidad diversa. Analicémoslos:

- Buenos datos - Illumina
- Malos datos - Illumina
- Contaminación por dímeros
- RNAs pequeños con lecturas a través del adaptador
- Datos PacBio
- Datos 454



## ¿Qué tan válido es nuestro análisis?


Dado que los datos que descargamos solo tienen 5000 secuencias, ¿podemos confiar en que los datos completos tendrán calidad similar? ¿por qué si o por qué no?

----------------



# Limpiando las secuencias

Existen un número de herramientas para limpiar secuencias y distintos investigadores tienen distintas preferencias. Cómo con otros problemas de secuenciación existe un desarrollo activo de programas con este fin.

Nosotros utilizaremos un programa llamado Trimmomatic. Este es un programa basado en Java lo que, al igual que FastQC, lo hace muy rápido. Además cuenta con modos de filtrado para lecturas en conformación individual o en pares (paired-end).

El programa realiza diversas funciones, incluyendo:

- Remoción de adaptadores
- Remoción de bases de baja calidad en el 5’ o 3’ de la lectura
- Escaneo de la secuencia a través de una ventana de 4 nucleótidos, cortando cuando la calidad promedio de la base baja de 15.
- Eliminar secuencias más cortas que un tamaño particular
- Cortar un número específico de nucleótidos en el extremo 5’ o 3’
- Conversión de puntajes de calidad entre distintas versiones de Phred.

Las funciones específicas de acuerdo al manual son:

- **ILLUMINACLIP**: Cut adapter and other illumina-specific sequences from the read.
- **SLIDINGWINDOW**: Perform a sliding window trimming, cutting once the average quality within the window falls below a threshold.
- **LEADING**: Cut bases off the start of a read, if below a threshold quality.
- **TRAILING**: Cut bases off the end of a read, if below a threshold quality.
- **CROP**: Cut the read to a specified length.
- **HEADCROP**: Cut the specified number of bases from the start of the read.
- **MINLEN**: Drop the read if it is below a specified length.
- **TOPHRED33**: Convert quality scores to Phred-33.
- **TOPHRED64**: Convert quality scores to Phred-64.

La configuración cambia dependiendo si las secuencias son single end o paired-end.

Pueden encontrar descripciones más detalladas de sus funciones en el manual en línea.

## Veamos cómo funciona El software ya está instalado en el contenedor.



Regresemos a la carpeta donde están nuestros archivos fastq (`/usr/local/ANALYSIS/FastQC_Short`).

```bash
 cd ../..
```

La calidad de nuestras secuencias de prueba es bastante buena, pero vimos que la calidad baja en el extremo 3’, así como la presencia desproporcionada de bases en el extremo 5’. Limpiemos nuestras secuencias usando Trimmomatic.

Pediremos que corte:

- Los adaptadores Illumina TruSeq single end (SE) del extremo 3’
- Los primeros 10 nucleótidos del 5’
- Bases de mala calidad usando una ventana de 4 nucleótidos con un promedio mínimo de Phred de 15
- Secuencias menores a 60 nucleótidos.

```bash
 trimmomatic SE Partial_SRR2467141.fastq Trimmed_Partial_SRR2467141.fastq ILLUMINACLIP:TruSeq3-SE.fa:2:30:10 HEADCROP:10 SLIDINGWINDOW:4:15 MINLEN:60
```
```plaintext
TrimmomaticSE: Started with arguments:
 Partial_SRR2467141.fastq Trimmed_Partial_SRR2467141.fastq ILLUMINACLIP:TruSeq3-SE.fa:2:30:10 HEADCROP:10 SLIDINGWINDOW:4:15 MINLEN:60
Automatically using 2 threads


java.io.FileNotFoundException: /usr/local/ANALYSIS/FastQC_Short/TruSeq3-SE.fa (No such file or directory)
    at java.base/java.io.FileInputStream.open0(Native Method)
    at java.base/java.io.FileInputStream.open(FileInputStream.java:219)
    at java.base/java.io.FileInputStream.<init>(FileInputStream.java:157)
    at org.usadellab.trimmomatic.fasta.FastaParser.parse(FastaParser.java:54)
    at org.usadellab.trimmomatic.trim.IlluminaClippingTrimmer.loadSequences(IlluminaClippingTrimmer.java:110)
    at org.usadellab.trimmomatic.trim.IlluminaClippingTrimmer.makeIlluminaClippingTrimmer(IlluminaClippingTrimmer.java:71)
    at org.usadellab.trimmomatic.trim.TrimmerFactory.makeTrimmer(TrimmerFactory.java:32)
    at org.usadellab.trimmomatic.Trimmomatic.createTrimmers(Trimmomatic.java:59)
    at org.usadellab.trimmomatic.TrimmomaticSE.run(TrimmomaticSE.java:318)
    at org.usadellab.trimmomatic.Trimmomatic.main(Trimmomatic.java:85)


Quality encoding detected as phred33
Input Reads: 5000 Surviving: 4856 (97.12%) Dropped: 144 (2.88%)
TrimmomaticSE: Completed successfully
```
Podemos ver que hubo muy pocas secuencias que fueron descartadas pero aún así son cosas que podrían haber afectado nuestro análisis y/o ensamble. Vemos además que hay un error:

```plaintext
java.io.FileNotFoundException: /usr/local/ANALYSIS/FastQC_Short/TruSeq3-SE.fa (No such file or directory)
```

Esto se debe a que las secuencias de los adaptadores no se encuentran en el mismo directorio que nuestros datos. Una manera de compensar esto es descargar los archivos necesarios (estos archivos se proveen con Trimmomatic pero los descargaremos dado que estamos trabajando en el servidor).

Descarguemos este archivo y ejecutemos nuevamente Trimmomatic:

```bash
 wget https://raw.githubusercontent.com/timflutre/trimmomatic/master/adapters/TruSeq3-SE.fa
 trimmomatic SE Partial_SRR2467141.fastq Trimmed_Partial_SRR2467141.fastq ILLUMINACLIP:TruSeq3-SE.fa:2:30:10 HEADCROP:10 SLIDINGWINDOW:4:15 MINLEN:60
```
```plaintext
TrimmomaticSE: Started with arguments:
 Partial_SRR2467141.fastq Trimmed_Partial_SRR2467141.fastq ILLUMINACLIP:TruSeq3-SE.fa:2:30:10 HEADCROP:10 SLIDINGWINDOW:4:15 MINLEN:60
Automatically using 2 threads

Using Long Clipping Sequence: 'AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTA'
Using Long Clipping Sequence: 'AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC'
ILLUMINACLIP: Using 0 prefix pairs, 2 forward/reverse sequences, 0 forward only sequences, 0 reverse only sequences
Quality encoding detected as phred33
Input Reads: 5000 Surviving: 4847 (96.94%) Dropped: 153 (3.06%)
TrimmomaticSE: Completed successfully
```

Este resultado nos muestra que el programa se ejecutó adecuadamente y eliminó secuencias que contienen los adaptadores que indicamos. Comparando con nuestros resultados previos vemos que encontró 9 secuencias con estos adaptadores.

Si usamos el comando `ls` podemos ver que se generó un archivo de resultados llamado `Trimmed_Partial_SRR2467141.fastq`.

```bash
 ls
Partial_SRR2467141.fastq
Partial_SRR2467142.fastq
Partial_SRR2467143.fastq
Partial_SRR2467144.fastq
Partial_SRR2467145.fastq
Partial_SRR2467146.fastq
Partial_SRR2467147.fastq
Partial_SRR2467148.fastq
Partial_SRR2467149.fastq
Partial_SRR2467150.fastq
Partial_SRR2467151.fastq
QUAL
**Trimmed_Partial_SRR2467141.fastq**
TruSeq3-SE.fa
```

## Problemas de memoria

En cuanto empieces a utilizar Java para análisis de secuenciación frecuentemente te encontrarás con problemas de memoria. Estos se resuelven especificando exactamente cuánta memoria le permites asignar a Java usando las banderas:

- `-Xms<size>`        set initial Java heap size
- `-Xmx<size>`        set maximum Java heap size

Solo debes asegurarte que la memoria que pides es congruente con los recursos disponibles en tu computadora o servidor.

Revisemos ahora los resultados de nuestra limpieza de secuencias. Primero veremos las primeras líneas del archivo limpio.

```bash
 head -n 12 Trimmed_Partial_SRR2467141.fastq
```

```plaintext
@SRR2467141.1 SALLY:472:C6NCDACXX:8:1101:1392:1873 length=101
TCGTTCTTCTTGAAGAAGACGTTACCTACGGCGTATTTGCCCATCTCAGGTATGTCTAGATCAAGATCTAACTTGAATTCTCTTTTCATAA
+SRR2467141.1 SALLY:472:C6NCDACXX:8:1101:1392:1873 length=101
HB?<?CFGGGC9@FF@GGGG>EEEDGDHGFHGE;AEFH>AC@D;@B>C>CCC@C>>DDCC3:>AA5>CC>>CCD>@

CCDCDCCCCC@C@>C
@SRR2467141.2 SALLY:472:C6NCDACXX:8:1101:1326:1950 length=101
CTGGCCAAATGCCCCATTATTTTGAGTATTGTTATTTCCAAATAAACTGTTACTATTACTGCCAGCGGCAGAAGTGAATCCACAGATC
+SRR2467141.2 SALLY:472:C6NCDACXX:8:1101:1326:1950 length=101
FH?GEHEHIDEHCCFEHIE@GIGCHG<DGGIHIGIGGIGHIIFIHFFGDHGGIIHIGIIIIGGEH@EADB>=AA>3@>CCCCCCBC@C
@SRR2467141.3 SALLY:472:C6NCDACXX:8:1101:1477:1959 length=101
TTAGGCACATCATCCTGATAAGTGTACTTACCAGGATATATACCATCGGTATTGATGTTATCGGCATCACATAAAACTAATTCACCAGAAA
+SRR2467141.3 SALLY:472:C6NCDACXX:8:1101:1477:1959 length=101
HHHJJJIIJJIJJJJJJEIJJJIIJJJIJIJJIJJIIIJJJIIIHIIIJDHIJJIGJJIJJJJJIHHHFFFFFEEEEEEDDEDDDDDDDDD
```

Podemos ver que hasta en las primeras líneas se efectuaron cortes, en su mayoría en el extremo 3’.

Revisemos ahora los resultados globales usando FastQC:

```bash
$ fastqc -O ./QUAL/ Trimmed_Partial_SRR2467141.fastq
```

```plaintext
Started analysis of Trimmed_Partial_SRR2467141.fastq
Approx 20% complete for Trimmed_Partial_SRR2467141.fastq
Approx 40% complete for Trimmed_Partial_SRR2467141.fastq
Approx 60% complete for Trimmed_Partial_SRR2467141.fastq
Approx 80% complete for Trimmed_Partial_SRR2467141.fastq
Analysis complete for Trimmed_Partial_SRR2467141.fastq
```

Abrimos el resultado html `Trimmed_Partial_SRR2467141_fastqc.html`.

### ¿Qué diferencias hay entre las secuencias crudas y limpias?

Compara los archivos html entre las secuencias crudas y limpias. Resume: ¿Qué diferencias existen? ¿Cuál crees que sea la causa de esas diferencias?

Existen otras herramientas como fastx-toolkit, scythe y sickle que realizan procesos similares. Estas herramientas están en su versión de Biolinux si las quieren comparar.

## Guardando la imagen modificada

Nos podemos salir de docker o hacer esto antes, en otra terminal usamos el siguiente comando para ver qué contenedores se están ejecutando:

```bash
 docker ps -a
```

```plaintext
CONTAINER ID        IMAGE                              COMMAND                  CREATED              STATUS                       PORTS               NAMES
77b7aa677672        joseancorona/curso_transcriptomica:base      "/bin/bash"              About a minute ago   Exited (0) 53 seconds ago                        happy_raman
```

El más reciente es el que están utilizando, el `CONTAINER ID` debe ser el mismo que ven en su prompt del lado izquierdo.

Usamos este identificador para guardar nuestro trabajo:

```bash
 docker commit 77b7aa677672 joseancorona/curso_transcriptomica:practica
```

## Tarea - Análisis de calidad

Realiza los análisis de calidad para todas las muestras, así como el trimming. Realiza los análisis de FastQC correspondientes. Si tienes tiempo, intenta utilizar MultiQC para visualizar tus resultados, ya está instalado en la imagen.
