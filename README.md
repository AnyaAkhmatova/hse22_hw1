# hse22_hw1

# Майнор "Биоинформатика". 2 год, 1 семестр.
## Домашнее задание 1.

### Список команд, которые были выполнены на сервере:

Создаем директорию для ДЗ1:
```
mkdir hw1
cd hw1
```

Cоздаем символические ссылки на общие файлы:
```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```

Выбираем случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs(код 927):
```
seqtk sample -s927 oil_R1.fastq 5000000 > sample_pair_end_oil_R1.fastq
seqtk sample -s927 oil_R2.fastq 5000000 > sample_pair_end_oil_R2.fastq
seqtk sample -s927 oilMP_S4_L001_R1_001.fastq 1500000 > sample_mate_pairs_oilMP_S4_L001_R1_001.fastq
seqtk sample -s927 oilMP_S4_L001_R2_001.fastq 1500000 > sample_mate_pairs_oilMP_S4_L001_R2_001.fastq
```

С помощью программ fastQC и multiQC оцениваем качество исходных чтений и получаем по ним общую статистику:
```
mkdir fastqc
fastqc -o fastqc sample_pair_end_oil_R1.fastq sample_pair_end_oil_R2.fastq sample_mate_pairs_oilMP_S4_L001_R1_001.fastq sample_mate_pairs_oilMP_S4_L001_R2_001.fastq
mkdir multiqc
multiqc -o multiqc fastqc
```

Подрезаем чтения по качеству и удаляем адаптеры:
```
platanus_trim sample_pair_end_oil_R1.fastq sample_pair_end_oil_R2.fastq
platanus_internal_trim sample_mate_pairs_oilMP_S4_L001_R1_001.fastq sample_mate_pairs_oilMP_S4_L001_R2_001.fastq
```

Удаляем исходные .fastq файлы, полученные с помощью программы seqtk:
```
rm -rf sample_pair_end_oil_R1.fastq sample_pair_end_oil_R2.fastq sample_mate_pairs_oilMP_S4_L001_R1_001.fastq sample_mate_pairs_oilMP_S4_L001_R2_001.fastq
```

С помощью программ fastQC и multiQC оцениваем качество подрезанных чтений и получаем по ним общую статистику:
```
mkdir fastqc_trimmed
fastqc -o fastqc_trimmed sample_pair_end_oil_R1.fastq.trimmed sample_pair_end_oil_R2.fastq.trimmed sample_mate_pairs_oilMP_S4_L001_R1_001.fastq.int_trimmed sample_mate_pairs_oilMP_S4_L001_R2_001.fastq.int_trimmed
mkdir multiqc_trimmed
multiqc -o multiqc_trimmed fastqc_trimmed
```

Собираем контиги из подрезанных чтений:
```
platanus assemble –o Oil –f sample_pair_end_oil_R1.fastq.trimmed sample_pair_end_oil_R2.fastq.trimmed 2> assemble.log
```

Cобираем скаффолды из контигов, а также из подрезанных чтений:
```
platanus scaffold –o Oil –c Oil_contig.fa -IP1 sample_pair_end_oil_R1.fastq.trimmed sample_pair_end_oil_R2.fastq.trimmed -OP2 sample_mate_pairs_oilMP_S4_L001_R1_001.fastq.int_trimmed sample_mate_pairs_oilMP_S4_L001_R2_001.fastq.int_trimmed 2> scaffold.log
```

Уменьшаем количетво гэпов с помощью подрезанных чтений:
```
platanus gap_close –o Oil –c Oil_scaffold.fa -IP1 sample_pair_end_oil_R1.fastq.trimmed sample_pair_end_oil_R2.fastq.trimmed -OP2 sample_mate_pairs_oilMP_S4_L001_R1_001.fastq.int_trimmed sample_mate_pairs_oilMP_S4_L001_R2_001.fastq.int_trimmed 2> gapclose.log
```
 
Удаляем  .fastq файлы, содержащие подрезанные чтения:
```
rm -rf sample_pair_end_oil_R1.fastq.trimmed sample_pair_end_oil_R2.fastq.trimmed sample_mate_pairs_oilMP_S4_L001_R1_001.fastq.int_trimmed sample_mate_pairs_oilMP_S4_L001_R2_001.fastq.int_trimmed
```

____

### Скриншоты и статистика из файлов multiQC:

[Исходные чтения](html_reports/multiqc_report.html)

![multiqc1](screenshots/multiqc1.jpg)
![multiqc2](screenshots/multiqc2.jpg)
![multiqc3](screenshots/multiqc3.jpg)

[Подрезанные чтения](html_reports/multiqc_trimmed_report.html)

![multiqc_trimmed1](screenshots/multiqc_trimmed1.jpg)
![multiqc_trimmed2](screenshots/multiqc_trimmed2.jpg)
![multiqc_trimmed3](screenshots/multiqc_trimmed3.jpg)

