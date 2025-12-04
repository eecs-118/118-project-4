# EECS 118 -- P4: MDPs, Value Iteration, and Reinforcement Learning

Version 0.1

In this project lab, you will have a wonderful time implementing MDPs, and solving them using value iteration and reinforcement learning techniques.

Note: Details about how to submit your work for this project lab will be provided later on Canvas assignments page. Yes, there is only one week left in the entire quarter. But I believe in you. You can do it.

---

## Instructions

1. See `README.md` for Labs 1, 2 and 3 for setup and implementation instructions. If you already have your environment set up from previous labs, make sure you are working in a WORKSPACE in VSCode. If you are not sure, please ask the instructor for help.
2. Download `reinforcement.zip` from Canvas and unzip it into your working directory. Your folder structure should look like this:

	 ```
	 your_workspace_folder/
	 ├── reinforcement/
	 │   ├── mdp.py
	 │   ├── valueIterationAgents.py
	 │   ├── qlearningAgents.py
	 │   ├── pacman.py
	 │   ├── autograder.py
	 │   └── (other files)
	 :
	 └── (other files and folders from previous labs)
	 ```
3. In your terminal (`Ctrl/Cmd + J` in VSCode), go to the `reinforcement` directory:

	 ```bash
	 cd reinforcement
	 ```
	 
4. Test your environment setup by running the GridWorld environment with manual controls:
	 ```bash
	 python gridworld.py -m
	 ```
	 You should see a window pop up with a grid world. Use the arrow keys to move your catman around the grid. Notice how catman slips occasionally and ends up in an unintended direction. Close the window to proceed. 
	 
5. (Optional) You can also try a random agent:
	 ```bash
	 python gridworld.py -h
	 ```
	 This will run a random agent in the gridworld environment. Close the window to proceed.
	 

You're all set and ready to go. If you like, you can pass any of the following arguments to `gridworld.py` to customize the environment:
```log
  -h, --help            show this help message and exit
  -d DISCOUNT, --discount=DISCOUNT
                        Discount on future (default 0.9)
  -r R, --livingReward=R
                        Reward for living for a time step (default 0.0)
  -n P, --noise=P       How often action results in unintended direction
                        (default 0.2)
  -e E, --epsilon=E     Chance of taking a random action in q-learning
                        (default 0.3)
  -l P, --learningRate=P
                        TD learning rate (default 0.5)
  -i K, --iterations=K  Number of rounds of value iteration (default 10)
  -k K, --episodes=K    Number of epsiodes of the MDP to run (default 1)
  -g G, --grid=G        Grid to use (case sensitive; options are BookGrid,
                        BridgeGrid, CliffGrid, MazeGrid, default BookGrid)
  -w X, --windowSize=X  Request a window width of X pixels *per grid cell*
                        (default 150)
  -a A, --agent=A       Agent type (options are 'random', 'value', 'q', and
                        'learn', default random)
  -t, --text            Use text-only ASCII display
  -p, --pause           Pause GUI after each time step when running the MDP
  -q, --quiet           Skip display of any learning episodes
  -s S, --speed=S       Speed of animation, S > 1.0 is faster, 0.0 < S < 1.0
                        is slower (default 1.0)
  -m, --manual          Manually control agent
  -v, --valueSteps      Display each step of value iteration
```

### Working with the Code

You are only allowed to modify the following files:
- `reinforcement/valueIterationAgents.py`: Here, you will implement the Value Iteration algorithm (E41).
- `reinforcement/analysis.py`: Here, you will design reward structures for different strategies (E42).
- `reinforcement/qlearningAgents.py`: Here, you will implement the Q-Learning algorithm (E43)


---

## E41 -- Value Iteration

Implement the `ValueIterationAgent` class in `valueIterationAgents.py`.
To do so, you will need to complete the following methods:
- `runValueIteration()`: This method should run the value iteration algorithm for a specified number of iterations. In each iteration, you should update the value of each state based on the Bellman equation.
- `computeActionFromValues(state)`: Given a state, this method should return the best action to take in that state according to the current value function stored in `self.values`.
- `computeQValueFromValues(state, action)`: Given a state and an action, this method should return the Q-value of that action in that state according to the current value function stored in `self.values`.

To test your implementation, run the following command:
```bash
python gridworld.py -a value -i 50 -k 10
```
- `-a value` specifies that we want to use the Value Iteration agent.
- `-i 50` specifies that we want to run 50 iterations of value iteration.
- `-k 10` specifies that we want to run 10 episodes of the MDP.

Check your score by running the autograder:

```bash
python autograder.py -q E41
```

After 100 iterations, you should get the following values for the states in the BookGrid:
![alt text](/assets/values_after_100_iterations.png)
![alt text](/assets/qvs_after_100_iterations.png)

---

## E42 -- Designing a Reward Structure

Design a reward structure for the CliffGrid environment that encourages the agent to reach the goal while avoiding the cliff.

To do so, you will need to modify `reinforcement/analysis.py` to set appropriate values for the discount factor `answerDiscount`, noise `answerNoise`, and living reward `answerLivingReward`. Also, uncomment either `return answerDiscount, answerNoise, answerLivingReward` or `return 'NOT POSSIBLE'` as appropriate.

1. `question2a()`: The agent should prefer the close exit (square marked +1) while risking the cliff (squares marked -10).
   Hint: The close exit is closer than the far exit (+10), so you want to encourage shorter paths. You encourage shorter paths by setting a small negative living reward and a moderate discount factor. Make sure that the combined effect of discounting and living reward makes longer paths less desirable. Also, set the noise to zero to eliminate the risk of falling off the cliff due to stochastic movements.
2. `question2b()`: Close exit (+1) good, cliff (-10) not so bad (can risk it).
3. `question2c()`: Far exit (+10) good, cliff (-10) not so bad (can risk it).
4. `question2d()`: Far exit (+10) good, cliff (-10) very bad (can't risk it).
5. `question2e()`: Stay in limbo. Neither heaven (exit) nor hell (cliff).

To visualize the effect of your reward structure, run the following command:
```bash
python gridworld.py -g DiscountGrid -a value --discount [YOUR_DISCOUNT] --noise [YOUR_NOISE] --livingReward [YOUR_LIVING_REWARD]
```

For example, you can run:
```bash
python gridworld.py -g DiscountGrid -a value --discount 0.9 --noise 0.2 --livingReward -1.0
```
Check your score by running the autograder:

```bash
python autograder.py -q E42
```

---

## E43 -- Q-Learning

You will implement the `QLearningAgent` class in `qlearningAgents.py`.
To do so, you will need to complete the following methods in `qlearningAgents.py`:
- `update(state, action, nextState, reward)`: This method should update the Q-value for the given state and action based on the observed transition to `nextState` with the given `reward`.
- `computeValueFromQValues(state)`: This method should return the maximum Q-value for the given state over all possible actions.
- `getQValue(state, action)`: This method should return the Q-value for the given state and action. If the state-action pair has not been seen before, it should return 0.0.
- `computeActionFromQValues(state)`: This method should return the action that has the highest Q-value for the given state.

To visualize your Q-learning agent in action, run the following command:
```bash
python gridworld.py -a q -k 5 -m
```
- `-a q` specifies that we want to use the Q-Learning agent.
- `-k 5` specifies that we want to run 5 episodes of the MDP.
- `-m` allows manual control, so you can see how the agent learns over time.


Check your score by running the autograder:

```bash
python autograder.py -q E43
```

---
![Mr Catman](/assets/mr_catman.jpg)
That's it for now, folks.
And that's it for this course.
Good luck on your finals!

