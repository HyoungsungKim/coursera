# Rinforcement Learning 

Week1

***Reinforcement learning is different from supervised learning,*** the kind of learning studied in most current research in the field of machine learning.

- *Supervised learning* is learning from a training set of labeled examples provided by a knowledgable external supervisor.
- 감독 학습은 외부의 지능이 있는 감독에 의해 라벨링된 트레이닝 셋 예제로부터 학습 함
  - Each example is a description of a situation together with a specification***—the label—***of the correct action the system should take to that situation, which is often to identify a category to which the situation belongs.
  - The object of this kind of learning is for the system to extrapolate(추론하다), or generalize, its responses so that it acts correctly in situations not present in the training set.
  - This is an important kind of learning, ***but alone it is not adequate for learning from interaction.*** In interactive problems it is often impractical to obtain examples of desired behavior that are both correct and representative of all the situations in which the agent has to act. In uncharted(지도에 표시되어 있지 않은) territory—where one would expect learning to be most beneficial—***an agent must be able to learn from its own experience.***
- 지도학습은 라벨링 된 데이터에 적절하게 동작하지 않음
  - 에이전트는 경험으로부터 학습 할 수 있어야 함

***Reinforcement learning is also different from what machine learning researchers call unsupervised learning,*** which is typically about finding structure hidden in collections of unlabeled data.

- 강화학습은 라벨링 되지 않는 데이터의 집합으로부터 특징을 찾는 비지도 학습과도 다름
- The terms supervised learning and unsupervised learning would seem to exhaustively classify machine learning paradigms, but they do not.
- Although one might be tempted to think of reinforcement learning as a kind of unsupervised learning because it does not rely on examples of correct behavior, ***reinforcement learning is trying to maximize a reward signal instead of trying to find hidden structure.***
- 강화학습은 숨겨진 구조를 찾으려 노력하는게 아니라, 보상 신호를 최대화 하기 위해 노력 함
- Uncovering structure in an agent’s experience can certainly be useful in reinforcement learning, but by itself does not address the reinforcement learning problem of maximizing a reward signal.

***We therefore consider reinforcement learning to be a third machine learning paradigm,*** alongside supervised learning and unsupervised learning and perhaps other paradigms.

- 따라서 우리는 강화학습을 세번째 머신러닝 패러다임으로 고려 함.

One of the challenges that arise in reinforcement learning, and not in other kinds of learning, is the ***trade-off between exploration(탐험) and exploitation(이용).***

- ***To obtain a lot of reward, a reinforcement learning agent must prefer actions that it has tried in the past and found to be effective in producing reward.***
- ***But to discover such actions, it has to try actions that it has not selected before.*********
  - 보상을 얻기위해 에이전트는 지금까지 해왔던 action을 시도하고 더 효과적인 보상을 찾음(***Exploration(이용)***)
  - 하지만 더 큰 리워드를 얻기 위해서는 지금까지 하지 않았던 곳을 ***Exploitation(탐험)***해야 함
  - Exploration만 하면 Exploitation이 부족해지고, Exploitation을 하면 Exploration이 부족해짐
- The agent has to exploit what it has already experienced in order to obtain reward, but it also has to explore in order to make better action selections in the future. The dilemma is that neither exploration nor exploitation can be pursued exclusively without failing at the task.
- On a stochastic task, each action must be tried many times to gain a reliable estimate of its expected reward. The exploration–exploitation dilemma has been intensively studied by mathematicians for many decades, yet remains unresolved.
  - 어떻게 나눠야 하는지 아직 풀리지 않은 문제
- Another key feature of reinforcement learning is that ***it explicitly considers the whole problem of a goal-directed agent interacting with an uncertain environment.*** 

Reinforcement learning takes the opposite tack(방향), starting with a complete, interactive, goal-seeking agent.

- All reinforcement learning agents have explicit goals, can sense aspects of their environments, and can choose actions to influence their environments.
  - Moreover, it is usually assumed from the beginning that the agent has to operate despite significant uncertainty about the environment it faces.
- When reinforcement learning involves planning, it has to address the interplay between planning and real-time action selection, as well as the question of how environment models are acquired and improved. ***When reinforcement learning involves supervised learning, it does so for specific reasons that determine which capabilities are critical and which are not.***
- ***For learning research to make progress, important subproblems have to be isolated and studied,*** but they should be subproblems that play clear roles in complete, interactive, goal-seeking agents, even if all the details of the complete agent cannot yet be filled in.

## Elements of Reinforcement Learning

Beyond the agent and the environment, one can identify four main sub elements of a reinforcement learning system:

- A ***policy***
  - ***A policy defines the learning agent’s way of behaving at a given time.***
  - Roughly speaking, ***a policy is a mapping from perceived states of the environment to actions to be taken when in those states.***
  - It corresponds to what in psychology would be called a set of stimulus–response rules or associations. In some cases the policy may be a simple function or lookup table, whereas in others it may involve extensive computation such as a search process.
  - ***The policy is the core of a reinforcement learning agent in the sense that it alone is sufficient to determine behavior.***
  - In general, ***policies may be stochastic,*** specifying probabilities for each action
