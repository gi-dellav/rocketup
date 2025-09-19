+++
title = 'Beating Ultimate Tic-Tac-Toe using optimization functions'
date = 2025-08-26T11:31:49+02:00
type = "post"
draft = false
+++

*Check the source code [here](https://github.com/gi-dellav/WeightedUTTT).*


---


Ultimate Tic-Tac-Toe is a really nice game based on the classic Tic-Tac-Toe, where two players play on a 3x3 "metagrid" made of other 3x3 standard Tic-Tac-Toe grids.

When one player takes a cell, the other must play in the corresponding metacell (ex. if I played in the top-left of any metacell, the next player will play in the top-left metacell), and the objective is to put togheter three metacells in a line (just like Tic-Tac-Toe).

## Part I. The classic methods

While Tic-Tac-Toe is an extremely easy game for computers, UTTT is not; this is beacuse the amount of different combinations of the grid is exponentially bigger (3⁹ vs 3⁸¹) and beacuse of the consequences that each action brings to the rest of the game.

Most UTTT bots are implemented using [Monte Carlo Tree Search](https://en.wikipedia.org/wiki/Monte_Carlo_tree_search); more can be read online, but the core concept is to run thosands of possible randomized simulations of the rest of the match in order to select the best turn action as the one that leads to the highest amount of winning endings.

Another technique, mostly used in games like Go, is [Reinforcemenct Learning](https://en.wikipedia.org/wiki/Reinforcement_learning), where we try to train a Neural Network using reward function in order to optimize it to be as good as possible at the selected task (in this case, winning at UTTT).

---

There are problems with those implementations: both MCTS and RL are slow, needing either thousands of simulations or high amounts of matrix operations for determining the selected move, MCTS is not deterministic and NNs are not explainable.

Well, can we build a fast, deterministic, easily explanable alghorithm for UTTT (and potentialy ANY table-top game) that can be on par with MCTS or Neural Networks?

## Part II. A new frontier for evaluation functions

If you see the implementation of chess engines, we can see that almost all of them rely on the core concept of [evaluation functions](https://en.wikipedia.org/wiki/Evaluation_function): they are function that when given the current state of the game and a legal move, it outputs a numerical score indicating the quality of the move. Those engines then use the scores in order to select the move with the highest score (check [Minimax](https://en.wikipedia.org/wiki/Minimax) for more info).

For chess, eval functions are great, but for UTTT, being a constrained high-depth game, traditional eval function can't be implemented as there are no simple heuristic for recognizing the status of the game.

Well, how can we make accurate evaluation functions that can balance lots of different aspects while being tested as valid for playing bots (evals not just as an indicator but as a valid way to indicate the best move possible)? Our response is parametric evaluation functions.

Parametric evalution functions are like evaluation functions but they also take as input a set of parameters used as constants or coefficents in order to calculate the real evaluation score: we can then use [global optimization alghorithms](https://en.wikipedia.org/wiki/Global_optimization) in order to find good values for those sets of parameters.

Parametric evaultion functions can also be recursive functions, like in this case, where it has a coefficent applied to the best response turn that can be done by the opponent after our move; all of the non-recursive weights in this case are used in order to make a base evaluation, then we select the *k* best moves and calculate the recursive scores with *n* depth.

---


For the WeightedUTTT evalution function, we'll use this list of parameters:

- `take_cell`: add if the move will complete a metacell
- `take_double_cell`: add if placing here will make a compleatable 2-in-a-row in that metacell
- `take_double_grid`: add if winning this metacell will make a compleatable 2-in-a-row of metacells

---

- `stop_cell`: add if the move will stop the enemy from completing the metacell
- `stop_win`: add if the move will stop the enemy from winning
- `stop_double_cell`: add if the move will stop the enemy from making a compleatable 2-in-a-row in that metacell
- `stop_double_grid`: add if the move will stop the enemy from making a compleatable 2-in-a-row of metacells

---

- `giving_cell`: add if the move will allow the enemey to win a metacell
- `giving_double_cell`: add if the move will allow the enemy to make a compleatable 2-in-a-row in that metacell
- `giving_double_grid`: add if the move will allow the enemy to make a compleatable 2-in-a-row of metacells

---

- `play_corner`: add if the move was played in the corner of the metacell
- `play_sides`: add if the move was played in the sides of the metacell
- `play_center`: add if the move was played in the center of the metacell

---

- `best_enemy_move`: multiply by the weight of the best legal move of the enemy
- `$apply_softsign`: special flag; if true, apply the softsign function ($\{f}(x) = \frac{x}{1 + |x|}$) for the calculations of `best_enemy_mode`.
- `$ignore_giving`: special flag; if true, ignore the values of the `giving_` family of parameters if this is not the last layer of depth in the tree evaluation.

(Note 1: automatic wins are always equal to 1000000.0, automatic losses are always equal to -1000000.0; this is really important because it allows propagation, where paths that force wins or losses can be passed down long chains of multiplications [ex. tree evaluation of a move 10 turns ahead], allowing for a better selection of moves)

(Note 2: `$apply_softsign` is relevant as softsign can normalize `best_enemy_move` to a range from -1 to 1)

(Note 3: `$ignore_giving` is relevant as the recursive score calculations also calculate how good the opponent's moves are, so the `giving_` family of parameters are less relevant, and might stop us from building strategies based on giving the enemy a single positive move in order to build a longer-term strategy)

Special flags should be chosed before the optimization of the parameters, as it (slightly) changes the function that we are trying to optimize.

---


The WeightedUTTT function will also take two special parameters: *n* and *k*, with *n* equal to the number of future turns analyzed and *k* equal to the number of moves fully analized at each future turn.

*n* and *k* don't need to be optimized, don't need to change based on the values of the other parameters.

When *n* and *k* are used during the evaluation of multiple moves, another formula needs to be used in order to split the number of turns analyzed based on the number of legal moves considered (called *l*), where we find $n' = n - \log_k(l)$.

Thanks to the fact that this formula scales based on the value of *l*, we can consider small deltas between average and maximum compute time for the evaluation function (the runtime is now highly predicatable, like MCTS or NNs), being that, with $f(k,n,l) = k^{\,n - \log_k l}$ (equal to the number of evaluations for each legal move), $l*f(k,n,l)$ is equal to the total number of board evaluations and it scales linearly (with ANY value of $l>1$) with the compute time used to analyze all legal moves.


## Part III. Preparing the function

[rewrite]

Now, we have our WeightedUTTT alghorithm, but we need a way to optimize its parameters, so we are going to design 2 different reward functions that act as the real "optimized" functions (think of it as a way to reward great operations and punish bad attempts, kinda like giving a treat to a dog every time he does a trick):

- *self-play reward function*: Plays against the current best set of parameters, gets a reward of +*n* if it wins and -(1/*n*) if it loses, with *n* equal to the best sets of parameters already defeated plus 1.
- *Monte Carlo's enemy reward function*: Plays 6 matches against a bot that uses the MCST with a predefined number of iterations, starts with the reward of -3 and adds 1 for each win; when it wins 4 out of 6 matches, it multiplies the number of interations by K and gives an extra bonus of +6; this allows to keep training against better and better opponents.

We are going to test both of those reward function with 3 different global optimization alghorithms; for the first alghorithm we'll try [genetic alghorithms](https://en.wikipedia.org/wiki/Genetic_algorithm), which simulate evolution creating generations and generations of individuals that have the set of parameters as their genotype.

[/rewrite]

---


Before we start optimizing, I want to stop for a second and talk about non-stationary processes: both of these reward functions are non-stationary, as it that the same input can give different outputs at different steps without being random: this is beacuse the optimization process for both of these functions have variables that change over time, influencing the result.

Optimizing non-stationary processes is harder and this is the reason why we aren't trying some of the most standard global optimization alghorithms, like Bayesian optimiziation. More can be read [on the definition of *stationary process*](https://en.wikipedia.org/wiki/Stationary_process) and on non-stationary optimizations in [these](https://arxiv.org/abs/2506.02980) [two](https://arxiv.org/abs/1307.5449) articles.

CMA-ES, which is the optimizer that we will use for this project, should work decently with smooth non-stationary processes like our reward function.

---

We also need to talk about the bot engine, that is the component that uses the eval function in order to determine the best strategy: this might seem like a simple pick-the-best problem, and it mostly is, except for the openings, where it is better to not use eval functions (they are slow and mostly useless in this situations) and instead we use one of these two opening strategies we defined:
- *The jumper*: Starts at the center of the center minigrid if you are the first player; if it is possible to play a move that leads the enemy to an empty cell, select it, if there isn't switch to the eval-based engine
- *The drunk*: As expressed by [this 2022 article](https://arxiv.org/pdf/2207.06239), using random placement for the first 4 moves removes the risk of forced wins of the opponent; this is important if the opponent can adapt/evolve in order to find exploits in the openings (ex. using our *self-play reward function*)

Optimized parameters and special parameters (*n* and *k*) shouldn't be influenced by the different strategies used, so we'll using *the Jumper* for the *Monte Carlo's enemy reward function* and a combination of both openings for the *self-play reward function*.

After the optimization process, we'll try both strategies on real opponents (both humans and high-rollout MCTS) in order to determine the best one in most cases (a good strategy could be "play with *the Jumper*, but if it loses 3 matches in a row it means that the opponent is optimized against WeightedUTTT so we need to switch to *the Drunk*").


## Part IV. Optimizing parameters using CMA-ES

Now we are going to try usign [CMA-ES](https://en.wikipedia.org/wiki/CMA-ES), which evolves a population of possible solutions, updating the [covariance matrix](https://en.wikipedia.org/wiki/Covariance_matrix) so that new samples are more likely to have better results.

## Part V. Final results
