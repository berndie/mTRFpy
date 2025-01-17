Statistical inference
=====================

In the previous section, we used nested cross-validation to compute an unbiased estimate of a forward TRF. The correlation between the predicted and actual brain response was 0.024 - is this a high or low value and has it any meaning at all? To answer this question, we need a statistical test.

Permutation
-----------
One way to determine the significance of an estimate without making assumptions about the underlying distribution is permutation testing [#f1]_. There are different ways to permute the data but the overarching idea is the same - the causal relationship between variables of interest (in our case stimulus and neural response) is removed by random permutation. Then, the statistical model is applied to the permuted data to compute the estimate of interest (e.g. correlation coefficient). This process is repeated many (thousand) times, each time with a different random permutation. Thus, we obtain the permutation distribution which reflects the expected observations if there is no actual relationship between the variables of interest. The p-value of the actual observation is given by the probability of obtaining a value as large or larger under the permutation distribution. A low p means that it is unlikely to observe a given value if there were no relationship between the variables.

Significance
------------

In the below example, we are using the :py:func:`permutation_distribution` function from the :py:mod:`stats` module which randomly permutes the data and, for each permutation, estimates the accuracy of the TRF model using cross-validation. The number of permutations is set by the :py:const:`n_permute` parameter (in this demo we use 100 but usually you would use more in an actual analysis) and the number of cross-validation folds :py:const:`k` should be the same that was used to estimate model accuracy for the actual data. Finally, we then compute the p-value of the observed correlation as the number of elements in the permuted distribution that are equal or higher divided by the number of permutations. ::
    
    import numpy as np
    from matplotlib import pyplot as plt
    from mtrf.model import TRF, load_sample_data
    from mtrf.stats import permutation_distribution
    r_obs = 0.024 # the previously observed correlation
    regularization = 6000 # the lambda value that worked best
    trf = TRF()  # use forward model
    stimulus, response, fs = load_sample_data(n_segments=5)
    tmin, tmax = 0, 0.4  # range of time lags
    r_perm, mse_perm = permutation_distribution(
        trf, stimulus, response, fs, tmin, tmax, regularization, n_permute=100
        )
    p = sum(r_perm>=r_obs)/len(r_perm)
    plt.hist(r_perm, bins=10)
    plt.axvline(x=r_obs, ymin=0, ymax=1, color='black', linestyle='--')
    plt.xlabel('Correlation [r]')
    plt.ylabel('Number of models')
    plt.annotate(f'p={p.round(2)}', (0.04, 14))
    plt.show()

.. image:: images/perm.png
    :align: center
    :scale: 30 %

The p-value of the observed correlation is 0.15, which means that we obtain a correlation of that or larger size for 15 percent of all models which were trained on randomly permuted data. Thus, we would not reject our null hypothesis at the typically used significance level :math:`\alpha=0.05`. This is hardly surprising given that EEG has a poor signal-to-noise ratio and we are only using two minutes of data.

.. [#f1] Ernst, M. D. (2004). Permutation methods: a basis for exact inference. Statistical Science, 676-685.
