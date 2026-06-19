---
layout: post
title: Analyzing Card Sort Data with HCA, Dendrograms, and MDS
subtitle: How to create and compare clustering visualizations for information architecture
tags: [R script, Card Sorting, HCA, Dendrograms, MDS, Information Architecture]
---

# Analyzing Card Sort Data with HCA, Dendrograms, and MDS

In the previous page, you learned how to prepare your card sorting data file.

This page shows you how to analyze that file using:

- A distance matrix
- Hierarchical Cluster Analysis, or HCA
- Different dendrogram methods
- Multidimensional Scaling, or MDS
- Cluster assignment tables

The goal is not simply to run the code.

The goal is to create outputs that help you make information architecture decisions.

---

## What You Will Learn

By the end of this page, you will be able to:

- Explain what HCA does in plain language
- Understand why different dendrograms can produce different groupings
- Compare clustering methods
- Choose a starting number of clusters
- Create a dendrogram
- Create an MDS plot
- Export a table showing which cards belong to which cluster

This tutorial uses the same standard file name from the previous page:

```text
card_sort_data.csv
```

The grocery dataset is used as an example, but the same code can be used with your own card sorting data if your file follows the required structure.

---

## Where This Page Fits in the Workflow

Card sorting analysis has three major phases.

### Phase 1: Prepare the data

You organize your file so that:

- Each row is one card or item.
- The first column is called `Name`.
- Each remaining column is a category.
- The category columns contain numbers.

This was covered in the previous page.

### Phase 2: Generate the analysis outputs

This is what we do on this page.

We will create:

- A distance matrix
- Several dendrograms
- A method comparison table
- A selected dendrogram with clusters
- An MDS plot
- A cluster assignment table

### Phase 3: Interpret the outputs

This comes after the code.

You will ask:

- How many clusters make sense?
- Which items belong together?
- Which items seem ambiguous?
- What should the categories be called?
- How should these results inform the website or app structure?

That interpretation step is important enough to have its own page.

---

# Part 1: Understanding the Main Ideas

Before running the code, it helps to understand what each method is doing.

---

## What Is HCA?

HCA stands for **Hierarchical Cluster Analysis**.

In card sorting, HCA helps us see which cards are grouped in similar ways by participants.

Imagine each card is a puzzle piece.

HCA helps us see which puzzle pieces seem to fit together.

For example, in a grocery card sort, participants may often place these items together:

- Apples
- Bananas
- Strawberries
- Oranges

HCA may suggest that these items belong in the same group because their sorting patterns are similar.

In a website project, the same logic applies.

If users often group several pages together, HCA may suggest that those pages belong in the same navigation category.

---

## Comic Checkpoint: HCA as a Puzzle

In the HF751 comic, HCA is introduced through a student club example.

The university can support only a few new clubs, but students requested many possible clubs. HCA helps identify which clubs seem to belong together based on student preferences.

This is a helpful way to think about card sorting analysis.

HCA does not make the final decision for you.

It helps you see possible groupings.

---

## What Is a Distance Matrix?

Before R can build clusters, it needs to compare every card with every other card.

This comparison is stored in a **distance matrix**.

A distance matrix answers questions like:

- How similar are apples and bananas?
- How similar are apples and yogurt?
- How similar are coffee and tea?
- How similar are frozen peas and frozen pizza?

In this tutorial, cards are compared based on their category patterns.

If two cards were placed in similar categories, they are treated as close.

If two cards were placed in very different categories, they are treated as far apart.

---

## What Is a Dendrogram?

A dendrogram is a tree-like diagram.

It shows how cards are grouped together step by step.

Cards that join together lower in the tree are more similar.

Cards that join together higher in the tree are less similar.

In UX terms, a dendrogram helps you see:

- Which cards naturally belong together
- Which groups might become categories
- Which large groups might need subcategories
- Which items may be difficult to place

A dendrogram is not a final website menu.

It is evidence that helps you design one.

---

## What Is a Reference Line?

A dendrogram can be “cut” into a chosen number of clusters.

That chosen number is often called `k`.

For example:

