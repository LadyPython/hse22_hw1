# hse22_hw1

* Создаем папочку для дз:
```bash
mkdir hw1
cd hw1
```

* Создаем симлинки на файлы:
```bash
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```

* Случайно выбираем 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs:
```bash
seqtk sample -s928 oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s928 oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s928 oilMP_S4_L001_R1_001.fastq 1500000 > subMP1.fastq
seqtk sample -s928 oilMP_S4_L001_R2_001.fastq 1500000 > subMP2.fastq
```

* Оцениваем качество исходных чтений с помощью `fastqc` и получаем статистику с помощью `multiqc`:
```bash
mkdir fastqc_result
fastqc sub1.fastq sub2.fastq subMP1.fastq subMP2.fastq -o fastqc_result
multiqc fastqc_result -o multiqc_result
```
_Статистику можно найти в файле [multiqc_report.html](multiqc_report.html)._

* Подрезаем чтения по качеству и удаляем адаптеры:

```bash
platanus_trim sub1.fastq sub2.fastq
```

```commandline
Number of trimmed read with adapter: 
NUM_OF_TRIMMED_READ(FORWARD) = 209171
NUM_OF_TRIMMED_BASE(FORWARD) = 206986
NUM_OF_TRIMMED_READ(REVERSE) = 209215
NUM_OF_TRIMMED_BASE(REVERSE) = 353663
NUM_OF_TRIMMED_PAIR(OR) = 209239
NUM_OF_TRIMMED_PAIR(AND) = 209147


Number of trimmed read because of low quality or too short (< 11bp): 
NUM_OF_TRIMMED_READ(FORWARD) = 908564
NUM_OF_TRIMMED_BASE(FORWARD) = 18125009
NUM_OF_TRIMMED_READ(REVERSE) = 1179702
NUM_OF_TRIMMED_BASE(REVERSE) = 35925471
NUM_OF_TRIMMED_PAIR(OR) = 1631218
NUM_OF_TRIMMED_PAIR(AND) = 457048
```

```bash
platanus_internal_trim subMP1.fastq subMP2.fastq
```

```commandline
Number of trimmed read with internal adapter: 
NUM_OF_TRIMMED_READ(FORWARD) = 971667
NUM_OF_TRIMMED_BASE(FORWARD) = 169530267
NUM_OF_TRIMMED_READ(REVERSE) = 956532
NUM_OF_TRIMMED_BASE(REVERSE) = 171440124
NUM_OF_TRIMMED_PAIR(OR) = 1187764
NUM_OF_TRIMMED_PAIR(AND) = 740435


Number of trimmed read with adapter: 
NUM_OF_TRIMMED_READ(FORWARD) = 11145
NUM_OF_TRIMMED_BASE(FORWARD) = 366348
NUM_OF_TRIMMED_READ(REVERSE) = 11166
NUM_OF_TRIMMED_BASE(REVERSE) = 389195
NUM_OF_TRIMMED_PAIR(OR) = 11172
NUM_OF_TRIMMED_PAIR(AND) = 11139


Number of trimmed read because of low quality or too short (< 11bp): 
NUM_OF_TRIMMED_READ(FORWARD) = 360729
NUM_OF_TRIMMED_BASE(FORWARD) = 11667281
NUM_OF_TRIMMED_READ(REVERSE) = 468276
NUM_OF_TRIMMED_BASE(REVERSE) = 23883600
NUM_OF_TRIMMED_PAIR(OR) = 724081
NUM_OF_TRIMMED_PAIR(AND) = 104924
```

* Оцениваем качество подрезанных чтений с помощью `fastqc` и получаем статистику с помощью `multiqc`:
```bash
mkdir fastqc_trimmed_result
fastqc sub1.fastq.trimmed sub2.fastq.trimmed subMP1.fastq.int_trimmed subMP2.fastq.int_trimmed -o fastqc_trimmed_result
multiqc fastqc_result -o multiqc_trimmed_result
```
_Статистику можно найти в файле [multiqc_trimmed_report.html](multiqc_trimmed_report.html)._

* Собираем **контиги** из подрезанных чтений с помощью `platanus assemble`:
```bash
platanus assemble -f sub1.fastq.trimmed sub2.fastq.trimmed -o sub
```

* Собираем **скаффолды** из контигов и подрезанных чтений с помощью `platanus scaffold`:
```bash
platanus scaffold -c sub_contig.fa -b sub_contigBubble.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 subMP1.fastq.int_trimmed subMP2.fastq.int_trimmed -o sub
```

* Уменьшаем количество гэпов с помощью `platanus gap_close`:
```bash
platanus gap_close -c sub_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 subMP1.fastq.int_trimmed subMP2.fastq.int_trimmed -o sub
```

___

## Бонус
```bash
seqtk sample -s928 oil_R1.fastq 500000 > bonus_sub1.fastq
seqtk sample -s928 oil_R2.fastq 500000 > bonus_sub2.fastq
seqtk sample -s928 oilMP_S4_L001_R1_001.fastq 150000 > bonus_subMP1.fastq
seqtk sample -s928 oilMP_S4_L001_R2_001.fastq 150000 > bonus_subMP2.fastq
mkdir bonus_fastqc_result
fastqc bonus_sub1.fastq bonus_sub2.fastq bonus_subMP1.fastq bonus_subMP2.fastq -o bonus_fastqc_result
multiqc bonus_fastqc_result -o bonus_multiqc_result
platanus_trim bonus_sub1.fastq bonus_sub2.fastq
platanus_internal_trim bonus_subMP1.fastq bonus_subMP2.fastq
mkdir bonus_fastqc_trimmed_result
fastqc bonus_sub1.fastq.trimmed bonus_sub2.fastq.trimmed bonus_subMP1.fastq.int_trimmed bonus_subMP2.fastq.int_trimmed -o bonus_fastqc_trimmed_result
multiqc bonus_fastqc_result -o bonus_multiqc_trimmed_result
platanus assemble -f bonus_sub1.fastq.trimmed bonus_sub2.fastq.trimmed -o bonus_sub
platanus scaffold -c bonus_sub_contig.fa -b bonus_sub_contigBubble.fa -IP1 bonus_sub1.fastq.trimmed bonus_sub2.fastq.trimmed -OP2 bonus_subMP1.fastq.int_trimmed bonus_subMP2.fastq.int_trimmed -o bonus_sub
platanus gap_close -c bonus_sub_scaffold.fa -IP1 bonus_sub1.fastq.trimmed bonus_sub2.fastq.trimmed -OP2 bonus_subMP1.fastq.int_trimmed bonus_subMP2.fastq.int_trimmed -o bonus_sub
```