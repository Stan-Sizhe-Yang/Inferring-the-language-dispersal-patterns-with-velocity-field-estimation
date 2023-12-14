# Inferring-language-dispersal-patterns-with-velocity-field-estimation
**R package and tutorial of the language velocity field estimation (LVF) for reconstructing langauge dispersal pattern.**
## I. File description
**This repository contains three files:**

**1.  Supplementary_Materials: Datasets**
  
&emsp;&emsp;The datasets of four language cases utilized in this study. The detailed information can be found in README within this file.  

**2.  Supplementary_Materials：R_codes_for_validations_and_comparisons_of_LVF**
  
&emsp;&emsp;The R codes for model validations and comparisons conducted in this study. The detailed information can be found in README within this file.
    
&emsp;&emsp;Note that due to the memory limitation, the file "Supplementary_Materials：R_codes_for_validations_and_comparisons_of_LVF" is zipped into two .rar files: "R_codes_for_validations_and_comparisons_of_LVF.part1" and "R_codes_for_validations_and_comparisons_of_LVF.part2". These two .rar files need to be unzipped together to extract the complete files of Supplementary_Materials：R_codes_for_validations_and_comparisons_of_LVF.  

**3.  Supplementary_Materials：R_package_and_tutorial_of_LVF**
  
&emsp;&emsp;The R package namely LVF and the tutorial for its implementaion. The detailed information can be found in README within this file.  

## II. Detailed tutorial for applying LVF

&emsp;&emsp;**Here, we will detailly explain each function and its parameters of "LVF" package based on the schematic overview (Figure 1) of the rationale and procedure of LVF.**

![433661_2_figure_8379516_s5by4z_convrt](https://github.com/Stan-Sizhe-Yang/Inferring-language-dispersal-patterns-with-velocity-field-estimation/assets/46415427/1191995a-bfaa-41a2-bc2a-41082104aace)
**Figure 1: Schematic overview of the LVF for inferring the dispersal trajectories and centres of languages.**

### 1. Applying LVF with one function "LVF.run.fun"
&emsp;&emsp;The LVF can be conducted using only one function: LVF.run.fun
```
LVF.res <- LVF.run.fun(data,
  
                       # The parameters below are for plotting the language velocity field.
                         
                       # They should better be chosen depending on the visual scale and geographic range of the figure.
                         
                       grid.smooth.map.sep2 = 2, grid.smooth.sigma = 1, grid.smooth.map.n = 10,
                         
                       PCA.velocity.length = 0.8, map.region = 'world',
                         
                       spatial.map.length = 0.1, spatial.map.cex = 0.3, spatial.map.arrow.cex = 0.03,
                         
                       grid.map.length = 1, grid.map.cex = 1, grid.map.arrow.cex = 0.05, grid.map.n.grid = 4)
```
&emsp;&emsp;The input dataset for parameter "data" should be a data.frame containing these columns:
  
&emsp;&emsp;![image](https://github.com/Stan-Sizhe-Yang/Inferring-language-dispersal-patterns-with-velocity-field-estimation/assets/46415427/281429ec-14b6-460f-862c-f55c0967f2f8)
  
&emsp;&emsp;The first column contains the labels of the language samples and the second column contains the names of the language samples. These two columns should be at "character" format. The third and fourth columns contains the longitudes and latitudes of the language samples. They should be at the "numeric" format. The rest of the columns contains the values of the binary-coded lingusitic traits and they should be at "numeric" format.

### 2. Decomposing function "LVF.run.fun"
&emsp;&emsp;**The "LVF.run.fun" contains belows functions**
#### 1. Prepare input variables
* Filter out the language samples without geographic coordinates
  ```
  data.omit <- data[!is.na(data[, 4]), ]
  ```
* Filter out the linguistic traits with missing values >75%
  ```
  data.02 <- data.01[, keep.missing(data.01)]
  ```
* Mode imputation for missing values of each linguistic trait
  ```
  data.03 <- imputation(data.02)
  ```
* Prepare the rest of the input variables
  ```
  label <- as.factor(data.omit[, 1])
  coord <- data.omit[, c("Longitude", "Latitude")]
  coord.mat <- data.frame(name = label, longtitude = coord[, 1], latitude = coord[, 2])
  ```
#### 2. Convert the binary values of each linguistic trait into state frequencies
&emsp;&emsp;The conversion of the binary value for each linguistic trait in each language sample is accomplished with "knn.smooth.func" function by choosing _nearest.n_ geographically closest language samples to that language sample (Figure 1c).
  
```
smooth.data <- knn.smooth.func(data.03, nearest.n = 10, extral.info = coord)
```
  
&emsp;&emsp;The default value for parameter "nearest.n" is 10. The smooth.data should be with this format.

   ![image](https://github.com/Stan-Sizhe-Yang/Inferring-language-dispersal-patterns-with-velocity-field-estimation/assets/46415427/6b36fb4e-bcd0-418d-a021-fb22ad7109d5)

### 3. Establish the velocity field within high-dimensional space
* Estimate the prestige parameter in our dynamic model using Poisson process. The default value for the parameter _lambda_ which denotes the mutation rate of Poisson process is 1 (Figure 1d1).
  ```
  prestige <- possion.prcess.func(data.03, lambda = 1)
  ```
* Reconstruct the past state for each linguistic trait and calcualte the high-dimensional velocity field (Figure 1d2).
  ```
  pre <- pre.smooth.data.func(smooth.data, prestige)
  ```
  The output "pre" is a list that contains two elements. The first one is the matrix containing the past states for linguistic traits in language samples. The second is the matrix containing the high-dimensional velocity vectors for language samples.

### 4. PCA projection of high-dimensional velocity field
&emsp;&emsp;Perform the Principle Component Analysis (PCA) to extract the lingusitic relatedness among language samples within two-dimensional PC space and project the high-dimensional velocity field into this two-dimensional PC space (Figure 1e).
```
pca.vel <- pca.velocity(data.03, pre, label)
color <- velocity.plt(pca.vel$data.comb, pca.vel$v.dirc, l = 1.2, ratio = 1)
```
&emsp;&emsp;The output "pca.vel" contains two elements. The first one is the matrix containing the PC values of the binary-coded lingusitic traits. The second one is the matrix containing the velocity vectors of language samples within PC space. The "velocity.plt" function will return a figure that is the visualization of the velocity field within PC space as shown in below. The parameter _l_ is the length of the arrow visualized in this figure and the parameter _ratio_ is the aspect ratio of this figure.
  
![image](https://github.com/Stan-Sizhe-Yang/Inferring-language-dispersal-patterns-with-velocity-field-estimation/assets/46415427/9eb61030-93bb-4ee6-b2ea-c80afab26d06)