- If `k = 3`, the dendrogram is divided into 3 clusters.
- If `k = 6`, the dendrogram is divided into 6 clusters.
- If `k = 9`, the dendrogram is divided into 9 clusters.

In the comic, a reference line is drawn across the dendrogram. Where that line crosses the branches determines the number of clusters.

This is useful, but it is not automatic.

You choose `k` based on:

- The data
- The dendrogram
- The MDS plot
- The number of cards
- The project goals
- The existing website structure
- Whether the categories would make sense to users

---

## What Is MDS?

MDS stands for **Multidimensional Scaling**.

MDS creates a map-like view of the cards.

Cards that appear close together on the map were categorized similarly.

Cards that appear far apart were categorized differently.

A dendrogram shows a hierarchy.

MDS shows a spatial map.

Both are useful.

The dendrogram helps you see nested groups.

The MDS plot helps you check whether the clusters make sense visually.

---

## Why Do We Compare Different Dendrograms?

Different clustering methods use different rules for combining cards.

This means different methods can produce slightly different dendrograms.

That is normal.

Think of the methods as different ways of asking:

> “How should we decide whether two groups are similar enough to combine?”

In this tutorial, we compare five common methods:

| Method | Beginner-friendly explanation |
|---|---|
| `complete` | Conservative. It tends to create tighter groups because it considers the farthest items in each cluster. |
| `average` | Middle-ground. It uses the average distance between items in two clusters. |
| `single` | Very flexible. It can create long chains of items and is often less useful for final UX categories. |
| `ward.D` | Tries to create compact clusters. Often useful for exploring information architecture. |
| `ward.D2` | Similar to Ward.D, but uses a slightly different version of the Ward method. |

There is no need to memorize these methods.

The important idea is this:

> We compare methods because we want a clustering solution that is supported by the data and useful for UX interpretation.

---

# Part 2: R Code for HCA, Dendrograms, and MDS

The following code assumes that your prepared CSV file is called:

```text
card_sort_data.csv
```

Place the CSV file in the same folder as your R script.

---

## Step 1: Install and Load Packages

```r
# ============================================================
# MeasureUX Card Sorting Analysis
# HCA, Dendrograms, and MDS
# ============================================================

packages <- c("ggplot2", "ggrepel", "factoextra")

for (pkg in packages) {
  if (!require(pkg, character.only = TRUE)) {
    install.packages(pkg)
    library(pkg, character.only = TRUE)
  }
}

dir.create("outputs", showWarnings = FALSE)
```

### What this code does

This code loads the packages needed for the analysis.

It also creates a folder called `outputs`.

All tables and images created by the script will be saved in that folder.

---

## Step 2: Load Your Card Sorting Data

```r
# -----------------------------
# User settings
# -----------------------------

data_file <- "card_sort_data.csv"

# Change k later after inspecting your outputs
k <- 6

# -----------------------------
# Load the data
# -----------------------------

raw_data <- read.csv(data_file, check.names = FALSE)

if (!"Name" %in% names(raw_data)) {
  stop("Your file must contain a column called 'Name'. This column should contain the card or item names.")
}

cat("Number of cards/items:", nrow(raw_data), "\n")
cat("Number of columns in the file:", ncol(raw_data), "\n")

head(raw_data)
```

### What this code does

This code loads your CSV file.

It also checks whether the file contains a column called `Name`.

The `Name` column is required because it contains the card labels that will appear in the dendrogram and MDS plot.

---

## Step 3: Prepare the Item-by-Category Matrix