____

### Ссылки на jupiter ноутбуки:

В [jupiter-ноутбуке](src/analysis_of_contigs_and_scaffolds.ipynb) содержатся
- анализ полученных контигов (общее количество контигов, их общая длина, длина самого длинного контига, N50)
- анализ полученных скаффолдов (общее кол-во скаффолдов, их общая длина, длина самого длинного скаффолда, N50)
- данные по количеству гэпов и их общей длине для самого длинного скаффолда
- анализа полученных скаффолдов после gap_close (общее кол-во скаффолдов, их общая длина, длина самого длинного скаффолда, N50)
- данные по количеству гэпов и их общей длине для самого длинного скаффолда после gap_close


**Результаты:**


**1. Анализ контигов**

Общее количество контигов: 613

Общая длина контигов: 3925649

Длина самого длинного контига: 179307

N50: 47993


**2. Анализ скаффолдов**

Общее количество скаффолдов: 65

Общая длина скаффолдов: 3876111

Длина самого длинного скаффолда: 3835005

N50: 3835005


Количество гэпов самого длинного скаффолда: 62

Общая длина гэпов самого длинного скаффолда: 7011


**3. Анализ скаффолдов после gap_close**

Общее количество скаффолдов: 65

Общая длина скаффолдов: 3921655

Длина самого длинного скаффолда: 3880382

N50: 3880382


Количество гэпов самого длинного скаффолда (после gap_close): 9

Общая длина гэпов самого длинного скаффолда (после gap_close): 1647

____

###  Бонус

С помощью аналогичных команд получены еще 2 сборки. Первая использует в 2 раза меньше чтений(2500000 PE, 750000 MP), чем исходная(5000000 PE, 1500000 MP), вторая - в 4 раза меньше чтений(1250000 PE, 375000 MP).

____

### В 2 раза меньше чтений

Скриншоты и статистика из файлов multiQC:

[Неподрезанные чтения](bonus/half/html_reports/multiqc_report.html)

![multiqc1](bonus/half/screenshots/multiqc21.jpg)
![multiqc2](bonus/half/screenshots/multiqc22.jpg)
![multiqc3](bonus/half/screenshots/multiqc23.jpg)

[Подрезанные чтения](bonus/half/html_reports/multiqc_trimmed_report.html)

![multiqc_trimmed1](bonus/half/screenshots/multiqc_trimmed21.jpg)
![multiqc_trimmed2](bonus/half/screenshots/multiqc_trimmed22.jpg)
![multiqc_trimmed3](bonus/half/screenshots/multiqc_trimmed23.jpg)

[Jupiter-ноутбук](bonus/half/src/analysis_of_contigs_and_scaffolds.ipynb)

____

### В 4 раза меньше чтений

Скриншоты и статистика из файлов multiQC:

[Неподрезанные чтения](bonus/quarter/html_reports/multiqc_report.html)

![multiqc1](bonus/quarter/screenshots/multiqc41.jpg)
![multiqc2](bonus/quarter/screenshots/multiqc42.jpg)
![multiqc3](bonus/quarter/screenshots/multiqc43.jpg)

[Подрезанные чтения](bonus/quarter/html_reports/multiqc_trimmed_report.html)

![multiqc_trimmed1](bonus/quarter/screenshots/multiqc_trimmed41.jpg)
![multiqc_trimmed2](bonus/quarter/screenshots/multiqc_trimmed42.jpg)
![multiqc_trimmed3](bonus/quarter/screenshots/multiqc_trimmed43.jpg)

[Jupiter-ноутбук](bonus/quarter/src/analysis_of_contigs_and_scaffolds.ipynb)

____

| Параметр | Исходная сборка | В 2 раза меньше чтений | В 4 раза меньше чтений|
|----------|:---------------:|-----------------------:|----------------------:|
| Общее количество контигов |613 |719 |972 |
| Общая длина контигов |3925649 |3925446 |3916798 |
| Длина самого длинного контига |179307 |232786 |259998 |
| N50 контигов|47993 |75466 |81116 |
| Общее количество скаффолдов |65 |81 |120 |
| Общая длина скаффолдов |3921655 |3891128 |3865401 |
| Длина самого длинного скаффолда |3880382 |2105578 |2088042 |
| N50 скаффолдов|3880382 |2105578 |2088042 |
| Количество гэпов самого длинного скаффолда |9 |11 |21 |
| Общая длина гэпов самого длинного скаффолда |1647 |2875 |5202 |

При уменьшении количества данных качество сборки падает. Общее количество контигов и скаффолдов растет, длина самого длинного скаффолда и N50 для скаффолдов уменьшаются, количество и общая длина гэпов самого длинного скаффолда растут.
