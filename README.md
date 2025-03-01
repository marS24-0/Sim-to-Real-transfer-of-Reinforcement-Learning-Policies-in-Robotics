# Sim to Real transfer of Reinforcement Learning Policies in Robotics
##### by Gabriele Spina, Marco Sorbi, Christian Montecchiani.

- [Sim to Real transfer of Reinforcement Learning Policies in Robotics](#sim-to-real-transfer-of-reinforcement-learning-policies-in-robotics)
				- [by Gabriele Spina, Marco Sorbi, Christian Montecchiani.](#by-gabriele-spina-marco-sorbi-christian-montecchiani)
	- [Report Abstract](#report-abstract)
- [Requirements](#requirements)
- [Environment](#environment)
- [Algorithms](#algorithms)
	- [REINFORCE](#reinforce)
		- [How to run the code](#how-to-run-the-code)
	- [A2C](#a2c)
		- [How to run the code](#how-to-run-the-code-1)
	- [TRPO and PPO](#trpo-and-ppo)
		- [How to run the code](#how-to-run-the-code-2)
	- [Hyperparameters tuning](#hyperparameters-tuning)
- [Uniform Domain Randomization](#uniform-domain-randomization)
	- [How to run the code](#how-to-run-the-code-3)
- [Automatic Domain Randomization](#automatic-domain-randomization)
	- [How to run the code](#how-to-run-the-code-4)
		- [Example of monitoring during training](#example-of-monitoring-during-training)

The repository contains the code for the project *Sim to Real transfer of Reinforcement Learning Policies in Robotics* for the *Machine Learning and Deep Learning* 2021/2022 class.
The repository is intended as a support tool for the report of the project and it contains examples of some well-known algorithms and methods in the fields of Reinforcement Learning and Sim-to-Real transfer. The implementation is not thought to be efficient, thus we suggest you to avoid using this repo for purposes other than educational.

## Report Abstract
Nowadays, one of the main problems of the Reinforcement Learning paradigm is its challenging application to robotics, due to the difficulties of learning in the real world and modeling physics into simulations. In this report, we explore one of the more advanced methods of Domain Randomization, Automatic DR, to find out how it deals with former methods' issues and show its strengths and weaknesses.


# Requirements
- [Mujoco-py](https://github.com/openai/mujoco-py)
- [stable-baselines3](https://github.com/DLR-RM/stable-baselines3)
- [tensorboard](https://www.tensorflow.org/tensorboard/) [^fn5]

# Environment
The tests were performed using the [Hopper environment](https://www.gymlibrary.ml/environments/mujoco/hopper/) of Mujoco. The environment contains a flat world with a one-legged robot and the goal is to teach the leg how to move (jump) forward as fast as possible. In particular, two environments (*CustomHopper-source-v0* and *CustomHopper-target-v0*) were used to perform the *Sim-to-Sim* transfer. The two environments differ for the mass of the torso (`source=2.53429174; target=3.53429174`), while the other masses are unchanged.

![image](images/env_img.png)


# Algorithms
The repository contains different implementations of RL algorithms, an implementation of *Uniform Domain Randomization* and one of *Automatic Domain Randomization* algorithm of [OpenAI](https://openai.com/) introduced in [^fn1].

## REINFORCE
Three variations of the REINFORCE algorithm, following a slightly modified version of the one proposed by [^fn2]. The implementations differ for the usage of the *baseline* term:
1. no baseline;
2. whitening transformation baseline [^fn3];
3. state-value function baseline.

### How to run the code
Running `train.py` inside each folder will start a training by *episodes* on the selected environment, with the possibility to:
- select the number of training episodes;
- resume training from a previous model.

It is suggested to run the file with `--help` to list all the options.

## A2C
Four variations of the vanilla Advantage Actor Critic algorithm, also slightly modified starting from [^fn2]. The implementations differ for:
- loss functions of the critic network (`A2C_mse` and `A2C_v`);
- whether the updates are batched or not (`A2C_batched` and `A2C_stepwise`).

### How to run the code
Running `train.py` inside each folder will start a training by *episodes* on the selected environment, with the possibility to:
- select the number of training episodes;
- select training environment;
- select testing envinroment;
- resume training from a previous model;
- save the results.

It is suggested to run the file with `--help` to list all the options.


## TRPO and PPO
TRPO and PPO were imported from the [stable-baselines3](https://github.com/DLR-RM/stable-baselines3) package and tested on the Hopper environment. In particular, PPO is the algorithm chosen for the *Domain Randomization* implementation.

### How to run the code
The `train_test.py` files will start a training session by *episodes* or by *timesteps* on the selected environment [^fn6]:
- `--train source` will train the agent on the source environment, while logging intermediate tests on both the source and the target environments;
- `--train target` will train the agent on the target environment, while logging intermediate tests on the target environment;
- select the log folder with `--source-log-path` and `--target-log-path`. If `--train target` is selected, then the value of the source log path will be ignored;
- the scripts support also tensorboard logging in the default directory `{algorithm}_train_tensorboard/`;
- the default hyperparameters are the ones found with the tuning procedure (see next section).


It is suggested to run the file with `--help` to list all the options.


## Hyperparameters tuning
Contents of Tuning folder:
- `gridsearch` for TRPO and PPO; $^1$ $^2$
- `gridsearch` for REINFORCE and A2C; $^1$ $^2$
- `utils` for REINFORCE and A2C; $^2$
- `utils` for TRPO (`TRPO_train_test.py`);
- `utils` for PPO (`PPO_train_test.py`);
- `env` (source).

To do before running the code:
- open the interested file
- change imports (TRPO/PPO or REINFORCE/A2C)
- change hyperparameters to be tuned
- set `multiple_starts` (optional, default=4)
- set `log_path` (optional, default='outputs')


$^1$ : these are the only files supposed to be runned

$^2$ : modify the import statements at first for being sure that the code is importing the correct algorithm


# Uniform Domain Randomization
Uniform Domain Randomization uses a new custom randomized environment `CustomHopper-source-randomized-v0` created for the purpose. This randomized environment allows to set the bounds of the parameter distribution and it randomly samples the parameters at the beginning of each episode.
```
env = gym.make('CustomHopper-source-randomized-v0')
env.set_bounds(bounds)
# bounds = [(a, b), ...] is a list of tuples, each containing lower and upper bounds of the distribution
env.set_debug() # prints the current values for the masses
```

## How to run the code
`UDR_env` contains the randomized source environment (*CustomHopper-source-randomized-v0*) for UDR. `UDR_train.py` is used to run the algorithm and train a new policy. It can be run using the following parameters:
- `--env-train` training environment (default: *CustomHopper-source-randomized-v0*);
- `--env-test` testing environment;
- `--eval-freq` for periodically evaluating the agent in the target environment and saving intermediate results.


 `UDR_gridsearch.py` was using for tuning the bounds of the parameters distributions. The script can be run with parameters similar to the train script. However it is required that the parameters to be tuned are modified manually. They are:
 - *scale*: scaling factor for all the three masses. The uniform distribution to be tuned will be centered in $\mu_i = mass_i\times scale$;
 - *hw*: half width of the distribution;

such that the distribution will be uniform in the interval $\left[\mu-hw, \mu+hw\right]$.


It is suggested to run both the files with `--help` to list all the options.

# Automatic Domain Randomization
Automatic Domain Randomization was introduced by [OpenAI](https://openai.com/) as an adversarial approach to the Domain Randomization techniques. Our implementation relies on the algorithms and the callbacks provided by [stable-baselines3](https://github.com/DLR-RM/stable-baselines3). For simplicity purposes the randomized environment of the UDR was not used in this context; instead we provided the `ADR` class and implemented a custom callback (`ADRCallback`) class to handle better the modification of the bounds. The two classes are thought to be complementary and they have to be used together:
- `ADR` class contains the hyperparameters of the method and the core methods for handling the buffers, evaluating the performances and updating the bounds;
- `ADRCallback` class implements the `BaseCallback` of `stable-baselines3`. It is used to call the `ADR` methods at each episode and for debugging and logging purposes.

## How to run the code
Our implementation of ADR uses PPO as default algorithm. `ADR_train.py` is used to run the algorithm and train a new policy. The code displays also a live visualization of the bounds during the training procedure for monitoring purposes (an example can be found below).
 The script can be run using the following parameters:
- hyperparameters of ADR ($p_b, m, \Delta, t_L, t_H$)
- `--step` for selecting the method for updating the bound (*contant, proportional, small_or_big, random, gaussian_noise*)
- log paths and evaluation paths for logging and saving intermediate results and target evaluations (*tensorboard* supported)
- `--model` for resuming training of a previous model

The file `plot_bounds.py` is used to plot the evolution of bounds through time. The bounds should be logged in the `logs/train.npz` file in the same folder. It is possible to plot with respect to episodes or timesteps.

The best model ever found during the training is available in the `models/best_eval` folder and a simulation can be run with the `ADR_test.py` script.

It is suggested to run the files with `--help` to list all the options.


### Example of monitoring during training
```
NUM_TIMESTEPS: 16072704
N_EPISODES: 44025

	[3.61]«---------------[3.93]----»[4.01]
	[2.53]«---------[2.71]---------»[2.9]
	[4.9]«---------[5.09]-----»[5.2]

DATA BUFFER SIZE:
{'foot_high': 2,
 'foot_low': 0,
 'thigh_high': 0,
 'thigh_low': 0,
 'leg_high': 0,
 'leg_low': 0}

LAST PERFORMANCES:
[('foot_low', 1573.02703328)]

LAST UPDATES:
[('foot_low', 'low+', 0.01679891697261309)]

LOW_TH = 1000
HIGH_TH = 1500

----------------------------------------
| ep_length               | 315        |
| ep_return               | 1.37e+03   |
| n_episodes              | 44025      |
| num_timesteps           | 16072452   |
| rollout/                |            |
|    ep_len_mean          | 363        |
|    ep_rew_mean          | 1.5e+03    |
| time/                   |            |
|    fps                  | 1841       |
|    iterations           | 1308       |
|    time_elapsed         | 8728       |
|    total_timesteps      | 16072704   |
| train/                  |            |
|    approx_kl            | 0.03323253 |
|    clip_fraction        | 0.32       |
|    clip_range           | 0.2        |
|    entropy_loss         | 1.86       |
|    explained_variance   | 0.992      |
|    learning_rate        | 0.00025    |
|    loss                 | 9.44       |
|    n_updates            | 13070      |
|    policy_gradient_loss | 0.00747    |
|    std                  | 0.132      |
|    value_loss           | 26.1       |
----------------------------------------
```

![image](ADR/bounds_constant_example.png)

[^fn1]: https://arxiv.org/abs/1910.07113
[^fn2]: https://mitpress.mit.edu/books/reinforcement-learning-second-edition
[^fn3]: https://medium.com/@fork.tree.ai/understanding-baseiline-techniques-for-reinforce-53a1e2279b57
[^fn4]: it is suggested to run the files with --help, since some of the scripts support this feature
[^fn5]: tensorboard logging is used in some of the scripts concerning TRPO, PPO and therefore also UDR and ADR
[^fn6]: the training is done by episodes only to compare the methods with REINFORCE and A2C, while generally the number of training timesteps is a more significant maeasure of the training time