```r
# -----------------------------
# Prepare the item-category matrix
# -----------------------------

item_names <- raw_data$Name

category_data <- raw_data[, names(raw_data) != "Name"]

# Convert category columns to numeric
converted_data <- lapply(category_data, function(x) {
  suppressWarnings(as.numeric(x))
})

# Check for non-numeric values that are not blank
problem_columns <- names(category_data)[
  sapply(seq_along(category_data), function(i) {
    original_values <- trimws(as.character(category_data[[i]]))
    converted_values <- converted_data[[i]]
    any(is.na(converted_values) & original_values != "" & !is.na(original_values))
  })
]

if (length(problem_columns) > 0) {
  stop(
    paste(
      "These category columns contain non-numeric values:",
      paste(problem_columns, collapse = ", "),
      "\nPlease replace text values with numbers before running the analysis."
    )
  )
}

item_category_matrix <- as.data.frame(converted_data)

# Replace blank cells with 0
item_category_matrix[is.na(item_category_matrix)] <- 0

# Use card names as row labels
rownames(item_category_matrix) <- make.unique(item_names)

# Remove cards that have no category information
empty_rows <- rowSums(item_category_matrix) == 0

if (any(empty_rows)) {
  warning("Some cards had no category information and were removed.")
  item_category_matrix <- item_category_matrix[!empty_rows, ]
}

cat("Number of cards/items used in the analysis:", nrow(item_category_matrix), "\n")
cat("Number of category columns:", ncol(item_category_matrix), "\n")

head(item_category_matrix)
```

### What this code does

This code separates the card names from the category data.

It also checks that the category columns contain numbers.

This matters because R can only calculate distances using numeric values.

---

## Step 4: Remove Columns That Cannot Help the Analysis

```r
# -----------------------------
# Remove category columns with no variation
# -----------------------------

zero_variation_columns <- sapply(item_category_matrix, function(x) {
  sd(x, na.rm = TRUE) == 0
})

if (any(zero_variation_columns)) {
  cat("Removing category columns with no variation:\n")
  print(names(item_category_matrix)[zero_variation_columns])
}

analysis_matrix <- item_category_matrix[, !zero_variation_columns, drop = FALSE]

cat("Number of category columns used after cleaning:", ncol(analysis_matrix), "\n")
```

### What this code does

Some category columns may contain the same value for every card.

For example, a category column may be all zeros.

That kind of column does not help compare cards.

This step removes those columns so the analysis can run more smoothly.

---

## Step 5: Standardize the Data

```r
# -----------------------------
# Standardize the category data
# -----------------------------

scaled_matrix <- scale(analysis_matrix)
```

### What this code does

Standardization puts all category columns on a more comparable scale.

This helps prevent one category column from having too much influence simply because its values are larger.

### Beginner-friendly explanation

R is comparing patterns across many category columns.

Standardization helps R focus on the shape of the pattern rather than only the size of the numbers.

---

## Step 6: Create the Distance Matrix

```r
# -----------------------------
# Create the distance matrix
# -----------------------------

card_distance <- dist(scaled_matrix, method = "euclidean")

# Save a visual version of the distance matrix
distance_plot <- fviz_dist(
  card_distance,
  lab_size = 5
) +
  ggtitle("Distance Matrix for Card Sorting Data")

ggsave(
  filename = "outputs/01_distance_matrix_heatmap.png",
  plot = distance_plot,
  width = 12,
  height = 10
)
```

### What this code does

This code calculates how different each card is from every other card.

It also saves a heatmap of the distance matrix.

### How to interpret the distance matrix

Look for blocks of items that appear close together.

These blocks give an early clue about possible clusters.

Do not make your final decision from the distance matrix alone.

Use it as a first look before inspecting the dendrograms and MDS plot.

---

## Step 7: Run Several HCA Methods

```r
# -----------------------------
# Run HCA with several methods
# -----------------------------

methods <- c("complete", "average", "single", "ward.D", "ward.D2")

hca_solutions <- lapply(methods, function(method_name) {
  hclust(card_distance, method = method_name)
})

names(hca_solutions) <- methods
```

### What this code does

This code creates five different dendrogram solutions.

Each method uses a different rule for grouping cards.

This allows you to compare solutions instead of trusting only one dendrogram.

---

## Step 8: Compare the HCA Methods

