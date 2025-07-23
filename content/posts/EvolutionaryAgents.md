---
title: "Training Evolutionary Agents"
featured_image: '/images/EvolutionaryAgents/teaser.png'
description: "Evolution, Neural Networks and Simulations"
date: "2025-01-22"
tags: ["Evolutionary Algorithms", "Neural Networks"]
---

{{<youtube 0v2ELbM7Vz8>}}

Last time, I showcased some basics of [Evolutionary Algorithms]({{< ref "/posts/EvolutionaryAlgorithms.md" >}}), which can elegantly solve black box problems. Today I extend it to train agents using reinforcement learning and neural networks. If you want to check it out for yourself, feel free to look at the [source code](https://github.com/JorisAR/EvolutionaryAgents).

Reinforcement learning is a branch of machine learning where agents aim to learn some behavior that maximizes the reward obtained in some environment by taking the right actions given a particular state. While many ways and options exist, today I will focus on a rather simplified method, infused with evolutionary algorithms.

## Setup
Our goal is to essentially create a brain for an agent. This brain will be nothing more than a function that takes in a vector representing the state the agent is in, and the output indicates the action it should take. For instance, the agent may know its position, and may output a direction to move into next. In our case we represent this brain that processes the input state vector into an action output vector using a neural network.

{{< figure src="/portfolio/images/EvolutionaryAgents/nn.png" width=100% >}}

A simple feedforward neural network is nothing more than a function that transforms a vector into another vector. It does this by moving the input through several layers, where each layer is fully connected to the next one. It’s extremely powerful, as we can approximate many functions by finding the right set or parameters. These parameters are referred to as weights and biases, essentially varying how strong connections are within the network. As these only scale and offset values at nodes in the network, we need some non-linear activation function at each node to enable approximating a wider set of functions, which most commonly seems to be ReLu nowadays.

## Evolution
While this idea is rather straightforward, it leaves the challenge of finding the right parameters. Last time, I discussed how [Evolutionary Algorithms]({{< ref "/posts/EvolutionaryAlgorithms.md" >}}). only require a fitness evaluation per individual to search for an optimal solution. Thus, we can train our networks by creating a population of valid neural network configurations. For a particular amount of layers and connections, we can compute the required amount of weights and biases. Those concatenated together form one large string of floats, which will be the genome of each individual.


Then, to calculate fitness, we use a simulation. For each individual, we initialize an agent with a neural network using the individual’s genome. We then let the simulation play out, and hand out rewards depending on what happens to the agent. I refer to the accumulated reward at the end of the simulation as the agent’s fitness. Now that we have gathered fitness values for each genome, we can evolve the population, and repeat.


## Example: Finding Reward
{{< figure src="/portfolio/images/EvolutionaryAgents/example1.png" width=100% >}}
Arguably the most important part of the process is setting up the environment, as it greatly dictates the success of the algorithm. For an initial example, we want the agent to learn to walk to the large reward. It gets a 2-dimensional vector as input, which is its horizontal position on the x-z plane. As output, we have a 5-dimensional vector, where the largest value dictates the taken action. The five actions are move left, move right, move up, move down, or stand still. The order of inputs and outputs does not matter as long as it’s consistent, as the training should find a correct mapping between input and output. To set up the neural network, we set the first and the last layer sizes to the values we just discussed, and I use a small amount of hidden layers, as I do not expect the function to be very complicated.

Then, we set up the reward system. First, every tick it gets a penalty of -0.1, as we want it to walk to the goal as fast as possible. The red target acts as a trap of sorts, and gives a small reward of 5. Finally, the green target gives a large reward of 50. This structure should incentivize the agent to walk to the green reward as fast as possible. 


{{< figure src="/portfolio/images/EvolutionaryAgents/results1.png" width=100% title="Notice how it falls for the red trap first, but discovers the green better reward next." >}}




## Example: Car Racing
{{< figure src="/portfolio/images/EvolutionaryAgents/example2.png" width=100% >}}
Now a more interesting example is one where the agent learns to steer through a parkour for as long as possible without crashing. As input, we give each agent an 11-dimensional vector, containing its velocity, position and distance to the nearest object along 5 distinct rays relative to the body. To make it more generalizable, it may be better to not use position, and instead transform the velocity into local space, but this will do for our example. The output is a 3-dimensional vector. I use softmax to convert it into a discrete probability mass function, and randomly sample it to either: steer left or right at constant rates, or don’t do anything at all. By picking the action based on a weighted random distribution as opposed to selecting the action with the largest magnitude, we allow for more exploration during the training process.


As our goal is to train agents to drive without crashing, we reward it for driving longer distances. We do this by evenly spacing reward checkpoints along the track. When an agent collides with one, its reward increases by one. When it collides with the wall, it is punished by deducting 5 reward points. This is on top of the opportunity loss of not being able to collect more rewards that round, as it crashed and cannot move anymore. Therefore, the car is incentivized to drive for as long as possible without hitting the walls. The steering angle is too shallow for it to constantly rotate in place without hitting the walls, so I do not foresee that as a possible exploitation of the fitness function.

{{< figure src="/portfolio/images/EvolutionaryAgents/results2.png" width=100% title="At around 175 generations, it learned to drive the track. Time is cut at 3 minutes in simulation time, so that's why it spikes to ~150 fitness." >}}

## Conclusion
While often some form of gradient descent and backwards propagation is used to train neural networks, I wanted to show that EAs can provide a possible alternative. They have some wonderful benefits, such as being insanely parallelizable, and they are more resistant to noise. Even in combination with other techniques, we can use EAs to vary hyperparameters such as network size and learning rate, to come to an optimal neural network faster.

I have made all relevant [code open source](https://github.com/JorisAR/EvolutionaryAgents), and usable as a godot GDExtension. I do not believe it is fully production ready, but I hope it can still be of some use regardless.


