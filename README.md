# Active-Learning-for-gas-mixture-adsorption-prediciton #
## Files and Folders ##
`P-X` folder: Active learning code for adsorption prediction in P-X phase space for 3 gas mixtures.

The prior_sample.csv and complete.csv are the prior and ground truth data for the systems.
GP_mixtures.py is the Active learning engine (with dual-GP model)
Active-learning-sh is the model updation handler which works in linux environment. This was based on Notre Dame Center for computing resource which is a grid-engine system. One can modify the first few lines depending on the specific linux environment or to run it locally, the first line can be removed as well.

`P-X-T` folder: Active learning code for adsorption prediction in P-X-T phase space for 3 gas mixtures.
Same process as P-X phase space.
## Algorithm ##
A general explanation of the workflow is outline below:

![Krish_AL_mixtures_workflow](https://user-images.githubusercontent.com/36941306/223895808-1b404ce3-b044-44fe-be13-f31a2f252ccc.png)

Active learning workflow for predicting adsorption using gaussian process regression (GPR). 
The learning starts from pre-processing the prior data. Pressure and temperature are standardised, while the mole-fraction is linearly scaled to –1 and 1, i.e. X* = (X - 1/2)x25/12. Then it is passed through the dual-GPs, one for each species. Then prediction are done, and the associated uncertainties are extracted. The accuracies for both the species are tested for convergence. If any of the accuracy criteria is not met, learning continues, and the point with the highest uncertainty is added to the prior data. The active learning continues until the convergence condition is satisfied.
The Accuracy parameter is defined as follows:
```math
     ACC_{i} = 100 \times \frac{X_{+}}{X_{+}+X_{-}}
```
```math
\begin{aligned}
     \text{If at $X_{n_{i}}$,} \;
     |\frac{\sigma_{n_{i}}}{y^{'}_{n_{i}}}| <= \beta_{i}, \;
     \text{then,} \; X_{+} = X_{+} + 1 \;
     \text{else,} \;
      X_{-} = X_{-} + 1
      \end{aligned}
 ```
Also, the $\beta$ has a value of 2% for P-X and 5% for P-X-T calculation. $\sigma_{n_{i}}$ and  $y_{n_{i}}$ are the uncertainty and adsorption value (scaled) associated with the test point $X_{n_{i}}$. The threshold value $\beta_{i}$ is user-defined and can be set on the basis of the desired confidence the user needs. Also, we have kept the $\beta$ values same for the all the species in the three gas mixtures. We had $\beta$ set to 2\% for the P–X phase space, while we had chosen a high upper limit of 5\% for active learning in the P–X–T space.

## Key Performance parameters ##

Mean Relative Error (MRE):

```math
    \text{MRE in \%} = \left(\sum_{i=1}^{n} \Bigg|{\frac{Y_{\text{GP-predict}}(x_i) - Y_{\text{GCMC}}(x_i)}{Y_{\text{GCMC}}(x_i)+\boldsymbol{\epsilon}}}\Bigg|\right) \times \frac{100}{n}
```
Data requirement (% of ground truth):
```math
    \text{Data Requirement in \%} = \frac{{N_{Prior}} + {N_{Iterations-to-90\% Accuracy}}}{{N_{Ground-truth}}}\times100
```
R<sup>2</sup>:
```math
R^2(Y_GCMC, Y_GP-predict) = 1 - \frac{\sum_{i=1}^{n} (Y_{\text{GCMC}}(x_i) - Y_{\text{GP-predict}}(x_i))^2}{\sum_{i=1}^{n} (Y_{\text{GCMC}}(x_i) - \bar{Y_{\text{GCMC}}(x_i)})^2}
```

## Results ##

The performance of the Active learning workflow is below for both the phase spaces and three systems:

Results for the P-X phase space:

| Mixture | Kernel | Data requirement (% of ground truth) | MRE<sub>(species 1)</sub> (%) | MRE<sub>(species 2)</sub> (%) | R<sup>2</sup><sub>(species 1)</sub> | R<sup>2</sup><sub>(species 2)</sub> |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| CO<sub>2</sub>-CH<sub>4</sub> | RBF | 3.001 | 5.263 | 5.417 | 0.986 | 0.999 |
| Xe-Kr | RQ | 2.601 | 6.526 | 6.394 | 0.985 | 0.998|
| H<sub>2</sub>S-CO<sub>2</sub> | RQ | 2.561 | 7.149 | 7.154 | 0.982 | 0.995 |


Results for the P-X-T phase space:

| Mixture | Kernel | Data requirement (% of ground truth) | MRE<sub>(species 1)</sub> (%) | MRE<sub>(species 2)</sub> (%) | R<sup>2</sup><sub>(species 1)</sub> | R<sup>2</sup><sub>(species 2)</sub> |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| CO<sub>2</sub>-CH<sub>4</sub> | RBF+RBF+RBF | 6.611 | 5.461 | 9.256 | 0.988 | 0.990 |
| Xe-Kr | RBF+RBF+RBF | 6.650 | 4.850 | 7.025 | 0.990 | 0.990|
| H<sub>2</sub>S-CO<sub>2</sub> | RQ | 5.549 | 8.276 | 11.682 | 0.976 | 0.986 |

