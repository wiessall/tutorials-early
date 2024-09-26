---
title: 'Clean and validate'
teaching: 20
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions 

- How to clean and standardize case data?
- How to convert raw dataset into a `linelist` object?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Explain how to clean, curate, and standardize case data using `{cleanepi}` package
- Demonstrate how to covert case data to `linelist` data 
- Perform essential data-cleaning operations to be performed in a raw case dataset.

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::: prereq

This episode requires you to:

- Download the [simulated_ebola_2.csv](https://epiverse-trace.github.io/tutorials-early/data/simulated_ebola_2.csv)
- Save it in the `data/` folder.

:::::::::::::::::::::

## Introduction
In the process of analyzing outbreak data, it's essential to ensure that the dataset is clean, curated, standardized, and valid to facilitate accurate and reproducible analysis. This episode focuses on cleaning epidemics and outbreaks data using the [cleanepi](https://epiverse-trace.github.io/cleanepi/) package, and validate it using the [linelist](https://epiverse-trace.github.io/linelist/) package. For demonstration purposes, we'll work with a simulated dataset of Ebola cases.

Let's start by loading the package `{rio}` to read data and the package `{cleanepi}` to clean it. We'll use the pipe `%>%` to connect some of their functions, including others from the package `{dplyr}`, so let's also call to the tidyverse package:


``` r
# Load packages
library(tidyverse) # for {dplyr} functions and the pipe %>%
library(rio) # for importing data
library(here) # for easy file referencing
library(cleanepi)
```

::::::::::::::::::: checklist

### The double-colon

The double-colon `::` in R let you call a specific function from a package without loading the entire package into the current environment. 

For example, `dplyr::filter(data, condition)` uses `filter()` from the `{dplyr}` package.

This help us remember package functions and avoid namespace conflicts.

:::::::::::::::::::


The first step is to import the dataset following the guidelines outlined in the [Read case data](../episodes/read-cases.Rmd) episode. This involves loading the dataset into our environment and view its structure and content. 


``` r
# Read data
# e.g.: if path to file is data/simulated_ebola_2.csv then:
raw_ebola_data <- rio::import(
  here::here("data", "simulated_ebola_2.csv")
) %>%
  dplyr::as_tibble() # for a simple data frame output
```




``` r
# Return first five rows
raw_ebola_data
```

``` output
# A tibble: 15,000 × 7
      V1 `case id` age         gender status      `date onset` `date sample`
   <int>     <int> <chr>       <chr>  <chr>       <chr>        <chr>        
 1     1     14905 90          1      "confirmed" 03/15/2015   06/04/2015   
 2     2     13043 twenty-five 2      ""          Sep /11/Y    03/01/2014   
 3     3     14364 54          f       <NA>       09/02/2014   03/03/2015   
 4     4     14675 ninety      <NA>   ""          10/19/2014   31/ 12 /14   
 5     5     12648 74          F      ""          08/06/2014   10/10/2016   
 6     6     14274 seventy-six female ""          Apr /05/Y    01/23/2016   
 7     7     14132 sixteen     male   "confirmed" Dec /29/Y    05/10/2015   
 8     8     14715 44          f      "confirmed" Apr /06/Y    04/24/2016   
 9     9     13435 26          1      ""          09/07/2014   20/ 09 /14   
10    10     14816 thirty      f      ""          06/29/2015   06/02/2015   
# ℹ 14,990 more rows
```

##  A quick inspection

Quick exploration and inspection of the dataset are crucial before diving into any analysis tasks. The `{cleanepi}` package simplifies this process with the `scan_data()` function. Let's take a look at how you can use it:


``` r
cleanepi::scan_data(raw_ebola_data)
```

``` output
  Field_names  missing numeric     date character logical
1          V1 0.000000  1.0000 0.000000  0.000000       0
2     case id 0.000000  1.0000 0.000000  0.000000       0
3         age 0.064600  0.8348 0.000000  0.100600       0
4      gender 0.157867  0.0472 0.000000  0.794933       0
5      status 0.053533  0.0000 0.000000  0.946467       0
6  date onset 0.000067  0.0000 0.915733  0.084200       0
7 date sample 0.000133  0.0000 0.999867  0.000000       0
```


The results provides an overview of the content of every column, including column names, and the percent of some data types per column.
You can see that the column names in the dataset are descriptive but lack consistency, as some they are composed of multiple words separated by white spaces. Additionally, some columns contain more than one data type, and there are missing values in others.

## Common operations

This section  demonstrate how to perform some common data cleaning operations using the `{cleanepi}` package.

### Standardizing column names

For this example dataset, standardizing column names typically involves removing spaces and connecting different words with “_”. This practice helps maintain consistency and readability in the dataset.
However, the function used for standardizing column names offers more options. Type `?cleanepi::standardize_column_names` for more details.


``` r
sim_ebola_data <- cleanepi::standardize_column_names(raw_ebola_data)
names(sim_ebola_data)
```

``` output
[1] "v_1"         "case_id"     "age"         "gender"      "status"     
[6] "date_onset"  "date_sample"
```

::::::::::::::::::::::::::::::::::::: challenge 

- What differences you can observe in the column names?

::::::::::::::::::::::::::::::::::::::::::::::::

If you want to maintain certain column names without subjecting them to the standardization process, you can utilize the `keep` parameter of the `standardize_column_names()` function. This parameter accepts a vector of column names that are intended to be kept unchanged.

::::::::::::::::::::::::::::::::::::: challenge

Standardize the column names of the input dataset, but keep the “V1” column as it is.

::::::::::::::::::::::::::::::::::::::::::::::::

### Removing irregularities

Raw data may contain irregularities such as duplicated and empty rows and columns, as well as constant columns. `remove_duplicates` and `remove_constants` functions from `{cleanepi}`  remove such irregularities as demonstrated in the below code chunk. 


``` r
sim_ebola_data <- cleanepi::remove_constants(sim_ebola_data)
sim_ebola_data <- cleanepi::remove_duplicates(sim_ebola_data)
```

Note that, our simulated Ebola does not contain duplicated nor constant rows or columns. 

### Replacing missing values

In addition to the regularities, raw data can contain missing values that may be encoded by different strings, including the empty. To ensure robust analysis, it is a good practice to replace all missing values by `NA` in the entire dataset. Below is a code snippet demonstrating how you can achieve this in `{cleanepi}`:


``` r
sim_ebola_data <- cleanepi::replace_missing_values(
  data = sim_ebola_data,
  na_strings = ""
)
```

### Validating subject IDs

Each entry in the dataset represents a subject and should be distinguishable by a specific column formatted in a particular way, such as falling within a specified range, containing certain prefixes and/or suffixes, containing a specific number of characters. The `{cleanepi}` package offers the `check_subject_ids` function designed precisely for this task as shown in the below code chunk. This function validates whether they are unique and meet the required criteria.



``` r
sim_ebola_data <-
  cleanepi::check_subject_ids(
    data = sim_ebola_data,
    target_columns = "case_id",
    range = c(0, 15000)
  )
```

``` output
Found 1957 duplicated rows. Please consult the report for more details.
```

``` warning
Warning: Detected incorrect subject ids at lines: 
Use the correct_subject_ids() function to adjust them.
```

Note that our simulated  dataset does contain duplicated subject IDS.

### Standardizing dates

Certainly an epidemic dataset contains date columns for different events, such as the date of infection, date of symptoms onset, ..etc, and these dates can come in different date forms, and it good practice to unify them. The `{cleanepi}` package provides functionality for converting date columns in epidemic datasets into ISO format, ensuring consistency across the different date columns. Here's how you can use it on our simulated dataset:


``` r
sim_ebola_data <- cleanepi::standardize_dates(
  sim_ebola_data,
  target_columns = c(
    "date_onset",
    "date_sample"
  )
)

sim_ebola_data
```

``` output
# A tibble: 15,000 × 7
     v_1 case_id age         gender status    date_onset date_sample
   <int> <chr>   <chr>       <chr>  <chr>     <date>     <date>     
 1     1 14905   90          1      confirmed 2015-03-15 2015-04-06 
 2     2 13043   twenty-five 2      <NA>      NA         2014-01-03 
 3     3 14364   54          f      <NA>      2014-02-09 2015-03-03 
 4     4 14675   ninety      <NA>   <NA>      2014-10-19 2014-12-31 
 5     5 12648   74          F      <NA>      2014-06-08 2016-10-10 
 6     6 14274   seventy-six female <NA>      NA         2016-01-23 
 7     7 14132   sixteen     male   confirmed NA         2015-10-05 
 8     8 14715   44          f      confirmed NA         2016-04-24 
 9     9 13435   26          1      <NA>      2014-07-09 2014-09-20 
10    10 14816   thirty      f      <NA>      2015-06-29 2015-02-06 
# ℹ 14,990 more rows
```

This function coverts the values in the target columns, or will automatically figure out the date columns within the dataset (if `target_columns = NULL`) and convert them into the **Ymd**  format.

### Converting to numeric values

In the raw dataset, some column can come with mixture of character and numerical values, and you want to covert the character values explicitly into numeric. For example, in our simulated data set, in the age column some entries are written in words. 
The `convert_to_numeric()` function in `{cleanepi}` does such conversion as illustrated in the below code chunk.

``` r
sim_ebola_data <- cleanepi::convert_to_numeric(sim_ebola_data,
  target_columns = "age"
)

sim_ebola_data
```

``` output
# A tibble: 15,000 × 7
     v_1 case_id   age gender status    date_onset date_sample
   <int> <chr>   <dbl> <chr>  <chr>     <date>     <date>     
 1     1 14905      90 1      confirmed 2015-03-15 2015-04-06 
 2     2 13043      25 2      <NA>      NA         2014-01-03 
 3     3 14364      54 f      <NA>      2014-02-09 2015-03-03 
 4     4 14675      90 <NA>   <NA>      2014-10-19 2014-12-31 
 5     5 12648      74 F      <NA>      2014-06-08 2016-10-10 
 6     6 14274      76 female <NA>      NA         2016-01-23 
 7     7 14132      16 male   confirmed NA         2015-10-05 
 8     8 14715      44 f      confirmed NA         2016-04-24 
 9     9 13435      26 1      <NA>      2014-07-09 2014-09-20 
10    10 14816      30 f      <NA>      2015-06-29 2015-02-06 
# ℹ 14,990 more rows
```

## Epidemiology related operations

In addition to common data cleansing tasks, such as those discussed in the above section, the `{cleanepi}` package offers 
additional functionalities tailored specifically for processing and analyzing outbreak and epidemic data. This section 
covers some of these specialized tasks.

### Checking sequence of dated-events

Ensuring the correct order and sequence of dated events is crucial in epidemiological data analysis, especially 
when analyzing infectious diseases where the timing of events like symptom onset and sample collection is essential. 
The `{cleanepi}` package provides a helpful function called `check_date_sequence()` precisely for this purpose.

Here's an example code chunk demonstrating the usage of `check_date_sequence()` function in our simulated Ebola dataset


``` r
sim_ebola_data <- cleanepi::check_date_sequence(
  data = sim_ebola_data,
  target_columns = c("date_onset", "date_sample")
)
```

This functionality is crucial for ensuring data integrity and accuracy in epidemiological analyses, as it helps identify 
any inconsistencies or errors in the chronological order of events, allowing yor to address them appropriately.

### Dictionary-based substitution

In the realm of data pre-processing, it's common to encounter scenarios where certain columns in a dataset, such as the “gender” column in our simulated Ebola dataset,
are expected to have specific values or factors. However, it's also common for unexpected or erroneous values to appear in these columns, which need to be replaced with appropriate values. The `{cleanepi}` package offers support for dictionary-based substitution, a method that allows you to replace values in specific columns based on mappings defined in a dictionary. 
This approach ensures consistency and accuracy in data cleaning.

Moreover, `{cleanepi}` provides a built-in dictionary specifically tailored for epidemiological data. The example dictionary below includes mappings for the “gender” column.


``` r
test_dict <- base::readRDS(
  system.file("extdata", "test_dict.RDS", package = "cleanepi")
)
base::print(test_dict)
```

``` output
  options values    grp orders
1       1   male gender      1
2       2 female gender      2
3       M   male gender      3
4       F female gender      4
5       m   male gender      5
6       f female gender      6
```

Now, we can use this dictionary to standardize values of the the “gender” column according to predefined categories. Below is an example code chunk demonstrating how to utilize this functionality:


``` r
sim_ebola_data <- cleanepi::clean_using_dictionary(
  sim_ebola_data,
  dictionary = test_dict
)

sim_ebola_data
```

``` output
# A tibble: 15,000 × 7
     v_1 case_id   age gender status    date_onset date_sample
   <int> <chr>   <dbl> <chr>  <chr>     <date>     <date>     
 1     1 14905      90 male   confirmed 2015-03-15 2015-04-06 
 2     2 13043      25 female <NA>      NA         2014-01-03 
 3     3 14364      54 female <NA>      2014-02-09 2015-03-03 
 4     4 14675      90 <NA>   <NA>      2014-10-19 2014-12-31 
 5     5 12648      74 female <NA>      2014-06-08 2016-10-10 
 6     6 14274      76 female <NA>      NA         2016-01-23 
 7     7 14132      16 male   confirmed NA         2015-10-05 
 8     8 14715      44 female confirmed NA         2016-04-24 
 9     9 13435      26 male   <NA>      2014-07-09 2014-09-20 
10    10 14816      30 female <NA>      2015-06-29 2015-02-06 
# ℹ 14,990 more rows
```

This approach simplifies the data cleaning process, ensuring that categorical data in epidemiological datasets is accurately categorized and ready for further analysis.

> Note that, when the column in the dataset contains values that are not in the dictionary, the clean_using_dictionary() will raise an error. Users can use the cleanepi::add_to_dictionary() function to include the missing value into the dictionary. See the corresponding section in the package [vignette](https://epiverse-trace.github.io/cleanepi/articles/cleanepi.html) for more details.

### Calculating time span between different date events

In epidemiological data analysis it is also useful to track and analyze time-dependent events, such as the progression of a disease outbreak or the duration between sample collection and analysis.
The `{cleanepi}` package  offers a convenient function for calculating the time elapsed between two dated events at different time scales. For example, the below code snippet utilizes the `span()` function to compute the time elapsed since the date of sample for the case identified
 until the date this document was generated (2024-09-24).
 

``` r
sim_ebola_data <- cleanepi::timespan(
  sim_ebola_data,
  target_column = "date_sample",
  end_date = Sys.Date(),
  span_unit = "years",
  span_column_name = "time_since_sampling_date",
  span_remainder_unit = "months"
)

sim_ebola_data
```

``` output
# A tibble: 15,000 × 9
     v_1 case_id   age gender status    date_onset date_sample
   <int> <chr>   <dbl> <chr>  <chr>     <date>     <date>     
 1     1 14905      90 male   confirmed 2015-03-15 2015-04-06 
 2     2 13043      25 female <NA>      NA         2014-01-03 
 3     3 14364      54 female <NA>      2014-02-09 2015-03-03 
 4     4 14675      90 <NA>   <NA>      2014-10-19 2014-12-31 
 5     5 12648      74 female <NA>      2014-06-08 2016-10-10 
 6     6 14274      76 female <NA>      NA         2016-01-23 
 7     7 14132      16 male   confirmed NA         2015-10-05 
 8     8 14715      44 female confirmed NA         2016-04-24 
 9     9 13435      26 male   <NA>      2014-07-09 2014-09-20 
10    10 14816      30 female <NA>      2015-06-29 2015-02-06 
# ℹ 14,990 more rows
# ℹ 2 more variables: time_since_sampling_date <dbl>, remainder_months <dbl>
```

After executing the `span()` function, two new columns named `time_since_sampling_date` and `remainder_months` are added to the **sim_ebola_data** dataset, containing the calculated time elapsed since the date of sampling for each case, measured in years, and the remaining time measured in months.

## Multiple operations at once

Performing data cleaning operations individually can be time-consuming and error-prone. The `{cleanepi}` package simplifies this process by offering a convenient wrapper function called `clean_data()`, which allows you to perform multiple operations at once.

The `clean_data()` function applies a series of predefined data cleaning operations to the input dataset. Here's an example code chunk illustrating how to use `clean_data()` on a raw simulated Ebola dataset:


Further more, you can combine multiple data cleaning tasks via the pipe operator in "%>%", as shown in the below code snippet. 

``` r
# PERFORM THE OPERATIONS USING THE pipe SYNTAX
cleaned_data <- raw_ebola_data %>%
  cleanepi::standardize_column_names() %>%
  cleanepi::replace_missing_values(na_strings = "") %>%
  cleanepi::remove_constants(cutoff = 1.0) %>%
  cleanepi::remove_duplicates(target_columns = NULL) %>%
  cleanepi::standardize_dates(
    target_columns = c("date_onset", "date_sample"),
    error_tolerance = 0.4,
    format = NULL,
    timeframe = NULL
  ) %>%
  cleanepi::check_subject_ids(
    target_columns = "case_id",
    range = c(1, 15000)
  ) %>%
  cleanepi::convert_to_numeric(target_columns = "age") %>%
  cleanepi::clean_using_dictionary(dictionary = test_dict)
```

``` output
Found 1957 duplicated rows. Please consult the report for more details.
```

``` warning
Warning: Detected incorrect subject ids at lines: 
Use the correct_subject_ids() function to adjust them.
```

## Printing the clean report

The `{cleanepi}` package generates a comprehensive report detailing the findings and actions of all data cleansing 
operations conducted during the analysis. This report is presented as a webpage with multiple sections. Each section 
corresponds to a specific data cleansing operation, and clicking on each section allows you to access the results of 
that particular operation. This interactive approach enables users to efficiently review and analyze the outcomes of 
individual cleansing steps within the broader data cleansing process.

You can view the report using `cleanepi::print_report()` function. 


<p><figure>
    <img src="fig/report_demo.png"
         alt="Data cleaning report" 
         width="600"/> 
    <figcaption>
            <p>Example of data cleaning report generated by `{cleanepi}`</p>
    </figcaption>
</figure>

## Validating and tagging case data

In outbreak analysis, once you have completed the initial steps of reading and cleaning the case data,
it's essential to establish an additional foundation layer to ensure the integrity and reliability of subsequent
analyses. Specifically, this involves verifying the presence and correct data type of certain input columns within
your dataset, a process commonly referred to as "tagging." Additionally, it's crucial to implement measures to 
validate that these tagged columns are not inadvertently deleted during further data processing steps.

This is achieved by converting the cleaned case data into a `linelist` object using `{linelist}` package, see the 
below code chunk.


``` r
library(linelist)

linelist_data <- linelist::make_linelist(
  x = cleaned_data,
  id = "case_id",
  date_onset = "date_onset",
  gender = "gender"
)

linelist_data
```

``` output

// linelist object
# A tibble: 15,000 × 7
     v_1 case_id   age gender status    date_onset date_sample
   <int> <chr>   <dbl> <chr>  <chr>     <date>     <date>     
 1     1 14905      90 male   confirmed 2015-03-15 2015-04-06 
 2     2 13043      25 female <NA>      NA         2014-01-03 
 3     3 14364      54 female <NA>      2014-02-09 2015-03-03 
 4     4 14675      90 <NA>   <NA>      2014-10-19 2014-12-31 
 5     5 12648      74 female <NA>      2014-06-08 2016-10-10 
 6     6 14274      76 female <NA>      NA         2016-01-23 
 7     7 14132      16 male   confirmed NA         2015-10-05 
 8     8 14715      44 female confirmed NA         2016-04-24 
 9     9 13435      26 male   <NA>      2014-07-09 2014-09-20 
10    10 14816      30 female <NA>      2015-06-29 2015-02-06 
# ℹ 14,990 more rows

// tags: id:case_id, date_onset:date_onset, gender:gender 
```

The `{linelist}` package supplies tags for common epidemiological variables 
and a set of appropriate data types for each. You can view the list of available tags by the variable name
and their acceptable data types for each using `linelist::tags_types()`.


::::::::::::::::::::::::::::::::::::: challenge 

Let's **tag** more variables. In new datasets, it will be frequent to have variable names different to the available tag names. However, we can associate them based on how variables were defined for data collection.

Now:

- **Explore** the available tag names in {linelist}.
- **Find** what other variables in the cleaned dataset can be associated with any of these available tags.
- **Tag** those variables as above using `linelist::make_linelist()`.

:::::::::::::::::::: hint

Your can get access to the list of available tag names in {linelist} using:


``` r
# Get a list of available tags by name and data types
linelist::tags_types()

# Get a list of names only
linelist::tags_names()
```

:::::::::::::::::::::::

::::::::::::::::: solution


``` r
linelist::make_linelist(
  x = cleaned_data,
  id = "case_id",
  date_onset = "date_onset",
  gender = "gender",
  age = "age", # same name in default list and dataset
  date_reporting = "date_sample" # different names but related
)
```

How these additional tags are visible in the output? 

<!-- Do you want to see a display of available and tagged variables? You can explore the function `linelist::tags()` and read its [reference documentation](https://epiverse-trace.github.io/linelist/reference/tags.html). -->

::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::

To ensure that all tagged variables are standardized and have the correct data 
types, use the `linelist::validate_linelist()`, as 
shown in the example below:

```r
linelist::validate_linelist(linelist_data)
```

<!-- If your dataset requires a new tag, set the argument -->
<!-- `allow_extra = TRUE` when creating the linelist object with its corresponding-->
<!-- datatype. -->



::::::::::::::::::::::::: challenge

Let's **validate** tagged variables. Let's simulate that in an ongoing outbreak; the next day, your data has a new set of entries (i.e., rows or observations) but one variable change of data type. 

For example, the variable:
- `age` changes of type from a double (`<dbl>`) variable to character (`<chr>`),

To simulate it:

- **Change** the variable data type,
- **Tag** the variable into a linelist, and then 
- **Validate** it.

Describe how `linelist::validate_linelist()` reacts when input data has a different variable data type.

:::::::::::::::::::::::::: hint

We can use `dplyr::mutate()` to change the variable type before tagging for validation. For example:


``` r
cleaned_data %>%
  # simulate a change of data type in one variable
  dplyr::mutate(age = as.character(age)) %>%
  # tag one variable
  linelist::... %>%
  # validate the linelist
  linelist::...
```

::::::::::::::::::::::::::

:::::::::::::::::::::::::: hint

> Please run the code line by line, focusing only on the parts before the pipe (`%>%`). After each step, observe the output before moving to the next line.

If the `age` variable changes from double (`<dbl>`) to character (`<chr>`) we get the following:


``` r
cleaned_data %>%
  # simulate a change of data type in one variable
  dplyr::mutate(age = as.character(age)) %>%
  # tag one variable
  linelist::make_linelist(
    age = "age"
  ) %>%
  # validate the linelist
  linelist::validate_linelist()
```

``` error
Error: Some tags have the wrong class:
  - age: Must inherit from class 'numeric'/'integer', but has class 'character'
```

Why are we getting an `Error` message?

<!-- Should we have a `Warning` message instead? Explain why. -->

Explore other situations to understand this behavior. Let's try these additional changes to variables:

- `date_onset` changes from a `<date>` variable to character (`<chr>`), 
- `gender` changes from a character (`<chr>`) variable to integer (`<int>`).

Then tag them into a linelist for validation. Does the `Error` message propose to us the solution?

::::::::::::::::::::::::::

::::::::::::::::::::::::: solution


``` r
# Change 2
# Run this code line by line to identify changes
cleaned_data %>%
  # simulate a change of data type
  dplyr::mutate(date_onset = as.character(date_onset)) %>%
  # tag
  linelist::make_linelist(
    date_onset = "date_onset"
  ) %>%
  # validate
  linelist::validate_linelist()
```



``` r
# Change 3
# Run this code line by line to identify changes
cleaned_data %>%
  # simulate a change of data type
  dplyr::mutate(gender = as.factor(gender)) %>%
  dplyr::mutate(gender = as.integer(gender)) %>%
  # tag
  linelist::make_linelist(
    gender = "gender"
  ) %>%
  # validate
  linelist::validate_linelist()
```

We get `Error` messages because of the mismatch between the predefined tag type (from `linelist::tags_types()`) and the tagged variable class in the linelist.

The `Error` message inform us that in order to **validate** our linelist, we must fix the input variable type to fit the expected tag type. In a data analysis script, we can do this by adding one cleaning step into the pipeline.

::::::::::::::::::::::::: 

:::::::::::::::::::::::::

::::::::::::::::::::::::: discussion

Have you ever experienced an unexpected change of variable type when running a lengthy analysis during an emergency response? What actions did you take to overcome this inconvenience?

Imagine you automated your analysis to read your date directly from source, but the people in charge of the data collection decided to remove a variable you found useful. What step along the `{linelist}` workflow of tagging and validating would response to the absence of a variable?

:::::::::::::::::::::::::

:::::::::::::::::::::::::: instructor

If learners do not have an experience to share, we as instructors can share one.

An scenario like this usually happens when the institution doing the analysis is not the same as the institution collecting the data. The later can make decisions about the data structure that can affect downstream processes, impacting the time or the accuracy of the analysis results.

About losing variables, you can suggest learners to simulate this scenario:


``` r
cleaned_data %>%
  # simulate a change of data type in one variable
  select(-age) %>%
  # tag one variable
  linelist::make_linelist(
    age = "age"
  )
```

``` error
Error in base::tryCatch(base::withCallingHandlers({: 1 assertions failed:
 * Variable 'tag': Must be element of set
 * {'v_1','case_id','gender','status','date_onset','date_sample'}, but
 * is 'age'.
```

::::::::::::::::::::::::::

Safeguarding is implicitly built into the linelist objects. If you try to drop any of the tagged 
columns, you will receive an error or warning message, as shown in the example below.


``` r
new_df <- linelist_data %>%
  dplyr::select(case_id, gender)
```

``` warning
Warning: The following tags have lost their variable:
 date_onset:date_onset
```

This `Warning` message above is the default output option when we lose tags in a `linelist` object. However, it can be changed to an `Error` message using `linelist::lost_tags_action()`. 

::::::::::::::::::::::::::::::::::::: challenge 

Let's test the implications of changing the **safeguarding** configuration from a `Warning` to an `Error` message.

- First, run this code to count the frequency per category within a categorical variable:


``` r
linelist_data %>%
  dplyr::select(case_id, gender) %>%
  dplyr::count(gender)
```

- Set behavior for lost tags in a `linelist` to "error" as follows:


``` r
# set behavior to "error"
linelist::lost_tags_action(action = "error")
```
- Now, re-run the above code segment with `dplyr::count()`.

Identify:

- What is the difference in the output between a `Warning` and an `Error`?
- What could be the implications of this change for your daily data analysis pipeline during an outbreak response?

:::::::::::::::::::::::: solution

Deciding between `Warning` or `Error` message will depend on the level of attention or flexibility you need when losing tags. One will alert you about a change but will continue running the code downstream. The other will stop your analysis pipeline and the rest will not be executed. 

A data reading, cleaning and validation script may require a more stable or fixed pipeline. An exploratory data analysis may require a more flexible approach. These two processes can be isolated in different scripts or repositories to adjust the safeguarding according to your needs.

Before you continue, set the configuration back again to the default option of `Warning`:


``` r
# set behavior to the default option: "warning"
linelist::lost_tags_action()
```

``` output
Lost tags will now issue a warning.
```

::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

A  `linelist` object resembles a data frame but offers richer features 
and functionalities. Packages that are linelist-aware can leverage these 
features. For example, you can extract a data frame of only the tagged columns 
using the `linelist::tags_df()` function, as shown below:


``` r
linelist::tags_df(linelist_data)
```

``` output
# A tibble: 15,000 × 3
   id    date_onset gender
   <chr> <date>     <chr> 
 1 14905 2015-03-15 male  
 2 13043 NA         female
 3 14364 2014-02-09 female
 4 14675 2014-10-19 <NA>  
 5 12648 2014-06-08 female
 6 14274 NA         female
 7 14132 NA         male  
 8 14715 NA         female
 9 13435 2014-07-09 male  
10 14816 2015-06-29 female
# ℹ 14,990 more rows
```

This allows, the extraction of use tagged-only columns in downstream analysis, which will be useful for the next episode!

:::::::::::::::::::::::::::::::::::: callout

### When I should use `{linelist}`?

Data analysis during an outbreak response or mass-gathering surveillance demands a different set of "data safeguards" if compared to usual research situations. For example, your data will change or be updated over time (e.g. new entries, new variables, renamed variables).

`{linelist}` is more appropriate for this type of ongoing or long-lasting analysis.
Check the "Get started" vignette section about
[When you should consider using {linelist}?](https://epiverse-trace.github.io/linelist/articles/linelist.html#should-i-use-linelist) for more information.

:::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints 

- Use `{cleanepi}` package to clean and standardize epidemic and outbreak data
- Use `{linelist}` to tagg, validate, and prepare case data for downstream analysis.

::::::::::::::::::::::::::::::::::::::::::::::::
