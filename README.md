# Crop-Stress

## Model Description

The model used and created is a spatial linear model where the model accounts for spatial correlation between between observations. The model follows the formula $y \sim MVN(X\beta, \sigma^2R)$ where y is the crop water stress index (CWSI), X is the predictor matrix, $\beta$ is the effects of the predictors, $\sigma^2$ is the total variance, and R is a dense correlation matrix that accounts for the spatial correlation between the observations. The spatial correlation values are calculated using an exponential correlation function that takes into account the spatial distance between observations, and optimizes parameters $\phi$ and $\omega$ to define the correlation between observations. $\omega$ defines the maximum allowed correlation between two points at the same location. $\phi$ defines the range of that correlation. The predictor variables passed into the model are the slope, the topographic wetness index (twi), the aspect, the electrical conductivity of the soil (eca_shallow), the normalized difference vegetation index (ndvi), the provided spatial x and y coordinates, and twenty spatial engineered variables. The spatial engineered variables are engineered using a function that spreads twenty points around the observed area and then calculates the distances of the observations from those points, then applies a weight to those distances (so super far away points will have zero weight). After training this model and running a twenty fold cross validation the model actually scores pretty well with a 0.3524005 RMSE across the folds.

## Dependencies

Below is a list of required R packages in order to use and reproduce the same model we made. Includes all packages used in cleaning and preparing the data for training as well:

 - tidyverse
 - FRK
 - spmodel

## Objects

- obj  
  - Class: list  
  - Description: Main object loaded from `cwsi_model.rds`. Contains the trained model, spatial feature parameters, and data required for prediction.

- model  
  - Class: spmodel object  
  - Description: Trained spatial linear model used to generate predictions of CWSI.

- spatial_info  
  - Class: list  
  - Description: Contains parameters used to construct spatial features, including `centers`, `scale`, and `K`.

- data  
  - Class: data.frame  
  - Description: Dataset used for generating predictions. Serves as an example input structure for new data.


## Usage

Below is example code of how to load in the model, generate predictions, and plot those predictions (same code included in Crop Growth Example.qmd):

```r
#Downloading Libraries
library(tidyverse)
library(FRK)
library(spmodel)

# Load model
obj <- readRDS("cwsi_model.rds")
model <- obj$model
spatial_info <- obj$spatial_info

# Load new data
new_data <- obj$data

# Extract spatial info
centers <- spatial_info$centers
scale <- spatial_info$scale
K <- spatial_info$K

# Create spatial features
coords <- as.matrix(new_data[, c("point_x", "point_y")])

spatial_features <- local_basis(
  manifold = plane(),
  loc = centers,
  scale = rep(scale, K),
  type = "bisquare"
) |>
  eval_basis(coords) |>
  as.matrix()

colnames(spatial_features) <- paste0("SF", 1:K)

# Combine with data
prediction_df <- bind_cols(new_data, spatial_features)

# Predict
prediction_df$yhat <- predict(model, newdata = prediction_df)

# Plot
ggplot(prediction_df, aes(point_x, point_y, color = yhat)) +
  geom_point() +
  scale_color_viridis_c() +
  labs(title = "Predicted CWSI")
```

## Output

The code above creates two meaningful output: a prediction data frame, and a plot of those predictions. The prediction data frame is created in the process of making the predictions on new locations. It includes all of the predictor variables used and engineered to make predictions on the data, and a "yhat" column that includes the predicted CWSI. It is important to note that the "cwsi" column may be empty or unavailable in prediction settings because it is an area we are trying to predict on and don't have data on, and that the "yhat" column is the predicted CWSI. Furthermore all spatial features that were engineered in the process of training and creating the model follow the format of "SF{number}".

The plot that is output is just a plot of the predicted values made by the model. It maps out all the points according to the given x and y coordinate values, and then the color represents the predicted CWSI value. Bright and yellow representing a higher predicted CWSI value and darker and bluer color representing a lower CWSI value.