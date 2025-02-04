.. role:: raw-html(raw)
    :format: html

GPBoost: Summary of Most Important Parameters
=============================================

This page contains a summary of the most important parameters. We distinguish between (i) tuning and other parameters for the tree-boosting
part and (ii) modeling specifications and optimization parameters for the Gaussian process and random effects part. Currently, the GPBoost library
supports the following likelihoods / objective functions for combining tree boosting with Gaussian process and random effects models:
"gaussian", "bernoulli_probit" (="binary"), "bernoulli_logit", "poisson", "gamma". This distribution of the data can be specified through the
``objective`` paramter for the tree part or the ``likelihood`` parameter for the random effects model part.

.. contents:: **Contents**
    :depth: 2
    :local:
    :backlinks: none

Tree-boosting parameters
~~~~~~~~~~~~~~~~~~~~~~~~

Tree-boosting tuning parameters
-------------------------------
Below is a list of important tuning parameters for tree-boosting.

-  ``num_iterations`` :raw-html:`<a id="num_iterations" title="Permalink to this parameter" href="#num_iterations">&#x1F517;&#xFE0E;</a>`, default = ``100``, type = int, aliases: ``num_iteration``, ``n_iter``, ``num_tree``, ``num_trees``, ``num_round``, ``num_rounds``, ``num_boost_round``, ``n_estimators``, constraints: ``num_iterations >= 0``

   -  number of boosting iterations

   -  this is arguably the most important tuning parameter, in particular for regession settings

-  ``learning_rate`` :raw-html:`<a id="learning_rate" title="Permalink to this parameter" href="#learning_rate">&#x1F517;&#xFE0E;</a>`, default = ``0.1``, type = double, aliases: ``shrinkage_rate``, ``eta``, constraints: ``learning_rate > 0.0``

   -  shrinkage rate or damping parameter

   -  smaller values lead to higher predictive accuracy but require more computational time since more boosting iterations are needed

-  ``max_depth`` :raw-html:`<a id="max_depth" title="Permalink to this parameter" href="#max_depth">&#x1F517;&#xFE0E;</a>`, default = ``-1``, type = int

   -  maximal depth of a tree

   -  ``<= 0`` means no limit

-  ``min_data_in_leaf`` :raw-html:`<a id="min_data_in_leaf" title="Permalink to this parameter" href="#min_data_in_leaf">&#x1F517;&#xFE0E;</a>`, default = ``20``, type = int, aliases: ``min_data_per_leaf``, ``min_data``, ``min_child_samples``, constraints: ``min_data_in_leaf >= 0``

   -  minimal number of samples in a leaf

-  ``num_leaves`` :raw-html:`<a id="num_leaves" title="Permalink to this parameter" href="#num_leaves">&#x1F517;&#xFE0E;</a>`, default = ``31``, type = int, aliases: ``num_leaf``, ``max_leaves`` ``max_leaf``, constraints: ``1 < num_leaves <= 131072``

   -  maximal number of leaves of a tree

-  ``train_gp_model_cov_pars`` :raw-html:`<a id="train_gp_model_cov_pars" title="Permalink to this parameter" href="#train_gp_model_cov_pars">&#x1F517;&#xFE0E;</a>`, default = ``true``, type = bool

   -  if ``true``, the covariance parameters of the Gaussian process / random effects model are trained (estimated) in every boosting iteration of the GPBoost algorithm, otherwise not

-  ``use_gp_model_for_validation`` :raw-html:`<a id="use_gp_model_for_validation" title="Permalink to this parameter" href="#use_gp_model_for_validation">&#x1F517;&#xFE0E;</a>`, default = ``true``, type = bool

   -  set this to ``true`` to also use the Gaussian process / random effects model (in addition to the tree model) for calculating predictions on the validation data when using the GPBoost algorithm

-  ``leaves_newton_update`` :raw-html:`<a id="leaves_newton_update" title="Permalink to this parameter" href="#leaves_newton_update">&#x1F517;&#xFE0E;</a>`, default = ``false``, type = bool

   -  if ``true``, a Newton update step is done for the tree leaves after the gradient step

   -  applies only to the GPBoost algorithm for Gaussian data and cannot be used for non-Gaussian data