```r
# -----------------------------
# Compare the clustering methods
# -----------------------------

method_comparison <- data.frame(
  Method = methods,
  Cophenetic_Correlation = sapply(hca_solutions, function(hc) {
    cor(card_distance, cophenetic(hc))
  })
)

method_comparison$Cophenetic_Correlation <- round(
  method_comparison$Cophenetic_Correlation,
  3
)

method_comparison <- method_comparison[
  order(-method_comparison$Cophenetic_Correlation),
]

print(method_comparison)

write.csv(
  method_comparison,
  "outputs/02_hca_method_comparison.csv",
  row.names = FALSE
)
```

### What this code does

This code creates a table comparing the clustering methods.

The table is saved as:

```text
outputs/02_hca_method_comparison.csv
```

### How to interpret the table

The cophenetic correlation tells you how well a dendrogram preserves the original distances between cards.

In simple terms:

- Higher values are better.
- Lower values are weaker.
- The highest value is a good starting point.
- The final choice still requires interpretation.

This number helps you choose a dendrogram, but it does not replace UX judgment.

A method with a high value may still create categories that are too broad, too fragmented, or difficult to use in a real navigation menu.

---

## Step 9: Save All Dendrograms

```r
# -----------------------------
# Save one dendrogram for each method
# -----------------------------

for (method_name in methods) {
  png(
    filename = paste0("outputs/dendrogram_", method_name, ".png"),
    width = 1400,
    height = 900
  )

  plot(
    hca_solutions[[method_name]],
    cex = 0.55,
    hang = -1,
    main = paste("Dendrogram -", method_name)
  )

  dev.off()
}
```

### What this code does

This code saves one dendrogram image for each clustering method.

You can open these images and compare them visually.

### What to look for

When comparing dendrograms, ask:

- Do some methods create clearer groups than others?
- Are some dendrograms too messy?
- Are some groups too large?
- Are some groups too small?
- Do the groupings make sense for the content?
- Would these groups be useful for navigation?

---

## Step 10: Select a Method

```r
# -----------------------------
# Select the HCA method
# -----------------------------

selected_method <- method_comparison$Method[1]

# You can also manually choose a method by uncommenting one line below:
# selected_method <- "complete"
# selected_method <- "average"
# selected_method <- "ward.D"
# selected_method <- "ward.D2"

selected_hca <- hca_solutions[[selected_method]]

cat("Selected method:", selected_method, "\n")
```

### What this code does

By default, the script selects the method with the highest cophenetic correlation.

You can also choose a method manually.

This is useful when the highest-scoring method does not create the most interpretable UX categories.

---

## Step 11: Choose the Number of Clusters

At the top of the script, we used:

```r
k <- 6
```

This means the selected dendrogram will be divided into six clusters.

You should change `k` after inspecting your outputs.

For example:

```r
k <- 9
```

### How to choose k

There is no universal correct value.

A useful value of `k` should create clusters that are:

- Supported by the data
- Easy to explain
- Useful for navigation
- Not too broad
- Not too fragmented

For a small website, fewer clusters may be enough.

For a larger content set, more clusters may be needed.

The number of clusters should also make sense for the project.

For example, if a current website has 12 main menu categories, you may want to compare whether the card sort supports 12 categories or suggests a different structure.

---

## Step 12: Save the Selected Dendrogram with Cluster Boxes

```r
# -----------------------------
# Save selected dendrogram with cluster boxes
# -----------------------------

png(
  filename = paste0("outputs/03_selected_dendrogram_", selected_method, "_k", k, ".png"),
  width = 1600,
  height = 1000
)

plot(
  selected_hca,
  cex = 0.55,
  hang = -1,
  main = paste("Selected Dendrogram:", selected_method, "with k =", k)
)

rect.hclust(
  selected_hca,
  k = k,
  border = 2:(k + 1)
)

dev.off()
```

### What this code does

This code saves the selected dendrogram and draws boxes around the clusters.

The boxes show the cluster solution for your chosen value of `k`.

### How to interpret the dendrogram

Look at the cards inside each box.

Ask:

- Do these cards belong together?
- Would this cluster make sense as a category?
- Is the cluster too large?
- Is the cluster too small?
- Does the cluster need subcategories?
- Are there any cards that seem out of place?

---

## Step 13: Create a Cluster Assignment Table

