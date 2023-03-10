Regressionstest 1.0.0 vs 1.0.1
================
Rebecca Folmer Schade
2023-03-08

I have tried to do a regression test in two different ways: First I
compare the two data sets directly - Are they the same? Where are they
different?

Then, I calculate the sum of each column and compare that.

### Load data

``` r
# Load Consistency 1_0_0
con_00 <- read_csv2("../Data/Consistency 1_0_0.csv")

# Load Consistency 1_0_1
con_01 <- read_csv2("../Data/Consistency 1_0_1.csv")
```

### Rename columns

``` r
#===> Lets start with the easy stuff
# Create new objects 
clean_00 <- con_00
clean_01 <- con_01

# Remove _ in start of colnames 
colnames(clean_00) <- gsub("^[[:punct:]]+", "", colnames(clean_00))
colnames(clean_01) <- gsub("^[[:punct:]]+", "", colnames(clean_01))

# Change punctuation
colnames(clean_00) <- gsub("[[:punct:]]+", "_", colnames(clean_00))
colnames(clean_01) <- gsub("[[:punct:]]+", "_", colnames(clean_01))

# gsub("([[:lower:]])([[:upper:]])", "\\1_\\2", strn)

#===> And now for the hard one
# Define function
rep_num <- function(idx, col_list) {
  start_idx <- idx
  end_idx <- idx + 126
  sub <- col_list[start_idx:end_idx]
  prefix <- str_extract(sub[1], "CF.+")
  rep_with <- sprintf("%s_\\1", prefix)
  sub[2:127] <- gsub("(^\\d+)(.+)", rep_with, sub[2:127])
  
  return(sub)
}

# Define list of colnames
col_list_00 <- colnames(clean_00)
col_list_01 <- colnames(clean_01)

# Get index of CF columns
idx_00 <- grep(".+CF.+", col_list_00)
idx_01 <- grep(".+CF.+", col_list_01)

# modify column list 
l_00 <- lapply(idx_00, function(x) rep_num(x, col_list_00))
col_list_00[idx_00[1]:length(col_list_00)] <- unlist(l_00)
#
l_01 <- lapply(idx_01, function(x) rep_num(x, col_list_01))
col_list_01[idx_01[1]:length(col_list_01)] <- unlist(l_01)


# Rename columns
colnames(clean_00) <- col_list_00
colnames(clean_01) <- col_list_01
```

## Look at data structure

First I Check that the two data sets are the same size.

``` r
# Check how many columns they have
print(
  paste("The Version 1.0.0 data set has", ncol(clean_00), "columns", sep = " ")
  )
```

    ## [1] "The Version 1.0.0 data set has 1404 columns"

``` r
print(
  paste("The Version 1.0.1 data set has", ncol(clean_01), "columns", sep = " ")
  )
```

    ## [1] "The Version 1.0.1 data set has 1277 columns"

``` r
# > There are more columns in the old data set
print(
  paste("There are", 
        ncol(clean_00) - ncol(clean_01), 
        "more columns in the version 1.0.0 data set", sep = " ")
  )
```

    ## [1] "There are 127 more columns in the version 1.0.0 data set"

``` r
# What about the rows? 
print(
  paste("The Version 1.0.0 data set has", nrow(clean_00), "rows", sep = " ")
  )
```

    ## [1] "The Version 1.0.0 data set has 354 rows"

``` r
print(
  paste("The Version 1.0.1 data set has", nrow(clean_01), "rows", sep = " ")
  )
```

    ## [1] "The Version 1.0.1 data set has 353 rows"

The new dataset seems to be missing a row and a bunch of columns.  
Which ones?

``` r
# Which ID's do not appear in the other data set?  
print(
  paste(
    "Row",
    clean_00$RecordIdentifier[!clean_00$RecordIdentifier %in% clean_01$RecordIdentifier],
    "is misssing", sep = " ")
)
```

    ## [1] "Row F44936EAB043EDB60F3AA8A522D2D70548410BE6 is misssing"

