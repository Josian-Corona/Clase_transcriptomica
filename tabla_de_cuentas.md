
# Análisis transcriptómicos

## Análisis de expresión diferencial

### Objetivos de aprendizaje
- Aprender a realizar análisis de expresión diferencial usando DESeq2

### Datos requeridos
Para proceder con esta práctica, se requieren:
- El genoma `Sp_genome.fa`
- Las lecturas no filtradas

Utilizaremos los archivos bam filtrados por calidad que utilizamos en la clase de mapeo. Los análisis se realizarán en la plataforma R.

### Generación de directorio de trabajo
```bash
mkdir DESEQ2
cd DESEQ2/
```

### Contando lecturas
El primer paso es contar lecturas. Tanto DESeq2 como otros programas que usan la distribución negativa binomial utilizan conteos crudos (raw). Esto se refiere al número de lecturas que alinean a un transcrito específico sin sufrir ningún tipo de transformación. Como vieron en la clase pasada, esto se debe a que el programa estima un factor de normalización. Si usamos datos previamente normalizados, estaremos violando los supuestos.

Descargaremos la anotación de los genes de *Schizosaccharomyces pombe* desde aquí:
```bash
cp /usr/local/data/Sp_genes.gtf .
```
Usaremos el programa HTSeq. Ya se encuentra instalado en Docker.

Este programa cuenta el número de lecturas que alinean con cada gen anotado en un archivo de referencia. Nosotros le proporcionaremos nuestro archivo `Sp_genes.gtf`.

```bash
htseq-count -f bam -s no -r pos Sp_ds_sorted.bam /usr/local/data/Sp_genes.gtf > Sp_ds.counts 
htseq-count -f bam -s no -r pos Sp_hs_sorted.bam /usr/local/data/Sp_genes.gtf > Sp_hs.counts 
htseq-count -f bam -s no -r pos Sp_log_sorted.bam /usr/local/data/Sp_genes.gtf > Sp_log.counts 
htseq-count -f bam -s no -r pos Sp_plat_sorted.bam /usr/local/data/Sp_genes.gtf > Sp_plat.counts 
```

Veamos uno de los archivos de cuentas, primero las primeras 10 líneas:
```bash
head Sp_ds.counts
```
```
SPAC1002.19    23
SPAC1093.06c   48
SPAC10F6.01c   362
SPAC10F6.05c   91
SPAC11D3.18c   25
SPAC11E3.06    28
SPAC11E3.14    399
SPAC11H11.04   16
SPAC1250.07    71
SPAC12B10.14c  70
```

Y las últimas 10 líneas:
```bash
tail Sp_ds.counts
```
```
SPCC736.05                18
SPCC736.12c               57
SPCC757.03c               1064
SPCC794.10                68
SPCP25A2.02c              49
__no_feature              0
__ambiguous               2
__too_low_aQual           5483
__not_aligned             1579
__alignment_not_unique    3
```

Estas líneas al final del ejemplo muestran lecturas que:
- **No feature**: Lecturas que no alinean a una región anotada (gen)
- **Ambiguous**: Lecturas que alinean a más de un gen
- **Too low alignment quality**: Lecturas con baja calidad de alineamiento
- **Not aligned**: Lecturas no alineadas
- **Alignment not unique**: Lecturas que alinean más de una vez a la referencia

### Contando librerías de manera independiente
¿Por qué contamos las lecturas en cada librería de manera independiente si ensamblamos el transcriptoma de referencia usando todas las librerías (muestras)?

Es importante verificar que el conteo de HTSeq engloba a la mayoría de las lecturas. Si estos porcentajes son muy bajos, podrían indicar problemas con los parámetros de conteo (e.g. si le decimos que nuestra muestra no tiene direccionalidad (strandness) cuando sí la tiene).

Para realizar nuestros análisis de expresión debemos eliminar estas líneas. Uniremos los resultados usando el siguiente comando:
```bash
paste *counts | cut -f 1,2,4,6,8 | grep '__' -v > Sp_counts_table.txt
head Sp_counts_table.txt
```

Esto unió los distintos archivos en uno solo el cual ya podemos utilizar en R. Es importante verificar el orden de pegado de los archivos para no confundir una muestra con otra.

```plaintext
SPAC1002.19    23    44    2    230
SPAC1093.06c   48    22    34   59
SPAC10F6.01c   362   323   527  200
SPAC10F6.05c   91    149   54   203
SPAC11D3.18c   25    11    12   23
SPAC11E3.06    28    8     5    4
SPAC11E3.14    399   148   53   387
SPAC11H11.04   16    6     12   23
SPAC1250.07    71    36    29   72
SPAC12B10.14c  70    37    52   47
```