```r
# -----------------------------
# Create cluster assignment table
# -----------------------------

cluster_assignments <- cutree(selected_hca, k = k)

cluster_table <- data.frame(
  Item = names(cluster_assignments),
  Cluster = cluster_assignments
)

cluster_table <- cluster_table[
  order(cluster_table$Cluster, cluster_table$Item),
]

print(cluster_table)

write.csv(
  cluster_table,
  paste0("outputs/04_cluster_assignments_", selected_method, "_k", k, ".csv"),
  row.names = FALSE
)

cluster_summary <- aggregate(
  Item ~ Cluster,
  data = cluster_table,
  FUN = function(x) paste(x, collapse = ", ")
)

write.csv(
  cluster_summary,
  paste0("outputs/04_cluster_summary_", selected_method, "_k", k, ".csv"),
  row.names = FALSE
)
```

### What this code does

This code creates two output files.

The first file lists each item and its cluster:

```text
outputs/04_cluster_assignments_METHOD_kNUMBER.csv
```

The second file summarizes the items in each cluster:

```text
outputs/04_cluster_summary_METHOD_kNUMBER.csv
```

These files are very useful for interpretation.

The dendrogram gives you the visual structure.

The cluster table gives you the practical list of items in each group.

---

## Step 14: Create the MDS Plot

```r
# -----------------------------
# Create MDS plot
# -----------------------------

mds_result <- cmdscale(card_distance, eig = TRUE, k = 2)

mds_points <- as.data.frame(mds_result$points)
colnames(mds_points) <- c("Dimension_1", "Dimension_2")

mds_data <- data.frame(
  Item = rownames(mds_points),
  Dimension_1 = mds_points$Dimension_1,
  Dimension_2 = mds_points$Dimension_2,
  Cluster = factor(cluster_assignments[rownames(mds_points)])
)

mds_plot <- ggplot(
  mds_data,
  aes(
    x = Dimension_1,
    y = Dimension_2,
    color = Cluster,
    label = Item
  )
) +
  geom_point(size = 3) +
  ggrepel::geom_text_repel(
    size = 3,
    max.overlaps = 100
  ) +
  labs(
    title = paste("MDS Plot with", k, "Clusters"),
    subtitle = paste("HCA method:", selected_method),
    x = "Dimension 1",
    y = "Dimension 2"
  ) +
  theme_minimal()

print(mds_plot)

ggsave(
  filename = paste0("outputs/05_mds_plot_", selected_method, "_k", k, ".png"),
  plot = mds_plot,
  width = 12,
  height = 9
)
```

### What this code does

This code creates an MDS plot.

Each card appears as a point on the map.

The color shows the cluster assigned from the dendrogram.

### How to interpret the MDS plot

Ask:

- Do cards in the same cluster appear close together?
- Are some clusters spread across the plot?
- Are some clusters overlapping?
- Are some items far away from their assigned cluster?
- Would increasing or decreasing `k` make the structure clearer?

The axes are called Dimension 1 and Dimension 2.

Do not interpret them like regular survey variables.

The important thing is the distance between points.

Cards that are closer together are more similar in the card sorting data.

---

# Part 3: What Files Did the Script Create?

After running the full script, open the `outputs` folder.

You should see files such as:

```text
01_distance_matrix_heatmap.png
02_hca_method_comparison.csv
dendrogram_complete.png
dendrogram_average.png
dendrogram_single.png
dendrogram_ward.D.png
dendrogram_ward.D2.png
03_selected_dendrogram_METHOD_kNUMBER.png
04_cluster_assignments_METHOD_kNUMBER.csv
04_cluster_summary_METHOD_kNUMBER.csv
05_mds_plot_METHOD_kNUMBER.png
```

Each output has a different purpose.

| Output | What it helps you do |
|---|---|
| Distance matrix heatmap | See early signs of item groupings |
| Method comparison table | Compare dendrogram methods |
| Individual dendrograms | Inspect how each method groups the cards |
| Selected dendrogram | View the chosen cluster solution |
| Cluster assignment table | See which item belongs to which cluster |
| Cluster summary table | Review each cluster as a list of items |
| MDS plot | Check whether the clusters make sense spatially |

