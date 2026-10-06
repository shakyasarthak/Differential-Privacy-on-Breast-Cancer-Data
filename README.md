# Differential-Privacy-on-Breast-Cancer-Data

This project looks at what happens to basic statistics when we add differential privacy (DP) noise. I use the malignant patients from the Wisconsin Diagnostic Breast Cancer dataset and release five statistics for five features

## What it does

- Loads the breast cancer dataset and keeps only the malignant patients (212 records)
- Uses 5 features: mean radius, texture, perimeter, area and smoothness
- Calculates 5 statistics for each: mean, standard deviation, Q1, median and Q3
- Adds Laplace noise to each statistic using a total privacy budget of epsilon = 1
- Repeats the whole experiment 1000 times and compares the noisy results with the true values

## How it works

1. **Bounds.** Each feature is clipped to a fixed range [0, upper bound]. The range R = upper - lower sets how much noise is needed
2. **Sensitivity.** I use replace-one neighbors and treat n = 212 as public
   - Mean: R / n
   - Standard deviation: R / sqrt(n)
   - Quartiles: R
3. **Noise.** Laplace noise with scale = sensitivity / epsilon
4. **Budget.** 5 features x 5 statistics = 25 queries, so each query gets epsilon = 1 / 25 = 0.04 (basic composition)
5. **Post-processing.** Results are clipped to the bounds (std is kept at 0 or above) and the quartiles are sorted. This costs no extra privacy

I add the noise two ways: `np.random.laplace` and my own Laplace function (the difference of two exponential samples)

## Results

- The mean stays close to the true value on average, but any single noisy answer can still be far off
- Std and quartiles are mostly noise, because one person can change them much more than the mean
- numpy Laplace and my custom Laplace give the same results

## Limitations

- The bounds were rounded up from the dataset's own min and max. In a real deployment they should come from public knowledge, not from the data
- Laplace noise on quartiles is a poor fit. The exponential mechanism would do much better
- The 1000 repeats are only to measure noise. Releasing 1000 answers in real life would cost 1000 times the privacy budget
- My Laplace function is for learning, not for production use

## How to run

```bash
pip install numpy pandas matplotlib scikit-learn scipy ucimlrepo
jupyter notebook dp_descriptive_stat.ipynb
```

# Possible improvements

- Use fewer queries or split the budget unevenly
- Tighter bounds
- Exponential mechanism for quartiles
- More data (sensitivity shrinks as n grows)
