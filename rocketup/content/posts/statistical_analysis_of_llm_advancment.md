+++
title = 'Analysis of LLM Advancement'
date = 2026-01-06T11:31:49+02:00
type = "post"
draft = false
+++

*Discuss this article on [Hacker News]()*

---

In this article, I'll show the data behind the current development of LLMs and I'll try to formulate an hypotesis on the near future of LLMs.

## I. Pareto Frontier and the last two years

The [Pareto frontier](https://en.wikipedia.org/wiki/Pareto_front) is the set of all the values with the highest Y value on a given X value, acting as a stepped line rappresenting the best model for a given property at a given time.

As you can see in the plots below, in the last three years LLMs have kept improving, obtaining better results on the [Artifical Analysis Intelligence Index](https://artificialanalysis.ai/evaluations/artificial-analysis-intelligence-index), better speeds and better prices.

In the Intelligence plots, you can also see two relevant improvements, connected to fundumental changes to the inner workings (either in training or in inference) of LLMs:

- September 2024, OpenAI's [o1](https://en.wikipedia.org/wiki/OpenAI_o1) integrated [chain-of-thought reasoning](https://en.wikipedia.org/wiki/Prompt_engineering#Chain-of-thought) in order to allow for complex problem solving
- January 2025, Deepseek [R1](https://en.wikipedia.org/wiki/DeepSeek#R1) started using reinforcement learning from verifiable rewards, in order to optimize the model to produce better chain-of-thought tokens and so resolving more complex problems.

*(The Deepseek R1's improvement is shown in the first plot with OpenAI's o3, released in April 2025)*

The rate of improvement, with around 1 major fundumental improvement per year, will be considered later for extimating the release windows for improved models.

## II. The Great Doubling

We are currently at a stage where LLM can solve complex problems, while being inaccurate in our situations; in order to talk about the future of AI, we need to set a defined objective that describes a state where LLMs can be used for real-world usage with good accuracy, price and speed.

Seeing that we need objective and pre-defined targets, we can't use marketing terms like "AGI", "Superintelligence", or the famous "Singularity", but instead we need a single relative (so percentage improvement) statistic that can be applied to all major LLM measurments (in this case, Intelligence, Speed and Efficency).

I propose, considering the current state of [AAII](https://artificialanalysis.ai/evaluations/artificial-analysis-intelligence-index)(~48%), [FrontierMath](https://epoch.ai/frontiermath)(~30%) and of [Humanity's Last Exam](https://agi.safe.ai/)(~38%), to take as the next target for real-world LLMs to **double** our current measurments, allowing for LLMs with accurate tool calling, long-context instruction following and frontier scientific reasoning.

In this analysis, we will consider both +50% and +100% improvements, in order to make the progress simpler to visualize and to connect to the concept of fundumental technological advancment (like chain-of-thought, as saw before), and we'll call the +100% improvement *the Great Doubling*, as it will be the last major improvement needed before an LLM capable of automating half (or more) most office tasks.

## III. Where Are We?

Well, let's try to understand more about the advancement of LLMs, starting by downloading data from Artificial Analysis and plotting the Pareto frontier.

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

## IV. Predict the future

From the given data, we can calculate the best approximation of a linear and of a quadratic function to approximate

|                              | +50% [*Linear*] | +50% [*Quadratic*] | +100% [*Linear*] | +100% [*Quadratic*] |
|------------------------------|---------------|------------------|----------------|-------------------|
| Intelligence                 |   2027-12-05   |  2026-08-26    |  **2029-04-16**              | **2027-02-25**                  |
| Intelligence (*openweights*)  |  2029-05-18             |  2026-11-24     | 2031-01-15  | **2027-05-06**      |
| Efficency                | 2028-05-11     |  2026-07-16   | 2030-02-25   | **2027-02-12**   |
| Speed                    | 2030-07-03     |  2026-11-26      | 2032-12-19        |  **2027-06-08**     |

As we can see, the quadratic function predicts **Q1 2027** for the Intelligence and Efficency improvements and **Q2 2027** for the Open-Weights and Speed improvements.

We should also consider the major advancments cited in the first section: since the introducion of CoT, Intelligence scores more than doubled, and so we can extimate that 2 more major advancments are needed, and considering the rate of 1 advancment per year, we can expect the Great Doubling to happen **during 2027**.

## V. Conclusions

Considering everything that I said and adding one cushion trimester (to account for errors and unexpected delays), I extimate the release window of truly impactful LLMs to be in **Q3 2027**.

---

*Final Notes*

- You can find the source code for the plot-generation scripts [here](www.github.com/gi-dellav/llm-growth-plots).

- This post will have new versions when new SOTA models are released, with updated plots and predictions.

- Check out [this Geoffrey Hinton talk](https://www.youtube.com/watch?v=UccvsYEp9yc) if you want to understand more about the future developments of LLMs and other AI models.

- Interesting data insights released by EpochAI:
    - [AI capabilities progress has sped up](https://epoch.ai/data-insights/ai-capabilities-progress-has-sped-up)
    - [Global AI computing capacity is doubling every 7 months](https://epoch.ai/data-insights/ai-chip-production)
    - [Today’s largest data center can do more than 20 GPT-4-scale training runs each month](https://epoch.ai/data-insights/gpt-4s-trainable)
    - [Open-weight models lag state-of-the-art by around 3 months on average](https://epoch.ai/data-insights/open-weights-vs-closed-weights-models)
