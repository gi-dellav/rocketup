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

Another technique, mostly used in games like Go, is [Reinformenct Learning](https://en.wikipedia.org/wiki/Reinforcement_learning), where we try to train a Neural Network using reward function in order to optimize it to be as good as possible at the selected task (in this case, winning at UTTT).

---

There are problems with those implementations: both MCTS and RL are slow, needing either thousands of simulations or high amounts of matrix operations for determining the selected move, MCTS is not deterministic and NNs are not explainable.

Well, can we build a fast, deterministic, easily explanable alghorithm for UTTT (and potentialy ANY table-top game) that can be on par with MCTS or Neural Networks?

## Part II. A new frontier for evaluation functions

If you see the implementation of chess engines, we can see that almost all of them rely on the core concept of [evaluation functions](https://en.wikipedia.org/wiki/Evaluation_function): they are function that when given the current state of the game and a legal move, it outputs a numerical score indicating the quality of the move. Those engines then use the scores in order to select the move with the highest score (check [Minimax](https://en.wikipedia.org/wiki/Minimax) for more info).

For chess, eval functions are great, but for UTTT, being a constrained high-depth game, traditional eval function can't be implemented as there are no simple heuristic for recognizing the status of the game.

Well, how can we make accurate evaluation functions that can balance lots of different aspects while being tested as valid for playing bots (evals not just as an indicator but as a valid way to indicate the best move possible)? Our response is parametric evaluation functions.

Parameteric evalution functions are like evaluation functions but they also take as input a set of parameters used as constants or coefficents in order to calculate the real evaluation score: we can then use [global optimization alghorithms](https://en.wikipedia.org/wiki/Global_optimization) in order to find good values for those sets of parameters.

For the WeightedUTTT evalution function, we'll use this list of parameters:

- `taking_metacell`: add if the move will complete a metacell
- `stop_metacell`: add if the move will stop the enemy from completing the metacell
- `stop_win`: add if the move will stop the enemy from winning
- `double_metacell`: add if placing here will make a compleatable 2-in-a-row in that metacell
- `double_grid`: add if winning this metacell will make a compleatable 2-in-a-row of metacells

- `play_corner`: add if the move was played in the corner of the metacell
- `play_sides`: add if the move was played in the sides of the metacell
- `play_center`: add if the move was played in the center of the metacell

- `best_enemy_move`: multiply by the weight of the best move doable by the enemy
- `second_best_enemy_move`: multiply by the weight of the second best move doable by the enemy
- `apply_sigmoid`: special flag; if true, apply the [sigmoid function](https://en.wikipedia.org/wiki/Sigmoid_function) for the calculations of enemy's move weights.

(Note: if it is possible to win with one move, the score will always be equal to 10000)

The WeightedUTTT function will also take three special parameters: *n* and *k*,  with *n* equal to the number of future turns analyzed and *k* equal to the number of moves fully analized at each future turn.

*n* and *k* don't need to be optimized, don't need to change based on the values of the other parameters, and *k^n* is in a (approximately) linear relationship with the compute time and with the quality of the evaluations.

## Part III. Optimizing parameters using GAs

Now, we have our WeightedUTTT alghorithm, but we need a way to optimize its parameters, so we are going to design 2 different reward functions that act as the real "optimized" functions (think of it as a way to reward great operations and punish bad attempts, kinda like giving a treat to a dog every time he does a trick):

- *self-play reward function*: Plays against the current best set of parameters, gets a reward of +*n* if it wins and -(1/*n*) if it loses, with *n* equal to the best sets of parameters already defeated plus 1.
- *Monte Carlo's enemy reward function*: Plays 6 matches against a bot that uses the MCST with a predefined number of iterations, starts with the reward of -3 and adds 1 for each win; when it wins 4 out of 6 matches, it multiplies the number of interations by K and gives an extra bonus of +6; this allows to keep training against better and better opponents.

We are going to test both of those reward function with 3 different global optimization alghorithms; for the first alghorithm we'll try [genetic alghorithms](https://en.wikipedia.org/wiki/Genetic_algorithm), which simulate evolution creating generations and generations of individuals that have the set of parameters as their genotype.

## Part IV. Optimizing parameters using Bayesian optimization

Now we are going to try using [Bayesian optimization](https://en.wikipedia.org/wiki/Bayesian_optimization), which tries to build a probabilistic model of the reward function in order to optimize the set of results.

## Part V. Optimizing parameters using CMA-ES

Now we are going to try usign [CMA-ES](https://en.wikipedia.org/wiki/CMA-ES), which evolves a population of possible solutions, updating the [covariance matrix](https://en.wikipedia.org/wiki/Covariance_matrix) so that new samples are more likely to have better results.

## Part VI. Final results