---

# Part 4: How to Use These Outputs

Use the outputs in this order.

## First, inspect the distance matrix

Look for blocks or groups.

This gives you a first impression of the structure.

## Second, compare the methods

Open the method comparison table.

Start with the method that has the highest cophenetic correlation.

Then inspect the dendrograms visually.

## Third, choose a starting value of k

Start with a reasonable number of clusters.

If you have many cards, very small values of `k` may create groups that are too broad.

If you use too many clusters, the structure may become too fragmented.

## Fourth, inspect the selected dendrogram

Look at the items inside each cluster box.

Ask whether the groupings make sense.

## Fifth, inspect the MDS plot

Check whether the items in each cluster appear close together.

If the MDS plot suggests that one cluster is too spread out, try increasing `k`.

If the MDS plot suggests that many clusters overlap or are too small, try decreasing `k`.

## Sixth, review the cluster summary table

This is where the analysis becomes practical.

The cluster summary table helps you start naming the categories.

---

# Part 5: Common Beginner Mistakes

## Mistake 1: Thinking the highest coefficient gives the final answer

The coefficient helps compare methods, but it does not make the UX decision for you.

A clustering method can have a strong coefficient and still produce categories that are not useful for navigation.

## Mistake 2: Choosing k too quickly

Do not choose `k` only because the plot looks colorful.

Try a few values.

For example:

```r
k <- 5
k <- 6
k <- 7
k <- 8
k <- 9
```

Each time you change `k`, rerun the selected dendrogram, cluster table, and MDS sections.

## Mistake 3: Ignoring large clusters

A large cluster may be meaningful, but it may also be too broad for a menu.

Large clusters often need subcategories.

## Mistake 4: Ignoring strange items

If an item appears in a cluster where it does not seem to belong, do not ignore it.

That item may be ambiguous.

Ambiguous items are useful because they show where users may disagree.

## Mistake 5: Treating the dendrogram as the final website structure

The dendrogram is a guide.

The final information architecture should also consider:

- User expectations
- Stakeholder goals
- Existing navigation
- Business requirements
- Content strategy
- Usability testing or tree testing results

---

# Part 6: Complete R Script

You can copy and paste the full script below.

