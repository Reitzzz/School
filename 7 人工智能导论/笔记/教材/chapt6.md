# 第6章 智能计算及其应用（导论5）

## 1. 智能计算概述
智能计算（计算智能，Computational Intelligence）不同于传统的基于逻辑推理的符号主义人工智能，它主要受自然界生物进化、群体行为或人脑机制启发，属于次符号主义。主要分支包括：进化计算、群智能算法、模糊逻辑与人工神经网络。

## 2. 进化计算与遗传算法 (Genetic Algorithm, GA)
受达尔文进化论“物竞天择，适者生存”启发的一种全局优化搜索算法。
- **基本概念**：染色体（个体方案）、基因（方案的组成部分）、种群（候选解的集合）。
- **适应度函数 (Fitness Function)**：衡量个体优劣的标准，适应度越高的个体在下一代中存活的概率越大。
- **基本操作算子**：
  1. **选择 (Selection)**：如轮盘赌选择法，保留优秀个体。
  2. **交叉 (Crossover)**：交换两个父代个体的部分基因，产生新解。
  3. **变异 (Mutation)**：以极小的概率随机改变个体的一个或多个基因，维持种群多样性。

```python
# 遗传算法基础流程伪代码
def genetic_algorithm(pop_size, max_gen):
    population = init_population(pop_size) # 随机初始化种群
    for generation in range(max_gen):
        # 计算每个个体的适应度
        fitness = [calc_fitness(ind) for ind in population]
        
        new_population = []
        while len(new_population) < pop_size:
            # 轮盘赌选择两个优秀的父代
            p1, p2 = selection(population, fitness)
            # 交叉重组产生两个子代
            c1, c2 = crossover(p1, p2)
            # 依概率触发基因变异
            mutate(c1); mutate(c2)
            new_population.extend([c1, c2])
            
        population = new_population # 子代接管成为当前种群
    return get_best_individual(population)
```

## 3. 群智能优化算法
模拟自然界生物群体的集体协同行为来求解复杂问题。
- **粒子群优化算法 (PSO)**：模拟鸟群觅食行为。每个粒子代表一个潜在解，具有位置 $x_i$ 和速度 $v_i$。粒子的速度更新依赖于其自身的历史最优位置 $pbest$ 和整个群体的全局最优位置 $gbest$：
  $$v_{i}^{(t+1)} = \omega v_{i}^{(t)} + c_1 r_1 (pbest_i - x_i^{(t)}) + c_2 r_2 (gbest - x_i^{(t)})$$
  $$x_{i}^{(t+1)} = x_i^{(t)} + v_{i}^{(t+1)}$$
- **蚁群算法 (ACO)**：模拟蚂蚁寻找食物路径的行为。主要依赖**信息素**（Pheromone）机制，路径越短、走过的蚂蚁越多，积累的信息素越浓，从而吸引后续蚂蚁的概率越大。

## 4. 模糊计算 (Fuzzy Computing)
处理具有模糊性和不确定性问题的方法。
- **模糊集合 (Fuzzy Sets)**：引入了隶属度（Degree of Membership）的概念，元素属于集合的程度用区间 $[0,1]$ 内的实数表示。
- **隶属度函数**：记为 $\mu_A(x)$，常见的函数形状有三角形、梯形和高斯型。
- **模糊逻辑推理**：包含模糊化（将精确输入转化为模糊量）、模糊规则库匹配（IF-THEN规则）、反模糊化（将模糊输出重新转换为精确量）三个步骤。