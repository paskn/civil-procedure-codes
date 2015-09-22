# Split codes into chunks
Lincoln Mullen  
September 21, 2015  

The goal is to split codes into chunks, then identify candidates with minhash and LSH. (See the textreuse package documentation.)


```r
library(textreuse)
library(stringr)
source("R/split-doc.R")
dir.create("legal-codes-split", showWarnings = FALSE)
```

Create a regular expression to section the codes.


```r
ny_1850_pattern <- regex("(\n§((\\s+)?)\\d+((\\.)?)|\n\\$((\\s+)?)\\d+((\\.)?)|\nchapter|\ntitle|\narticle|\nt i t l e|\nRULE \\w+\\.|\n\\d{4,}\\.|\nSEC((\\.)?)\\s+\\d+((\\.)?)|\n8EC((\\.)?)\\s+\\d+((\\.)?)|\nSE0((\\.)?)\\s+\\d+((\\.)?)|\nSEO((\\.)?)\\s+\\d+((\\.)?)|\nSEQ((\\.)?)\\s+\\d+((\\.)?))", ignore_case = TRUE)
test <- str_detect(c("\n§ 1842 This should be true", "\nThis should be false §",
                     "\n§ 142. This should be true", "\nThis should be false §",
                     "\n§142. This should be true", "\nThis should be false §",
                     "\nchapter should be true", "\nThis chapter should be false",
                     "\nCHAPTER should be true", "\nthis CHAPTER should be false",
                     "\nT I T L E should be true", "false",
                     "\n$342. This should be true", "false",
                     "\n$ 323. This should be true", "false",
                     "\nRULE XVI. This should be true", "This rule is false",
                     "\n3652. From AL1852 true", "232. False",
                     "\n3652. From AL1852 true", "Citation 232. False",
                     "\nSEC. 431. true", "This SEC 133 false",
                     "\n8EC. 3323. true", "false",
                     "\nSEC.   2545 true", "false"),
                   pattern = ny_1850_pattern)
all(test == c(TRUE, FALSE))
```

```
## [1] TRUE
```

Split the codes.


```r
split_doc("legal-codes/NY1850.txt", ny_1850_pattern, "legal-codes-split/NY1850")
split_doc("legal-codes/CA1851.txt", ny_1850_pattern, "legal-codes-split/CA1851")
split_doc("legal-codes/NC1868.txt", ny_1850_pattern, "legal-codes-split/NC1868")
split_doc("legal-codes/AL1852.txt", ny_1850_pattern, "legal-codes-split/AL1852")
split_doc("legal-codes/OH1853.txt", ny_1850_pattern, "legal-codes-split/OH1853")
split_doc("legal-codes/OH1853extended.txt", ny_1850_pattern, "legal-codes-split/OH1853extended")
split_doc("legal-codes/CA1850.txt", ny_1850_pattern, "legal-codes-split/CA1850")
split_doc("legal-codes/GA1860.txt", ny_1850_pattern, "legal-codes-split/GA1860")
split_doc("legal-codes/CA1868.txt", ny_1850_pattern, "legal-codes-split/CA1868")
split_doc("legal-codes/AZ1865.txt", ny_1850_pattern, "legal-codes-split/AZ1865")
```

What are the mean, median, and mode lengths of sections in the NY 1850 code?


```r
ny_1850_code <- TextReuseCorpus(Sys.glob("legal-codes-split/NY1850*"),
                                tokenizer = tokenize_words)
ny_1850_wc <- wordcount(ny_1850_code)
mean(ny_1850_wc)
```

```
## [1] 71.57197
```

```r
median(ny_1850_wc)
```

```
## [1] 59
```

```r
head(sort(-table(ny_1850_wc))) # mode
```

```
## ny_1850_wc
##  49  32  38  43  50  46 
## -35 -32 -32 -31 -31 -30
```

```r
boxplot(ny_1850_wc, main = "Word counts of sections in NY1850")
```

![](011-split-codes_files/figure-html/unnamed-chunk-4-1.png) 

## Known problems

- NC1868 has sections without section marks, so subsequent sections are joined together not split up.
