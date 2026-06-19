---
title: "Preparing Your Card Sorting Data File"
description: "Learn how to format your own card sorting data so it can be analyzed with HCA, dendrograms, and MDS in R."
---

# Preparing Your Card Sorting Data File

Before running the R script, your data must be organized in a specific format.

The example used in this tutorial is a grocery card sorting dataset, but the same method can be used with many types of card sorting studies. Your cards might be:

- Website pages
- Menu labels
- Product categories
- Help center topics
- App settings
- Dashboard features
- Grocery items
- Any other content you want users to organize

The important part is not the topic of the dataset.

The important part is the structure of the file.

---

## What You Will Learn

By the end of this page, you will know how to:

- Prepare your own card sorting data file
- Use the example grocery dataset as a practice file
- Rename your file so it works with the tutorial script
- Structure the spreadsheet correctly
- Check whether your data is ready for analysis in R

This page prepares your data for the next steps: distance matrices, hierarchical cluster analysis, dendrograms, and multidimensional scaling.

---

## Standard File Name

For this tutorial, we will use the following standard file name:

```text
card_sort_data.csv
```

To follow the tutorial exactly, rename your prepared file:

```text
card_sort_data.csv
```

Place this file in the same folder as your R script.

This keeps the tutorial simple because the R script always looks for the same file name.

---

## Using the Example Grocery Dataset

The grocery dataset provided in this tutorial is only an example.

You can use it to practice the workflow before applying the method to your own project.

In the example dataset:

- The cards are grocery items.
- The categories are participant-created grocery categories.
- The numbers show how often items were associated with those categories.
- The goal is to identify possible information architecture categories for a grocery website.

You may see the example file named something like:

```text
card_sort_data_example_grocery.csv
```

To run the tutorial script without changing any code, make a copy of the file and rename the copy:

```text
card_sort_data.csv
```

Later, when you are ready to analyze your own data, replace the example file with your own prepared file using the same name:

```text
card_sort_data.csv
```

---

## If You Want to Keep Your Own File Name

You do not have to rename your file if you prefer to keep the original name.

Instead, change this line in the R script:

```r
data_file <- "card_sort_data.csv"
```

For example, if your file is called:

```text
my_website_cards.csv
```

change the R code to:

```r
data_file <- "my_website_cards.csv"
```

For beginners, renaming the file to `card_sort_data.csv` is usually easier because it avoids file-name errors.

---

## Required File Structure

Your file should be organized as an **item-by-category table**.

Each row should represent one card or item.

Each column should represent one category.

The first column must be called:

```text
Name
```

This column contains the names of the cards.

All other columns should contain numbers.

A simple example might look like this:

| Name | Fruits | Vegetables | Dairy | Snacks | Frozen Foods |
|---|---:|---:|---:|---:|---:|
| Apples | 1 | 0 | 0 | 0 | 0 |
| Bananas | 1 | 0 | 0 | 0 | 0 |
| Broccoli | 0 | 1 | 0 | 0 | 0 |
| Yogurt | 0 | 0 | 1 | 0 | 0 |
| Chips | 0 | 0 | 0 | 1 | 0 |
| Frozen peas | 0 | 1 | 0 | 0 | 1 |

In this structure:

- Rows are the cards.
- Columns are the categories.
- Numbers show whether, or how often, each card was placed in each category.
- A value of `0` means the card was not placed in that category.
- A value greater than `0` means the card was placed in that category by one or more participants.

---

## What the Numbers Mean

The numbers in the table can represent counts.

For example, imagine 7 participants completed the card sort.

If 5 participants placed **Apples** in a category called **Fruits**, the value would be:

```text
5
```

If no participant placed **Apples** in **Dairy**, the value would be:

```text
0
```

A simplified version might look like this:

| Name | Fruits | Vegetables | Dairy |
|---|---:|---:|---:|
| Apples | 5 | 2 | 0 |
| Bananas | 7 | 0 | 0 |
| Yogurt | 0 | 0 | 7 |

This tells us that apples and bananas have similar category patterns, while yogurt has a different pattern.

The R script uses these patterns to understand which cards are similar to each other.

---

## Why the File Must Be Organized This Way

The analysis compares cards based on their category patterns.

For example, imagine these two rows:

| Name | Fruits | Vegetables | Dairy | Snacks |
|---|---:|---:|---:|---:|
| Apples | 6 | 1 | 0 | 0 |
| Bananas | 7 | 0 | 0 | 0 |

Apples and bananas have similar patterns because both are mostly associated with fruit-related categories.

Now compare apples with yogurt:

| Name | Fruits | Vegetables | Dairy | Snacks |
|---|---:|---:|---:|---:|
| Apples | 6 | 1 | 0 | 0 |
| Yogurt | 0 | 0 | 7 | 0 |

These patterns are very different, so the analysis will treat apples and yogurt as less similar.

This is the foundation of the distance matrix, HCA, dendrograms, and MDS plots used later in the tutorial.

---

## If You Export Data from a Card Sorting Platform

Different tools export card sorting data in different formats.

Your export may not look exactly like the required format at first.

For this tutorial, your final file should still follow this structure:

```text
Name,Category 1,Category 2,Category 3,Category 4
```

For example:

```text
Name,Fruits,Vegetables,Dairy,Snacks
Apples,6,1,0,0
Bananas,7,0,0,0
Yogurt,0,0,7,0
Chips,0,0,0,6
```

If your platform exports raw participant-level data, you may need to summarize it first.

The goal is to create one row per card and one column per category.

---

## From Raw Sorting Data to the Required Format