``` r
# Which columns are different? 
diff_col <- setdiff(colnames(clean_00), colnames(clean_01))
# Write to txt
file <- file.path("..", "output", "100_vs_101", "missing_col.csv")
write.csv2(as_tibble(diff_col), file = file)
```

## One to one comparison

Lets remove the rows and columns that don???t appear in clean_01 so we can
compare the data sets mere exactly.

``` r
# Select all other columns
comp_00 <- clean_00 %>% select(-matches(diff_col))
# > Now the two dataframes have the same nr. of columns!

# Get rid of the extra row
comp_00 <- subset(comp_00, RecordIdentifier != "F44936EAB043EDB60F3AA8A522D2D70548410BE6")

# Let's check if all rows are present 
sum(!comp_00$RecordIdentifier %in% clean_01$RecordIdentifier)
sum(!clean_01$RecordIdentifier %in% comp_00$RecordIdentifier)
# > Yes! All rows are present!
```

Now for the more fine-grained comparison

``` r
# Let's have a look at different compariason methods
d <- summary(comparedf(comp_00, clean_01))
# These are the interesting ones
# d$comparison.summary.table
# d$diffs.byvar.table
# d$diffs.table


# Let's try to save the summaries with the relevant information as tables
# Summary[2] - overview
summary_table <- d$comparison.summary.table
# > Looks like we have 118 columns with different values.

# Summary[6] - columns with differences
diff_byvar <- as_tibble(d$diffs.byvar.table) %>% select(1:3) %>% filter(n > 0)
colnames(diff_byvar) <- c("col_100", "col_101", "n_differences")

# Summary[7] - differences by row
diff_byrow <- as_tibble(d$diffs.table) %>% select(1:5)
colnames(diff_byrow) <- c("col_100", "col_101", "row_name", "value_100", "value_101")

# Let's save them to csv
file_1 <- file.path("..", "output", "100_vs_101", "summary_table.csv")
write.csv2(summary_table, file = file_1)
#
file_2 <- file.path("..", "output", "100_vs_101", "diff_by_col.csv")
write.csv2(diff_byvar, file = file_2)
# 
file_3 <- file.path("..", "output", "100_vs_101", "diff_by_row.csv")
write.csv2(diff_byrow, file = file_3)
```

## Summary statistics

``` r
# Let's have a look at the summary stats
sum_stats_00 <- tail(comp_00, n = 6)
sum_stats_01 <- tail(clean_01, n = 6)

# In theory, we could find all the columns with different values by checking if the sums are the same
sum_list_00 = sum_stats_00[1, ] %>% t() %>% as.list() %>% unlist()
sum_list_01 = sum_stats_01[1, ] %>% t() %>% as.list() %>% unlist()

# Assign a dummy value
sum_list_00[is.na(sum_list_00)] <- "blank"
sum_list_01[is.na(sum_list_01)] <- "blank"

# Check for differences 
sum(sum_list_00 != sum_list_01)
all.equal(sum_list_00, sum_list_01)
# > This is curious - there are no differences between the two - and there should be. 
# > This means that there is an issue with the summary stats. 
```

I???ll try to calculate the sum of the columns myself, to see if the
method works, if nothing else.

``` r
# Isolating numeric variables
num_00 <- comp_00 %>% select_if(is.numeric)
num_01 <- clean_01 %>% select_if(is.numeric)

# Calculating sums
sums_00 <- colSums(num_00, na.rm = TRUE)
sums_01 <- colSums(num_01, na.rm = TRUE)

# Compare and count the differences
print(
  paste(sum(sums_00 != sums_01), "columns have different values")
  )
```

    ## [1] "129 columns have different values"

``` r
# > Interesting - the two comparison methods are in disagreement. 
```

This method says that there are 129 varaibles with differences - more
than comparing one to one. Let???s try to solve the mystery.