Note that GPBoost uses the LightGBM tree growing algorithm which grows trees using a leaf-wise strategy. I.e. trees are grown by splitting leaf nodes that maximize
the information gain until the maximal number of leaves ``num_leaves`` or the maximal depth of a tree ``max_depth`` is
attained, even when this leads to unbalanced trees. This in contrast to a depth-wise growth strategy of other boosting
implementations which builds more balanced trees. For shallow trees, small ``max_depth``, there is likely no difference between these two tree growing strategies.
If you only want to tune the maximal depth of a tree ``max_depth`` parameter and not the ``num_leaves`` parameter, it is recommended that you set the ``num_leaves`` parameter to a large value such as 131072.

Other regularization parameters
-------------------------------
-  ``lambda_l1``, ``lambda_l2`` and ``min_gain_to_split``

..
    Categorical features
    --------------------

    The tree building algorithm of GPBoost (i.e. the LightGBM tree building algorithm) can use categorical features directly (without one-hot encoding). It is common to represent categorical features with one-hot encoding, but this approach is suboptimal for tree learners. Particularly for high-cardinality categorical features, a tree built on one-hot features tends to be unbalanced and needs to grow very deep to achieve good accuracy.

    Instead of one-hot encoding, the optimal solution is to split on a categorical feature by partitioning its categories into 2 subsets. If the feature has ``k`` categories, there are ``2^(k-1) - 1`` possible partitions.
    But there is an efficient solution for regression trees `Fisher (1958) <http://www.csiss.org/SPACE/workshops/2004/SAC/files/fisher.pdf>`_. It needs about ``O(k * log(k))`` to find the optimal partition.
    The basic idea is to sort the categories according to the training objective at each split.

    For further details on using categorical features, please refer to the ``categorical_feature`` `parameter <./Parameters.rst#categorical_feature>`__.


Histogram-based tree growing algorithm
--------------------------------------
LightGBM, and thus GPBoost, uses histogram-based algorithms `[1, 2, 3] <#references>`__, which bucket continuous feature (covariate) values into discrete bins. This speeds up training and reduces memory usage.

-  ``max_bin`` :raw-html:`<a id="max_bin" title="Permalink to this parameter" href="#max_bin">&#x1F517;&#xFE0E;</a>`, default = ``255``, type = int, constraints: ``max_bin > 1``

   -  max number of bins that feature values will be bucketed in

   -  small number of bins may reduce training accuracy but may increase general power (deal with over-fitting)


Missing Value Handle
--------------------

-  Missing values are handled by default. Disable it by setting ``use_missing=false``.


Gaussian process and random effects parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Below is a list of parameters for specifying ``GPModel`` objects for modeling Gaussian processes and grouped random effects
and for specifying how these models are trained. These parameters are documented in a generic manner in the form they are
used in the R and Python package. The C API works slightly different.

Model specification parameters
------------------------------

-  ``likelihood`` : string, (default="gaussian")

   -  Likelihood function of the response variable = distribution of the label variable

   -  Currently supported values: "gaussian", "bernoulli_probit" (="binary"), "bernoulli_logit", "poisson", "gamma"

-  ``group_data`` : two dimensional array / matrix of doubles or strings, optional (default=None)

   -  Labels of group levels for grouped random effects

-  ``group_rand_coef_data`` : two dimensional array / matrix of doubles or None, optional (default=None)

   -  Covariate data for grouped random coefficients

-  ``ind_effect_group_rand_coef`` : integer vector / array of integers or None, optional (default=None)

   -  Indices that relate every random coefficients to a "base" intercept grouped random effect. Counting starts at 1.

-  ``gp_coords`` : two dimensional array / matrix of doubles or None, optional (default=None)

   -  Coordinates (features) for Gaussian process

-  ``gp_rand_coef_data`` : two dimensional array / matrix of doubles or None, optional (default=None)

   -  Covariate data for Gaussian process random coefficients

-  ``cov_function`` : string, (default="exponential")

   -  Covariance function for the Gaussian process. The following covariance functions are available: "exponential", "gaussian", "matern", and "powered_exponential". We follow the notation and parametrization of Diggle and Ribeiro (2007) except for the Matern covariance where we follow Rassmusen and Williams (2006)

