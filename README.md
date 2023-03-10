# Regressionstest 
Min besvarelse på Regressionstestopgaven for Specialisternes Academy forløb forår 2023

Jeg har løst opgaven på 2 måder: 
- Ved at lave en en-til-en sammenligning af hver række/kolonne, og isolere de enkelte celler der er forskellige i de to datasæt. 
- Ved at udregne summen af alle numeriske variabler, og se om de matcher.

### Repository struktur
__:file_folder: Data:__ Mappe med data der skal sammenlignes og valideres.     

__:file_folder: Markdowns:__ Mappe med RMarkdowns. De indeholder den kode jeg har brugt.
- Regressionstest.Rmd: RMardsown med kode til sammenligning a Consistency 1.0.0.csv og Consistency 1.0.1.csv.
- Regressionstest.md: Mardown der er nemmere at læse i GitHub.
- Regressionstest_101_vs_102.Rmd: RMarkdown med kode til sammenligning a Consistency 1.0.1.csv og Consistency 1.0.2.csv.
- Regressionstest_101_vs_102.md: Mardown der er nemmere at læse i GitHub.

__:file_folder: output:__ Mappe med output fra Markdowns.    
- __:file_folder: 100_vs_101:__ Til sammenligningen mellem 1.0.0 og 1.0.1
  - diff_by_col.csv: Liste over de kolonner der er forskelle i, og hvor mange
  - diff_by_row.csv: Liste over hvilke celler er forskellige
  - missing_col.csv: Liste over kolonner som er i 1.0.0, men ikke i 1.0.1
  - sum_diff.csv: Liste over de kolonner der er forskelle i, fundet ved at udregne summen af kolonnerne og sammenligne dem. 
  - summary_table.csv: Et overblik

- __:file_folder: 101_102:__ Til sammenligning mellem 1.0.1 og 1.0.2
  - diff_by_col.csv: Liste over de kolonner der er forskelle i, og hvor mange
  - diff_by_row.csv: Liste over hvilke celler er forskellige
  - summary_table.csv: Et overblik

__:page_facing_up: .gitignore__

### Ting der kunne gøres bedre
1. Jeg ville gerne have haft til til at lave en requirements.txt fil. 
2. Jeg ville gerne have lavet et script som hurtigt kunne have lavet samme tests som jeg gjorde. 
3. Jeg ville gerne have brugt mere tid på at validere data - måske ville jeg, via nogle beskrivende statistikker, kunne komme tættere på, hvilke forskelle mellem datasætne er fejl, og hvilke er forbedringer. 
4. Som jeg sidder og skriver det her, husker jeg at jeg glemte at tjekke, om de sammenfattende statistikker der er i bunden af 1.0.2 blev rettet. 
5. Jeg har det som om jeg stadig ikke helt ved om jeg har løst opgaven rigtigt. 