```r
# ============================================================
# MeasureUX Card Sorting Analysis
# HCA, Dendrograms, and MDS
# ============================================================

packages <- c("ggplot2", "ggrepel", "factoextra")

for (pkg in packages) {
  if (!require(pkg, character.only = TRUE)) {
    install.packages(pkg)
    library(pkg, character.only = TRUE)
  }
}

dir.create("outputs", showWarnings = FALSE)

# -----------------------------
# User settings
# -----------------------------

data_file <- "card_sort_data.csv"

# Start with a reasonable number, then adjust after inspecting outputs
k <- 6

# -----------------------------
# Load the data
# -----------------------------

raw_data <- read.csv(data_file, check.names = FALSE)

if (!"Name" %in% names(raw_data)) {
  stop("Your file must contain a column called 'Name'. This column should contain the card or item names.")
}

cat("Number of cards/items:", nrow(raw_data), "\n")
cat("Number of columns in the file:", ncol(raw_data), "\n")

# -----------------------------
# Prepare the item-category matrix
# -----------------------------

item_names <- raw_data$Name

category_data <- raw_data[, names(raw_data) != "Name"]

converted_data <- lapply(category_data, function(x) {
  suppressWarnings(as.numeric(x))
})

problem_columns <- names(category_data)[
  sapply(seq_along(category_data), function(i) {
    original_values <- trimws(as.character(category_data[[i]]))
    converted_values <- converted_data[[i]]
    any(is.na(converted_values) & original_values != "" & !is.na(original_values))
  })
]

if (length(problem_columns) > 0) {
  stop(
    paste(
      "These category columns contain non-numeric values:",
      paste(problem_columns, collapse = ", "),
      "\nPlease replace text values with numbers before running the analysis."
    )
  )
}

item_category_matrix <- as.data.frame(converted_data)

item_category_matrix[is.na(item_category_matrix)] <- 0

rownames(item_category_matrix) <- make.unique(item_names)

empty_rows <- rowSums(item_category_matrix) == 0

if (any(empty_rows)) {
  warning("Some cards had no category information and were removed.")
  item_category_matrix <- item_category_matrix[!empty_rows, ]
}

cat("Number of cards/items used in the analysis:", nrow(item_category_matrix), "\n")
cat("Number of category columns:", ncol(item_category_matrix), "\n")

# -----------------------------
# Remove category columns with no variation
# -----------------------------

zero_variation_columns <- sapply(item_category_matrix, function(x) {
  sd(x, na.rm = TRUE) == 0
})

if (any(zero_variation_columns)) {
  cat("Removing category columns with no variation:\n")
  print(names(item_category_matrix)[zero_variation_columns])
}

analysis_matrix <- item_category_matrix[, !zero_variation_columns, drop = FALSE]

cat("Number of category columns used after cleaning:", ncol(analysis_matrix), "\n")

# -----------------------------
# Standardize the category data
# -----------------------------

scaled_matrix <- scale(analysis_matrix)

# -----------------------------
# Create the distance matrix
# -----------------------------

card_distance <- dist(scaled_matrix, method = "euclidean")

distance_plot <- fviz_dist(
  card_distance,
  lab_size = 5
) +
  ggtitle("Distance Matrix for Card Sorting Data")

ggsave(
  filename = "outputs/01_distance_matrix_heatmap.png",
  plot = distance_plot,
  width = 12,
  height = 10
)

# -----------------------------
# Run HCA with several methods
# -----------------------------

methods <- c("complete", "average", "single", "ward.D", "ward.D2")

hca_solutions <- lapply(methods, function(method_name) {
  hclust(card_distance, method = method_name)
})

names(hca_solutions) <- methods

# -----------------------------
# Compare the clustering methods
# -----------------------------

method_comparison <- data.frame(
  Method = methods,
  Cophenetic_Correlation = sapply(hca_solutions, function(hc) {
    cor(card_distance, cophenetic(hc))
  })
)

method_comparison$Cophenetic_Correlation <- round(
  method_comparison$Cophenetic_Correlation,
  3
)

method_comparison <- method_comparison[
  order(-method_comparison$Cophenetic_Correlation),
]

print(method_comparison)

write.csv(
  method_comparison,
  "outputs/02_hca_method_comparison.csv",
  row.names = FALSE
)

# -----------------------------
# Save one dendrogram for each method
# -----------------------------

for (method_name in methods) {
  png(
    filename = paste0("outputs/dendrogram_", method_name, ".png"),
    width = 1400,
    height = 900
  )

  plot(
    hca_solutions[[method_name]],
    cex = 0.55,
    hang = -1,
    main = paste("Dendrogram -", method_name)
  )

  dev.off()
}

# -----------------------------
# Select the HCA method
# -----------------------------

selected_method <- method_comparison$Method[1]

# You can also manually choose a method by uncommenting one line below:
# selected_method <- "complete"
# selected_method <- "average"
# selected_method <- "ward.D"
# selected_method <- "ward.D2"

selected_hca <- hca_solutions[[selected_method]]

cat("Selected method:", selected_method, "\n")

# -----------------------------
# Save selected dendrogram with cluster boxes
# -----------------------------

png(
  filename = paste0("outputs/03_selected_dendrogram_", selected_method, "_k", k, ".png"),
  width = 1600,
  height = 1000
)

plot(
  selected_hca,
  cex = 0.55,
  hang = -1,
  main = paste("Selected Dendrogram:", selected_method, "with k =", k)
)

rect.hclust(
  selected_hca,
  k = k,
  border = 2:(k + 1)
)

dev.off()

# -----------------------------
# Create cluster assignment table
# -----------------------------

cluster_assignments <- cutree(selected_hca, k = k)

cluster_table <- data.frame(
  Item = names(cluster_assignments),
  Cluster = cluster_assignments
)

cluster_table <- cluster_table[
  order(cluster_table$Cluster, cluster_table$Item),
]

print(cluster_table)

write.csv(
  cluster_table,
  paste0("outputs/04_cluster_assignments_", selected_method, "_k", k, ".csv"),
  row.names = FALSE
)

cluster_summary <- aggregate(
  Item ~ Cluster,
  data = cluster_table,
  FUN = function(x) paste(x, collapse = ", ")
)

write.csv(
  cluster_summary,
  paste0("outputs/04_cluster_summary_", selected_method, "_k", k, ".csv"),
  row.names = FALSE
)

# -----------------------------
# Create MDS plot
# -----------------------------

mds_result <- cmdscale(card_distance, eig = TRUE, k = 2)

mds_points <- as.data.frame(mds_result$points)
colnames(mds_points) <- c("Dimension_1", "Dimension_2")

mds_data <- data.frame(
  Item = rownames(mds_points),
  Dimension_1 = mds_points$Dimension_1,
  Dimension_2 = mds_points$Dimension_2,
  Cluster = factor(cluster_assignments[rownames(mds_points)])
)

mds_plot <- ggplot(
  mds_data,
  aes(
    x = Dimension_1,
    y = Dimension_2,
    color = Cluster,
    label = Item
  )
) +
  geom_point(size = 3) +
  ggrepel::geom_text_repel(
    size = 3,
    max.overlaps = 100
  ) +
  labs(
    title = paste("MDS Plot with", k, "Clusters"),
    subtitle = paste("HCA method:", selected_method),
    x = "Dimension 1",
    y = "Dimension 2"
  ) +
  theme_minimal()

print(mds_plot)

ggsave(
  filename = paste0("outputs/05_mds_plot_", selected_method, "_k", k, ".png"),
  plot = mds_plot,
  width = 12,
  height = 9
)
```