Some card sorting platforms export data participant by participant.

For example, you may see something like this:

| Participant | Card | Category |
|---|---|---|
| P1 | Apples | Fruits |
| P1 | Bananas | Fruits |
| P1 | Yogurt | Dairy |
| P2 | Apples | Produce |
| P2 | Bananas | Produce |
| P2 | Yogurt | Dairy |

This format is useful, but it is not the final format needed for this tutorial.

You would first summarize it into an item-by-category table:

| Name | Fruits | Produce | Dairy |
|---|---:|---:|---:|
| Apples | 1 | 1 | 0 |
| Bananas | 1 | 1 | 0 |
| Yogurt | 0 | 0 | 2 |

This summarized table is the file you will save as:

```text
card_sort_data.csv
```

---

## Checklist Before Running the R Script

Before you run the code, check that:

- Your file is saved as a `.csv` file.
- The file is named `card_sort_data.csv`, unless you change the file name in the script.
- The first column is called `Name`.
- Each row contains one card or item.
- Each remaining column is a category.
- Category columns contain only numbers.
- Empty cells have been replaced with `0`.
- There are no extra notes, comments, or blank rows at the bottom of the file.
- The file is saved in the same folder as the R script.

---

## Starter R Code for Loading Your File

Use this code at the beginning of the analysis script.

```r
# ============================================================
# MeasureUX Card Sorting Analysis
# Data Preparation
# ============================================================

# The tutorial uses card_sort_data.csv as the standard file name.
# You can either rename your file to card_sort_data.csv
# or change the file name below.

data_file <- "card_sort_data.csv"

raw_data <- read.csv(data_file, check.names = FALSE)

# Check that the required Name column exists
if (!"Name" %in% names(raw_data)) {
  stop("Your file must have a first column called 'Name'. This column should contain the card/item names.")
}

# Separate item names from the category data
item_names <- raw_data$Name
item_category_matrix <- raw_data[, names(raw_data) != "Name"]

# Convert category columns to numeric
item_category_matrix <- as.data.frame(
  lapply(item_category_matrix, as.numeric)
)

# Replace missing values with 0
item_category_matrix[is.na(item_category_matrix)] <- 0

# Use item names as row labels
rownames(item_category_matrix) <- make.unique(item_names)

# Quick checks
cat("Number of cards/items:", nrow(item_category_matrix), "\n")
cat("Number of categories:", ncol(item_category_matrix), "\n")

head(item_category_matrix)
```

---

## What This Code Does

This code prepares your file for the rest of the analysis.

It does five important things.

First, it loads the CSV file:

```r
raw_data <- read.csv(data_file, check.names = FALSE)
```

Second, it checks that the file has a column called `Name`.

```r
if (!"Name" %in% names(raw_data)) {
  stop("Your file must have a first column called 'Name'.")
}
```

Third, it separates the card names from the category data.

```r
item_names <- raw_data$Name
item_category_matrix <- raw_data[, names(raw_data) != "Name"]
```

Fourth, it makes sure the category columns are numeric.

```r
item_category_matrix <- as.data.frame(
  lapply(item_category_matrix, as.numeric)
)
```

Fifth, it uses the card names as row labels.

```r
rownames(item_category_matrix) <- make.unique(item_names)
```

This means the item names will appear later in the dendrogram and MDS plot.

---

## Common File Problems

### Problem 1: The first column is not called `Name`

The R script expects the first column to be called:

```text
Name
```

Not:

```text
Item
Card
Card Name
Label
Product
Page
```

Rename the first column to `Name`.

---

### Problem 2: Some cells are blank

Blank cells can cause problems.

Replace blank cells with:

```text
0
```

A blank cell might mean the item was not placed in that category, but R does not always know that.

Using `0` makes the meaning clear.

---

### Problem 3: Category columns contain text

The category columns should contain numbers only.

This is correct:

| Name | Fruits | Dairy |
|---|---:|---:|
| Apples | 5 | 0 |
| Yogurt | 0 | 6 |

This is not correct:

| Name | Fruits | Dairy |
|---|---|---|
| Apples | yes | no |
| Yogurt | no | yes |

If your file uses words like `yes`, `no`, `true`, or `false`, convert them to numbers before running the analysis.

---

### Problem 4: There are extra notes in the file

Do not include notes, comments, totals, or explanations inside the CSV file.

The file should contain only:

- One header row
- One row per card
- One column called `Name`
- Category columns with numeric values

---

### Problem 5: The file is not in the same folder as the R script

If R cannot find the file, first check that:

- The file name is spelled correctly.
- The file ends in `.csv`.
- The file is in the same folder as the R script.

---

## Applying This to Your Own UX Project

The grocery dataset is only for practice.

For your own project, replace the grocery items with your own cards.

For example, if you are studying a university website, your cards might be:

- Tuition and Fees
- Academic Calendar
- Housing
- Student Clubs
- Advising
- Financial Aid
- Course Registration
- Career Services

If you are studying an app, your cards might be:

- Account Settings
- Notifications
- Payment Methods
- Search Filters
- Favorites
- Help Center
- Order History

The same analysis can be used as long as the data follows the required file structure.

---

## Key Takeaways

The tutorial uses `card_sort_data.csv` as the standard file name.

The grocery dataset is only an example.

Your own data can be used if it follows the same structure.

Each row should represent one card.

Each category should be a column.

The first column must be called `Name`.

All category columns should contain numeric values.

Once your data is prepared, you can use the same R script to create a distance matrix, HCA dendrogram, and MDS plot.
