+++
title = 'Analysis of LLM Advancement'
date = 2026-01-06T11:31:49+02:00
type = "post"
draft = false
+++

*You can find the source code for the plot-generation scripts [here](www.github.com/gi-dellav/llm-growth-plots).*

---

In this article, I'll show the data behind the current development of LLMs and I'll try to make hypotesis on the near future of LLMs.

## I. Pareto Frontier

## II. The Great Doubling

## III. Where Are We?

We then try to plot the linear and quadratic functions that are as close as possible to the given data points, obtaining:

- an MSE of 26.88 for the linear function (RMSE = 5.18; calculated on the Intelligence plot)

- an MSE of 7.78 for the quadratic function (RMSE = 2.79; calculated on the Intelligence plot)

Therefore, we can infer that the quadratic function is the one that best describes the Pareto frontier and that it is ~1.85x more accurate than the linear function.

### Evolution of LLMs' Intelligence

![plot](/llm-growth-score-time.png)

#### Evolution of LLMs' Intelligence (only open-weight models)

![plot](/llm-score-time-open-weights.png)

### Evolution of LLMs' Efficency

![plot](/llm-growth-score-price-time.png)

### Evolution of LLMs' Speed

![plot](/llm-growth-score-speed.png)

<br><br>

---

## IV. Future Predictions

From the given data, I calculated the best approximation of a linear and of a quadratic function to approximate

|                              | +50% [*Linear*] | +50% [*Quadratic*] | +100% [*Linear*] | +100% [*Quadratic*] |
|------------------------------|---------------|------------------|----------------|-------------------|
| Intelligence                 |   2027-12-05   |  2026-08-26    |  **2029-04-16**              | **2027-02-25**                  |
| Intelligence (*openweights*)  |  2029-05-18             |  2026-11-24     | 2031-01-15  | 2027-05-06      |
| Efficency                | 2028-05-11     |  2026-07-16   | 2030-02-25   | **2027-02-12**   |
| Speed                    | 2030-07-03     |  2026-11-26      | 2032-12-19        |  **2027-06-08**     |


## V. Conclusion

---

*P.S.* This post will have new versions when new SOTA models are released, with updated plots and predictions.