- A ***reward signal***
  - ***A reward signal defines the goal of a reinforcement learning problem.***
  - On each time step, the environment sends to the reinforcement learning agent a single number called the reward. ***The agent’s sole objective is to maximize the total reward it receives over the long run.***
  - The reward signal thus defines what are the good and bad events for the agent. 
  - ***The reward signal is the primary basis for altering the policy;*** if an action selected by the policy is followed by low reward, then the policy may be changed to select some other action in that situation in the future.
  - In general, ***reward signals may be stochastic functions of the state of the environment and the actions taken.***
- A ***value function***
  - ***A value function specifies what is good in the long run.***
  - Roughly speaking, the ***value of a state*** is the total amount of reward an agent can expect to accumulate over the future, starting from that state.
    - Whereas ***rewards determine the immediate, intrinsic desirability of environmental states***
    - ***Values indicate the long-term desirability of states after taking into account the states that are likely to follow and the rewards available in those states.***
      - Rewards는 즉각적으로 얻을 수 있는 바람직한 상황을 결정함
      - Value는 계정을 보상을 얻을 수 있는 states로 데려간 뒤 장기적인 바람직한 상황을 가르킴
    - To make a human analogy(비유),
      - Rewards are somewhat like pleasure (if high) and pain (if low)
      - Whereas values correspond to a more refined and far-sighted judgment of how pleased or displeased we are that our environment is in a particular state.
      - 사람으로 비유 했을떄 Rewards 즐거움이나 고통을 나타내고,
      - Values는 우리의 환경이 특정 states일 때 조금 더 기쁨이나 슬픔의 정제되고 장기적인 판단을 나타냄
  - ***Rewards are in a sense primary(주요 감각), whereas values, as predictions of rewards, are secondary.***
    - Without rewards there could be no values, and the only purpose of estimating values is to achieve more reward.
      - Rewards 없이 values는 존재 할 수 없음
      - Values의 목적은 rewards 이기 때문에
    - Nevertheless, it is values with which we are most concerned when making and evaluating decisions. Action choices are made based on value judgments.
      - 그럼에도 불구하고 우리는 values를 가장 중요하게 고려해야 함. action은 value를 기반으로 판단하기 때문에
    - We seek actions that bring about states of highest value, not highest reward, because these actions obtain the greatest amount of reward for us over the long run.
      - 우리는 높은 reward가 아니라 높은 value를 추구해야함. 결국 이게 가장 높은 reward를 얻는 방법이기 때문에
    - Unfortunately, ***it is much harder to determine values than it is to determine rewards.***
      - 불행하게도 values를 결정하는게 reward를 결정하는 것 보다 훨씬 어려움
      - Rewards are basically given directly by the environment
      - But values must be estimated and re-estimated from the sequences of observations an agent makes over its entire lifetime.
      - Values는 계속되는 시도를 통해서 얻어야 함
      - ***In fact, the most important component of almost all reinforcement learning algorithms we consider is a method for efficiently estimating values.*** 
- Optionally, a ***model*** of the environment
  - ***Model of the environment.***
    - This is something that mimics the behavior of the environment, or more generally, that allows inferences to be made about how the environment will behave.
    - For example, given a state and action, the model might predict the resultant next state and next reward. ***Models are used for planning, by which we mean any way of deciding on a course of action by considering possible future situations before they are actually experienced.***
    - 모델이 있으면 실수를 경험으로 배우지 않고 더 좋은 방향으로 나아갈 수 있음
    - ***Methods for solving reinforcement learning problems that use models and planning are called model-based methods,*** as opposed to simpler model-free methods that are explicitly trial-and-error learners—viewed as almost the opposite of planning. In Chapter 8 we explore reinforcement learning systems that simultaneously learn by trial and error, learn a model of the environment, and use the model for planning. Modern reinforcement learning spans the spectrum from low-level, trial-and-error learning to high-level, deliberative planning.

## 1.4 Limitations and Scope

Most of the reinforcement learning methods we consider in this book are structured around estimating value functions, but it is not strictly necessary to do this to solve reinforcement learning problems.

- For example, solution methods such as genetic algorithms, genetic programming, simulated annealing, and other ***optimization methods never estimate value functions.***
- Value 측정을 최적화 하지 못하면 강화 학습으로 해결 할 수 없음

These methods apply multiple static policies each interacting over an extended period of time with a separate instance of the environment.

- The policies that obtain the most reward, and random variations of them, ***are carried over to the next generation of policies, and the process repeats.***
  - We call these ***evolutionary methods*** because their operation is analogous to the way biological evolution produces organisms with skilled behavior even if they do not learn during their individual lifetimes.
- ***If the space of policies is sufficiently small, or can be structured*** so that good policies are common or easy to find—or if a lot of time is available for the search—***then evolutionary methods can be effective.*** In addition, evolutionary methods have advantages on problems in which the learning agent cannot sense the complete state of its environment.

***Our focus is on reinforcement learning methods that learn while interacting with the environment, which evolutionary methods do not do.***

- Methods able to take advantage of the details of individual behavioral interactions can be much more efficient than evolutionary methods in many cases.
- Evolutionary methods ignore much of the useful structure of the reinforcement learning problem:
  - They do not use the fact that the policy they are searching for is a function from states to actions;
  - They do not notice which states an individual passes through during its lifetime, or which actions it selects.
  - In some cases this information can be misleading (e.g., when states are misperceived), but more often it should enable more efficient search.
    - Although evolution and learning share many features and naturally work together, ***we do not consider evolutionary methods by themselves to be especially well suited to reinforcement learning problems and, accordingly, we do not cover them in this book.***



