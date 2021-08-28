# Analysis of AO3's Selective data dump for fan statisticians (March 2021)

*Disclaimer: the columns I use to describe the schemas of these data sets are not necessarily the actual column names of the files. This is a description of my data manipulation & aggregation processes, not a data dictionary. I also recommend checking out @mousemode's [repo on github](https://github.com/mousemode/AO3_Data_Dump) - their approach is more robust.*

## Preparation for General Tag Analysis

This was done in R, with the power of tidyverse! 

Original schema of AO3 data dump; all tag IDs are in one cell, separated by "+". Work IDs have been removed, so there is no primary key.

| Creation Date | Language | Word Count | Restricted or Not | Complete or Not  | All Associated Tag IDs |
| --- | --- | --- | --- | --- | --- |

I assigned works a unique ID by appending the row number to each record, then split all tag IDs on "+" and pivoted them. Now there are many rows for each work, each with a single Tag ID in the Tag ID cell.

| Row ID | Creation Date | Language | Word Count | Restricted or Not | Complete or Not  | Associated Tag ID  |
| --- | --- | --- | --- | --- | --- | --- |

Here's the code:
```
setwd("YOUR_WD")
library(tidyverse)
works <- read_csv("ao3works-20210226.csv")
works$ID <- 1:nrow(works)
works_pivot <- separate_rows(works, tags, sep = "\\+")
write_csv(works_pivot, "YOUR_WD/ao3workspivot", na = "NA", append = FALSE, col_names = !append)
```

## Aggregation for Tag Limit Analysis

I used Tableau Prep to aggregate the AO3 data dump to explore the characteristics of works on AO3 with 75+ tags - [see the viz on Tableau Public](https://public.tableau.com/app/profile/ladyofthelog/viz/AO3TagsperWork2008-2020/Overview). As the resulting data set contains only 247 rows, I was able to include it in this repo in case you'd like to play with it.

Original schema of AO3 data dump; all tag IDs are in one cell, separated by "+". Work IDs have been removed, so there is no primary key.

| Creation Date | Language | Word Count | Restricted or Not | Complete or Not  | All Associated Tag IDs |
| --- | --- | --- | --- | --- | --- |

I assigned works a unique ID by appending the row number to each record, then aggregated by counting tags on a work; individual tag IDs are dropped at this point. I also removed a few other fields as the numbers of non-EN works, Restricted works, and incomplete works are relatively low. 

| Row ID | Creation Date | Word Count | Tag ID Count |
| --- | --- | --- | --- |

Next, I manually created two sets of bins using calculated fields - Word Count Bins (from Word Count) and Excess Tags (from Tag ID count), then aggregated Creation Date at the year level.

| Row ID | Year of Creation Date | Word Count | Tag ID Count | Word Count Bins | Excess Tags |
| --- | --- | --- | --- | --- | --- |

I then calculated minimum, maximum, and median tag count values for all word count bins; separately, I calculated median tag count for works with excess tags within each word count bin. That created these *additional* columns:

| Median Tag Count | Min Tag Count | Max Tag Count | Bucketed Median Tag Count|
| --- | --- | --- | --- |

The data set is available [in this repo](https://github.com/ladyofthelog/ao3-data-dump/blob/main/AO3%20Aggregated%20Tagging%20Data.csv).
