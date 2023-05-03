# Midterm Project

For this project we will use the [Gaia DR3 data release](https://gea.esac.esa.int/archive/) to explore the solar neighborhood. This is the same selection of data we've explored recently, which can be pulled from the Gaia data archive with the following query:

```sql
SELECT TOP 300000 phot_g_mean_mag+5*log10(parallax)-10 AS mg, bp_rp, parallax FROM gaiadr3.gaia_source
WHERE parallax_over_error > 10
AND parallax > 10
AND phot_g_mean_flux_over_error>50
AND phot_rp_mean_flux_over_error>20
AND phot_bp_mean_flux_over_error>20
AND phot_bp_rp_excess_factor < 1.3+0.06*power(phot_bp_mean_mag-phot_rp_mean_mag,2)
AND phot_bp_rp_excess_factor > 1.0+0.015*power(phot_bp_mean_mag-phot_rp_mean_mag,2)
AND visibility_periods_used>8
AND astrometric_chi2_al/(astrometric_n_good_obs_al-5)<1.44*greatest(1,exp(-0.4*(phot_g_mean_mag-19.5)))
```

The data is also in the `data/` directory of this repository.

Using a subset of this data, we're going to focus on constructing models and inferring the parameters of those models using `NumPyro`.

## Data Down-Selection

To avoid lengthy run times we'll focus on only the nearby data.  Select out the objects with parallax > 40 milliarcseconds.


# Part I

For this part we'll remove (crudely) the white dwarf population (the dim secondary population of stars).  To do this we'll use a simple linear boundary to divide the population:

```python
x1, y1 = 0, 8
x2, y2 = 3, 16.5
m_div = (y2 - y1)/(x2 - x1)
b_div = y1 - m_div * x1
```

where we'll be focusing on the objects with $M_G < m_\mathrm{div} (BP-RP) + b_\mathrm{div}$.

1. Construct a `NumPyro` model that models the population as a line, assuming $M_G$ to be distributed normally about this line with a standard deviation to be inferred from the data.  Perform an MCMC.

    A. Discuss the differences between the prior distributions you choose for your model parameters and the marginal (i.e., one-dimensional) posterior distributions estimated with your MCMC.  Did you choose suitably uninformed priors?

    B. Plot the models corresponding to 50 randomly chosen posterior samples from your MCMC.  Discuss the quality of the fit, and in particular the role that the standard deviation parameter is playing in this fit.

2. Construct a new model that models the relation between $(BP-RP)$ and $M_G$ quadratically.  Carefully choose priors; be sure to explore choices for locations and scales that are appropriate for the new parameters.  Run an MCMC.

    A. Discuss the differences between the prior distributions you choose for your model parameters and the marginal (i.e., one-dimensional) posterior distributions estimated with your MCMC.  Did you choose suitably uninformed priors?

    B. Plot the model corresponding to 50 randomly chosen posterior samples from your MCMC.  Discuss the quality of the fit.

    C. **510 students:** Use your posterior estimates to predict the distribution we would expect for MG values of main sequence stars with color $(BP-RP) = 6$.

3. **510 students:** Explore higher order polynomials a bit.  How does the (qualitative) quality of the fit behave?

# Part II

Now let's bring the white dwarfs back into the mix, focusing on all the objects with parallax > 40 milliarcseconds.

1. Instead of using the point-estimate provided above for a dividing line between these two populations, let's infer it from the data.  Build a `NumPyro` model with parameters describing this line, in addition to lines individually describing the stars above and below this division (assume the scatter about these lines are described by the same sigma parameter for simplicity).

    **Tips:**
    1. Recall our outlier model for machinery for choosing parameters based on the values of other parameters.
    1. Expect lots of divergences, and don't worry too much about them if the traces otherwise look OK.
2. Plot all three lines for each of 50 randomly chosen posterior samples from your MCMC.  Discuss the quality of the fits, and in particular the constraints on the line dividing the populations.  Any ideas why sampling might be difficult?

3. **510 students:** Instead of a dividing line, try modeling these two populations using categorical parameters _a la_ our outlier model (also called a latent variable model).  How do the results compare to the linear division?  Are there any objects with uncertain categorizations?  Highlight the posterior distribution for the fraction of stars that are white dwarfs.