---

# Key Takeaways

HCA helps you find possible groups in card sorting data.

Dendrograms show those groups as a tree.

Different dendrogram methods may produce different results because they use different rules for combining cards.

The method comparison table helps you choose a reasonable starting method.

The number of clusters, `k`, is a design decision informed by the data.

MDS helps you check whether the clusters make sense as a spatial map.

The final categories must still be interpreted and named by the UX researcher.

The next page should focus on interpreting the outputs and translating them into information architecture recommendations.

<!-- MeasureUX copy buttons for R code blocks -->
<style>
.measureux-code-wrapper {
  position: relative;
}

.measureux-copy-button {
  position: absolute;
  top: 0.45rem;
  right: 0.45rem;
  z-index: 10;
  padding: 0.25rem 0.55rem;
  border: 1px solid #cccccc;
  border-radius: 4px;
  background: #f7f7f7;
  color: #333333;
  font-size: 0.75rem;
  cursor: pointer;
}

.measureux-copy-button:hover {
  background: #eeeeee;
}

.measureux-code-wrapper pre {
  padding-top: 2.2rem;
}
</style>

<script>
document.addEventListener("DOMContentLoaded", function () {
  const rCodeBlocks = document.querySelectorAll("pre > code.language-r");

  rCodeBlocks.forEach(function (codeBlock) {
    const pre = codeBlock.parentElement;

    if (pre.parentElement.classList.contains("measureux-code-wrapper")) {
      return;
    }

    const wrapper = document.createElement("div");
    wrapper.className = "measureux-code-wrapper";

    pre.parentNode.insertBefore(wrapper, pre);
    wrapper.appendChild(pre);

    const button = document.createElement("button");
    button.className = "measureux-copy-button";
    button.type = "button";
    button.innerText = "Copy";

    button.addEventListener("click", function () {
      const code = codeBlock.innerText;

      navigator.clipboard.writeText(code).then(function () {
        button.innerText = "Copied!";
        setTimeout(function () {
          button.innerText = "Copy";
        }, 1600);
      }).catch(function () {
        button.innerText = "Select code";
        setTimeout(function () {
          button.innerText = "Copy";
        }, 1600);
      });
    });

    wrapper.insertBefore(button, pre);
  });
});
</script>
