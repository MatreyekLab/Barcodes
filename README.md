Barcodes
================
Kenneth Matreyek
2025-05-27

As part of the AVE-ETS, we had been discussing barcoded variant
libraries. I don’t quite remember the context, but I think I suggested
we look at some real sequencing data of a barcoded variant library, and
I offered to dig up the PTEN VAMP-Seq library data. Of course, I had
other things to do in the month following this statement so I didn’t
look for those files until the long weekend immediately prior to the
next meeting. My original plan was to just find this blog post
\[<https://www.matreyeklab.com/simulating-sampling-during-recombination/1175/>\]
and say we use that, but then I realized that those data tables were for
variant frequencies, and not barcode counts. All of my old data from my
postdoc was on flash drives in the office at work, but I didn’t feel
like making the commute in just to for that. I decided to try to find
and re-process the raw data uploaded to GEO / SRA.

First, a number of steps that aren’t explicitly coded in this markdown
file.

1)  I downloaded the files from the relevant GEO sites. The first
    dataset from the NatGenet paper can be found here
    \[<https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE108727>\].
    The second dataset from the Genome Med paper where we published on a
    “fill-in” library we created to reintroduce some missing variants
    can be found here
    \[<https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE159469>\].
2)  I counted the barcodes using the original method we had used, which
    was just having Enrich2 do it, with a minimum quality score filter
    of 30.
3)  I imported the data into R to do some analyses here

``` r
library(tidyverse)
```

    ## Warning: package 'ggplot2' was built under R version 4.2.3

    ## Warning: package 'tidyr' was built under R version 4.2.3

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.2     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
theme_set(theme_bw())
theme_update(panel.grid.minor = element_blank())
```

``` r
first_1 <- read.delim(file = "Data/SRR6437841.tsv.gz", sep = "\t")
first_2 <- read.delim(file = "Data/SRR6437842.tsv.gz", sep = "\t")

first <- merge(first_1, first_2, by = "X")
first$count = rowMeans(first[,c("count.x","count.y")])

ggplot() + scale_x_log10() + #scale_y_log10() +
  geom_histogram(data = first %>% filter(count > 3), aes(x = count)) + geom_vline(xintercept = 100)
```

![](Barcodes_files/figure-gfm/VAMP-Seq%20GEO%20deposit%20of%20the%20original%20PTEN%20library%20plasmid%20sequencing-1.png)<!-- -->

``` r
## As a rough but effective approach, I like to look at the histgoram os counts to identify where the relative minima between the population containing 1  (largely erroneous barcodes from sequencing error) and the net non-zero population (assuming the sample was sequenced to enough depth). For this sample, since it was sequenced so deeply, this is around a count of 100.

first_filtered <- first %>% filter(count > 100)

#first_key <- read.delim(file = "Data/GSE108727_PTEN_barcodeInsertAssignments.tsv", sep = "\t", header = F)
first_key <- read.delim(file = "Data/first_key.tsv", sep = "\t", header = F)
colnames(first_key) <- c("X","variant")

first_df <- merge(first_filtered, first_key, by = "X", all.x = T)

First_lib_histogram <- ggplot() + scale_x_log10() + #scale_y_log10() +
  geom_histogram(data = first_df, aes(x = count), fill = "grey90", bins = 50) +
  geom_histogram(data = first_df %>% filter(!is.na(variant)), aes(x = count), bins = 50) + 
  geom_vline(xintercept = 100) + 
  labs(x = "Num of reads", y = "Count", title = "Lib1: Grey, all barcodes; Black, subassembled variants") +
  theme(panel.grid.minor = element_blank()) + 
  NULL; First_lib_histogram
```

![](Barcodes_files/figure-gfm/VAMP-Seq%20GEO%20deposit%20of%20the%20original%20PTEN%20library%20plasmid%20sequencing-2.png)<!-- -->

``` r
ggsave(file = "Output/First_lib_histogram.pdf", First_lib_histogram, height = 4, width = 5)
```

Some notes for the above plot. The grey bars of the histogram denote
counts for all barcodes that were observed. The black bars are barcodes
that were linked to a particular PTEN coding variant via PacBio
subassembly.

``` r
second_1 <- read.delim(file = "Data/SRR12818211.tsv.gz", sep = "\t")
second_2 <- read.delim(file = "Data/SRR12818212.tsv.gz", sep = "\t")

second <- merge(second_1, second_2, by = "X")
second$count = rowMeans(second[,c("count.x","count.y")])

ggplot() + scale_x_log10() + #scale_y_log10() +
  geom_histogram(data = second %>% filter(count > 1), aes(x = count)) + geom_vline(xintercept = 10)
```

![](Barcodes_files/figure-gfm/VAMP-Seq%20GEO%20deposit%20of%20the%20second%20PTEN%20library%20plasmid%20sequencing-1.png)<!-- -->

``` r
## As a rough but effective approach, I like to look at the histgoram os counts to identify where the relative minima between the population containing 1  (largely erroneous barcodes from sequencing error) and the net non-zero population (assuming the sample was sequenced to enough depth). For this sample, it's around 10.

second_filtered <- second %>% filter(count > 10)

second_key <- read.delim(file = "Data/Second_key.tsv", sep = "\t", header = T)
colnames(second_key)[2] <- "variant"

second_df <- merge(second_filtered, second_key, by = "X", all.x = T)

Second_lib_histogram  <- ggplot() + scale_x_log10() + #scale_y_log10() +
  geom_histogram(data = second_df, aes(x = count), fill = "grey90") +
  geom_histogram(data = second_df %>% filter(!is.na(variant)), aes(x = count)) + 
  geom_vline(xintercept = 100) +
  labs(x = "Num of reads", y = "Count", title = "Lib2: Grey, all barcodes; Black, subassembled variants") +
  theme(panel.grid.minor = element_blank()) + 
  NULL; Second_lib_histogram
```

![](Barcodes_files/figure-gfm/VAMP-Seq%20GEO%20deposit%20of%20the%20second%20PTEN%20library%20plasmid%20sequencing-2.png)<!-- -->

``` r
ggsave(file = "Output/Second_lib_histogram.pdf", Second_lib_histogram, height = 4, width = 5)

## I have no idea why it looks bimodal , btw. Probably a prob with library mixing.
```

``` r
write.table(file = "Output/First_df.tsv", first_df, quote = F, row.names = F)
write.table(file = "Output/Second_df.tsv", second_df, quote = F, row.names = F)
```

For anyone that wants to test this, the github repo for the above
scripts and data can be found here:
<https://github.com/MatreyekLab/Barcodes>