-  ``cov_fct_shape`` : double, (default=0.)

   -  Shape parameter of covariance function (=smoothness parameter for Matern covariance, irrelevant for some covariance functions such as the exponential or Gaussian)

-  ``vecchia_approx`` : bool, (default=False)

   -  If true, the Vecchia approximation is used

-  ``num_neighbors`` : integer, (default=30)

   -  Number of neighbors for the Vecchia approximation

-  ``vecchia_ordering`` : string, (default="none")

   -  Ordering used in the Vecchia approximation. "none" means the default ordering is used, "random" uses a random ordering

-  ``vecchia_pred_type`` : string, (default="order_obs_first_cond_obs_only")

   -  Type of Vecchia approximation used for making predictions.

   -  "order_obs_first_cond_obs_only" = observed data is ordered first and the neighbors are only observed points, "order_obs_first_cond_all" = observed data is ordered first and the neighbors are selected among all points (observed + predicted), "order_pred_first" = predicted data is ordered first for making predictions, "latent_order_obs_first_cond_obs_only" = Vecchia approximation for the latent process and observed data is ordered first and neighbors are only observed points, "latent_order_obs_first_cond_all" = Vecchia approximation for the latent process and observed data is ordered first and neighbors are selected among all points

-  ``num_neighbors_pred`` : integer or Null, (default=Null)

   -  Number of neighbors for the Vecchia approximation for making predictions

-  ``cluster_ids`` : one dimensional numpy array (vector) with integer data or Null, (default=Null)

   -  IDs / labels indicating independent realizations of random effects / Gaussian processes (same values = same process realization)


Optimization parameters
-----------------------

The following list shows options for the optimization of the variance and covariance parameters of ``gp_model`` objects
which contain Gaussian process and/or grouped random effects models. These parameters are passed to either the ``fit``
function of a ``gp_model`` object in Python and R or to the ``set_optim_params`` (and ``set_optim_coef_params``) function
prior to running the GPBoost algorithm.

-  ``optimizer_cov`` : string, optional (default = "fisher_scoring" for Gaussian data and "gradient_descent" for other likelihoods)

   -  Optimizer used for estimating covariance parameters.

   -  Options: "gradient_descent" or "fisher_scoring"

-  ``optimizer_coef`` : string, optional (default = "wls" for Gaussian data and "gradient_descent" for other likelihoods)

   -  Optimizer used for estimating linear regression coefficients, if there are any (for the GPBoost algorithm there are usually no).

   -  Options: "gradient_descent" or "wls". Gradient descent steps are done simultaneously with gradient descent steps for the covariance paramters. "wls" refers to doing coordinate descent for the regression coefficients using weighted least squares

-  ``maxit`` : integer, optional (default = 1000)

   -  Maximal number of iterations for optimization algorithm

-  ``delta_rel_conv`` : double, optional (default = 1e-6)

   -  Convergence criterion: stop optimization if relative change in parameters is below this value

-  ``init_coef`` : numeric vector / array of doubles, optional (default = Null)

   -  Initial values for the regression coefficients (if there are any, can be Null)

-  ``init_cov_pars`` : numeric vector / array of doubles, optional (default = Null)

   -  Initial values for covariance parameters of Gaussian process and random effects (can be Null)

-  ``lr_coef`` : double, optional (default = 0.01)

   -  Learning rate for fixed effect regression coefficients

-  ``lr_cov`` : double, optional (default = 0.01)

   -  Learning rate for covariance parameters

-  ``use_nesterov_acc`` : bool, optional (default = True)

   -  If True Nesterov acceleration is used (only for gradient descent)

-  ``acc_rate_coef`` : double, optional (default = 0.5)

   -  Acceleration rate for coefficients for Nesterov acceleration

-  ``acc_rate_cov`` : double, optional (default = 0.5)

   -  Acceleration rate for covariance parameters for Nesterov acceleration

-  ``momentum_offset`` : integer, optional (default = 2)

   -  Number of iterations for which no mometum is applied in the beginning

-  ``trace`` : bool, optional (default = False)

   -  If True, the value of the gradient is printed for some iterations. Useful for finding good learning rates.

-  ``std_dev`` : bool, optional (default = False)

   -  If True, (asymptotic) standard deviations are calculated for the covariance parameters