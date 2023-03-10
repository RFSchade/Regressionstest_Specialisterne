Regressionstest 101 vs 102
================
Rebecca Folmer Schade
2023-03-10

Let’s figure out if there are any improvements in version 1.0.2!

### Load data

``` r
# Load Consistency 1_0_0
og_tbl <- read_csv2("../Data/Consistency 1_0_0.csv")

# Load Consistency 1_0_1
old_tbl <- read_csv2("../Data/Consistency 1_0_1.csv")

# Load Consistency 1_0_2
new_tbl <- read_csv2("../Data/Consistency 1_0_2.csv")
```

### Rename columns

``` r
#===> Lets start with the easy stuff
# Create new objects 
old_clean <- old_tbl
new_clean <- new_tbl
og_clean <- og_tbl

# Remove _ in start of colnames 
colnames(old_clean) <- gsub("^[[:punct:]]+", "", colnames(old_clean))
colnames(new_clean) <- gsub("^[[:punct:]]+", "", colnames(new_clean))
colnames(og_clean) <- gsub("^[[:punct:]]+", "", colnames(og_clean))

# Change punctuation
colnames(old_clean) <- gsub("[[:punct:]]+", "_", colnames(old_clean))
colnames(new_clean) <- gsub("[[:punct:]]+", "_", colnames(new_clean))
colnames(og_clean) <- gsub("[[:punct:]]+", "_", colnames(og_clean))

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
col_list_old <- colnames(old_clean)
col_list_new <- colnames(new_clean)
col_list_og <- colnames(og_clean)

# Get index of CF columns
idx_old <- grep(".+CF.+", col_list_old)
idx_new <- grep(".+CF.+", col_list_new)
idx_og <- grep(".+CF.+", col_list_og)

# modify column list 
l_old <- lapply(idx_old, function(x) rep_num(x, col_list_old))
col_list_old[idx_old[1]:length(col_list_old)] <- unlist(l_old)
#
l_new <- lapply(idx_new, function(x) rep_num(x, col_list_new))
col_list_new[idx_new[1]:length(col_list_new)] <- unlist(l_new)
#
l_og <- lapply(idx_og, function(x) rep_num(x, col_list_og))
col_list_og[idx_og[1]:length(col_list_og)] <- unlist(l_og)


# Rename columns
colnames(old_clean) <- col_list_old
colnames(new_clean) <- col_list_new
colnames(og_clean) <- col_list_og
```

## Look at data structure

First I Check that the two data sets are the same size.

``` r
# Check how many columns they have
print(
  paste("The Version 1.0.1 data set has", ncol(old_clean), "columns", sep = " ")
  )
```

    ## [1] "The Version 1.0.1 data set has 1277 columns"

``` r
print(
  paste("The Version 1.0.2 data set has", ncol(new_clean), "columns", sep = " ")
  )
```

    ## [1] "The Version 1.0.2 data set has 1404 columns"

``` r
# > There are more columns in the new data set
print(
  paste("There are", 
        ncol(new_clean) - ncol(old_clean), 
        "more columns in the version 1.0.2 data set", sep = " ")
  )
```

    ## [1] "There are 127 more columns in the version 1.0.2 data set"

``` r
# What about the rows? 
print(
  paste("The Version 1.0.1 data set has", nrow(old_clean), "rows", sep = " ")
  )
```

    ## [1] "The Version 1.0.1 data set has 353 rows"

``` r
print(
  paste("The Version 1.0.2 data set has", nrow(new_clean), "rows", sep = " ")
  )
```

    ## [1] "The Version 1.0.2 data set has 354 rows"

It could look like the bug have been fixed on the newest version, just
looking at the data structure.

``` r
# Which ID's do not appear in the other data set?  
print(
  paste(
    "Row",
    new_clean$RecordIdentifier[!new_clean$RecordIdentifier %in% old_clean$RecordIdentifier],
    "is misssing", sep = " ")
)
```

    ## [1] "Row F44936EAB043EDB60F3AA8A522D2D70548410BE6 is misssing"

``` r
# > Great! It's the same row. 


# Which columns are different? 
diff_col <- setdiff(colnames(new_clean), colnames(old_clean))
# Are they the same as the ones?
print(
  paste("It is", 
        identical(setdiff(colnames(og_clean), colnames(old_clean)), setdiff(colnames(new_clean), colnames(old_clean))),
        "that it is the same columns")
)
```

    ## [1] "It is TRUE that it is the same columns"

## One to one comparison

Lets remove the rows and columns that don’t appear in old_clean so we
can compare the data sets mere exactly.

``` r
# Select all other columns
comp_new <- new_clean %>% select(-matches(diff_col))
# > Now the two dataframes have the same nr. of columns!

# Get rid of the extra row
comp_new <- subset(comp_new, RecordIdentifier != "F44936EAB043EDB60F3AA8A522D2D70548410BE6")

# Let's check if all rows are present 
sum(!comp_new$RecordIdentifier %in% old_clean$RecordIdentifier)
sum(!old_clean$RecordIdentifier %in% comp_new$RecordIdentifier)
# > Yes! All rows are present!
```

Now for the more fine-grained comparison

``` r
# Let's have a look at different compariason methods
d <- summary(comparedf(old_clean, comp_new))
# These are the interesting ones
# d$comparison.summary.table
# d$diffs.byvar.table
# d$diffs.table


# Let's try to save the summaries with the relevant information as tables
# Summary[2] - overview
summary_table <- d$comparison.summary.table
# > This time we have 35 variables and 12 rows with differences. 45 values total. 

# Summary[6] - columns with differences
diff_byvar <- as_tibble(d$diffs.byvar.table) %>% select(1:3) %>% filter(n > 0)
colnames(diff_byvar) <- c("col_101", "col_102", "n_differences")
# > Insured_Age still has the same amount of differences - they were corrected in 1.0.2
# > And given that those are set to 147.75, it is likely that those are artifacts. 

# Summary[7] - differences by row
diff_byrow <- as_tibble(d$diffs.table) %>% select(1:5)
colnames(diff_byrow) <- c("col_101", "col_102", "row_name", "value_101", "value_102")

# Let's save them to csv
file_1 <- file.path("..", "output", "101_vs_102", "summary_table.csv")
write.csv2(summary_table, file = file_1)
#
file_2 <- file.path("..", "output", "101_vs_102", "diff_by_col.csv")
write.csv2(diff_byvar, file = file_2)
# 
file_3 <- file.path("..", "output", "101_vs_102", "diff_by_row.csv")
write.csv2(diff_byrow, file = file_3)
```
