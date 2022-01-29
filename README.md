# Distillation For Machine Learning Models

This package provides several methods for model distillation and interpretability 
for general black box machine learning models. This page serves as a clear 
example for using this package, where we use the MASS crabs data set.

## General Prediction Wrapper

First we load in the crabs data set. This contains physical measurements of 
several species of crabs collected at Fremantle, West Australia.
```
library(MASS)
library(data.table)
library(ggplot2)
library(interpret)
library(Rforestry)

set.seed(491)

data <- MASS::crabs
levels(data$sex) <- list(Male = "M", Female = "F")
levels(data$sp) <- list(Orange = "O", Blue = "B")
colnames(data) <- c("Species","Sex","Index","Frontal Lobe",
                    "Rear Width", "Carapace Length","Carapace Width","Body Depth")

```

We can train a random forest to estimate the Carapace Width of the crabs based on the
other features. In order to use the interpretability features, we must create 
a `Predictor` class for the estimator we want to interpret. This class 
standardizes the predictions, tracks the outcome feature, and stores the 
training data.

```
# Get training data set
set.seed(491)
test_ind <- sample(1:nrow(data), nrow(data)%/%5)
train_reg <- data[-test_ind,]
test_reg <- data[test_ind,]

# Train a random forest on the data set
forest <- forestry(x=train_reg[,-which(names(train_reg)=="Carapace Width")],
                   y=train_reg[,which(names(train_reg)=="Carapace Width")])

# Create a predictor wrapper for the forest
# this allows us to use a standard wrapper for querying any 
# trained estimator
forest_predictor <- Predictor$new(model = forest, 
                                  data=train_reg, 
                                  y="Carapace Width",
                                  task = "regression")
```

## Interpretability Wrapper

Once we have initialized a `Predictor` object for the forest, we can pass this to the 
`Interpretor` class. In the future, this will have several methods implemented 
as different options, but now it defaults to creating PDP functions + plots for 
the estimator. Examining the `Interpretor`, we can see the current method 
selected: "pdp", the feature names, the training data indices, and the lists of 
pdp functions.

```
forest_interpret <- Interpreter$new(predictor = forest_predictor)

print(forest_interpret)
```

The pdp functions are stored in two lists, one for 1-d pdp functions and one for 2-d pdp functions.
For any feature, we can retrieve the pdp function by selecting the entry in the list with
that feature name

```
one_feat <- train_reg$`Frontal Lobe`
preds_pdp <- forest_interpret$functions.1d$`Frontal Lobe`(one_feat)
print(preds_pdp)

```

## Plotter Wrapper

In order to use these pdp functions to create plots, we can use the `Plotter` function.
This allows us to make standard pdp plots as well as conditional pdp plots with two variables.
Here we show a single variable pdp plot and three conditional pdp plots: two with with second categorical 
variables, and one with two continuous variables.

```
# check plotter wrapper
forest_plot <- Plotter$new(forest_interpret, features = c("Frontal Lobe"),
                          features.2d = data.frame(col1 = c("Frontal Lobe", "Frontal Lobe", "Frontal Lobe"),
                                                   col2 = c("Species", "Sex", "Rear Width")))

plots <- plot(forest_plot)
plots[[1]]
plots[[2]]
plots[[3]]
plots[[4]]
```

We can also adjust where the plots are centered at for each feature, and what
grid of points are used in the evaluations that create the plots.

```
# Set center and grid points for the plots
forest_plot$center.at$`Frontal Lobe`
forest_plot$grid.points$`Frontal Lobe`

# We can reset where we center the plot
set.center.at(forest_plot, "Frontal Lobe", 2) # for numeric
forest_plot$center.at$`Frontal Lobe`
```



## TODO
It would be nice to have in the future:
- ALE Plots
- A PDP surrogate model implemented
- Some sort of loneliness index implemented
- When the covariates of the observation are in the support of the training set, 
  create bootstrap confidence bands for the PDP predictions.