``` r
# Which columns are identified as different by the sum method? 
num_diff <- which(sums_00 != sums_01) %>% names()

# Which columns are different with compare cells method? 
cell_diff <- diff_byvar$col_100

# Now lets compare
print(
  paste(sum(!num_diff %in% cell_diff), "columns in num_diff but not in cell_diff")
  )
```

    ## [1] "11 columns in num_diff but not in cell_diff"

``` r
# > This was what I expected, but it never hurts to be sure

# Which columns? 
l <- which(!num_diff %in% cell_diff)
print(
  sprintf("These columns are: %s", 
          paste(num_diff[l], collapse = ", "))
  )
```

    ## [1] "These columns are: CFExpenseTotal_t0_83, CFExpenseTotal_t0_84, CFExpenseTotal_t0_85, CFExpenseTotal_t0_86, CFExpenseTotal_t0_87, CFExpenseTotal_t0_88, CFExpenseTotal_t0_89, CFExpenseTotal_t0_90, CFExpenseTotal_t0_91, CFExpenseTotal_t0_92, CFExpenseTotal_t0_93"

``` r
# > so it is a couple of ExpenseTotal columns.

# Compare two columns 
print(
  paste(sum(comp_00$CFExpenseTotal_t0_87 != clean_01$CFExpenseTotal_t0_87, na.rm = TRUE), 
        "of the values in CFExpenseTotal_t0_87 don't match"))
```

    ## [1] "7 of the values in CFExpenseTotal_t0_87 don't match"

``` r
# > Looks like there are a couple of cells that don't match - curiouser and curiouser

# Do I get the same result with an all.equal or an Identical?
print(
  paste("Using the identical() function it is", 
        identical(comp_00$CFExpenseTotal_t0_87, clean_01$CFExpenseTotal_t0_87),
        "that CFExpenseTotal_t0_87 is the same"
            ))
```

    ## [1] "Using the identical() function it is FALSE that CFExpenseTotal_t0_87 is the same"

``` r
#
print(
  paste("Using the all.equal() function it is", 
        all.equal(comp_00$CFExpenseTotal_t0_87, clean_01$CFExpenseTotal_t0_87),
        "that CFExpenseTotal_t0_87 is the same"
            ))
```

    ## [1] "Using the all.equal() function it is TRUE that CFExpenseTotal_t0_87 is the same"

``` r
# > Aha! I think I see the problem! Let me just try to see if I can confirm my theory...

# Examine the values
l <- which(comp_00$CFExpenseTotal_t0_87 != clean_01$CFExpenseTotal_t0_87)
# 
print(
  sprintf("In 1.0.0 the values are: %s", 
          paste(comp_00$CFExpenseTotal_t0_87[l] %>% formatC(format = "f", digits = 10), collapse = ", "))
  )
```

    ## [1] "In 1.0.0 the values are: 0.0000000000, 0.0000000000, 0.0000000000, 0.0000000032, 0.0000000032, 0.0000000032, 0.0000000032"

``` r
#
print(
  sprintf("In 1.0.1 the values are: %s", 
          paste(clean_01$CFExpenseTotal_t0_87[l] %>% formatC(format = "f", digits = 10), collapse = ", "))
  )
```

    ## [1] "In 1.0.1 the values are: 0.0000000001, 0.0000000001, 0.0000000001, 0.0000000035, 0.0000000035, 0.0000000035, 0.0000000035"

``` r
# > I was right! The difference is so small that comparedf() does not pick up on it! 
# > At least not with its default sensitivety. 
# > Though it can be discussed whether differences that small are even important - or maybe artfacts of calculation. 

# Let's get a list of the columns
sum_diff <- as_tibble(which(sums_00 != sums_01) %>% names())
file_4 <- file.path("..", "output", "100_vs_101", "sum_diff.csv")
write.csv2(sum_diff, file = file_4)
```
