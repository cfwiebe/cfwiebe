---
title: "Projects"
layout: default
---

# Selected Projects

---

## Wolverine Habitat Suitability Modeling

**Tools Used:** R · QGIS · SQL · ggplot2 · sf · Raster Analysis  
**Goal:** Predict and visualize optimal wolverine habitat in Oregon based on climate and landcover.

### Observational Data and Mapping

```r
wolverine_obs_raw <- read.csv("https://raw.githubusercontent.com/cfwiebe/datasets/refs/heads/main/wolverine_obs_raw.csv")
obs_qqll <- wolverine_obs_raw %>%
  group_by(obs) %>%
  summarise(count = n())
```

```r
ggplot(qqll_with_obs_plus1) +
  geom_sf(aes(fill = obs_plus1)) +
  scale_fill_continuous_sequential("Greens 3", trans = "log") +
  theme_minimal()
```

### Binary Observational Data and Climate Features

```r
montana_final$obs_binary <- ifelse(montana_final$obs > 0, 1, 0)
```

### Landcover Modeling

```r
ice_mod <- lm(obs_binary ~ perennial_icesnow + temp_dif + prec_dif, data = montana_final)
summary(ice_mod)
```

### Final Output: Ideal Wolverine Habitat in Oregon

![Wolverine Habitat Map](assets/ideal_wolverine_habitat_FINAL.png)

> Based on regression models and raster overlays, I estimated that ~12.3% of Oregon offers ideal habitat for wolverines — compared to 27.4% in Montana.

---

## Predictive Modeling of Heart Failure

**Tools Used:** R · KNN · Logistic Regression · Decision Trees · caret · rpart  
**Goal:** Predict heart disease from patient vitals using supervised learning techniques.

### Load & Normalize Data

```r
heart <- read.csv("https://raw.githubusercontent.com/cfwiebe/datasets/refs/heads/main/heart.csv")
normalize <- function(x) (x - min(x)) / (max(x) - min(x))
heart$MaxHRNorm <- normalize(heart$MaxHR)
```

### Stratified Train-Test Split

```r
heart0 <- filter(heart, HeartDisease == 0)
heart1 <- filter(heart, HeartDisease == 1)
heart_sample0 <- sample(1:nrow(heart0), 287)
heart_sample1 <- sample(1:nrow(heart1), 356)

trainStrat <- rbind(heart0[heart_sample0, ], heart1[heart_sample1, ])
testStrat  <- rbind(heart0[-heart_sample0, ], heart1[-heart_sample1, ])
```

### K-Nearest Neighbors

```r
library(class)
trainFea <- select(trainStrat, -HeartDisease)
testFea  <- select(testStrat, -HeartDisease)
trainOut <- trainStrat$HeartDisease
testOut  <- testStrat$HeartDisease

knn.heartPredFinal <- knn(train = trainFea, test = testFea, cl = trainOut, k = 11)
mean(knn.heartPredFinal == testOut)  # ~73.8% accuracy
```

### Logistic Regression

```r
modLR <- glm(HeartDisease ~ Cholesterol, data = trainStrat, family = "binomial")
summary(modLR)
```

### Decision Trees & Bagging

```r
library(caret)
caretBag <- train(HeartDisease ~ ., data = trainStrat, method = "treebag",
                  trControl = trainControl("cv", number = 10))
vip(caretBag)
```

> The bagged tree model achieved ~75% accuracy, with clear variable importance outputs.

---

**Next Steps:**  
- Deploy predictive tools as R Shiny apps  
- Integrate SHAP explanations for transparency in clinical settings
