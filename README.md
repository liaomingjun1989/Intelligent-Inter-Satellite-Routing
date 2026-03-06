基于DDQN的智能网络路由系统，用于在动态网络环境中进行数据包路由决策。

## 项目简介

本项目实现了一个使用深度强化学习进行网络路由的仿真系统。系统通过DDQN算法学习最优路由策略，能够在动态变化的网络环境中高效地路由数据包。项目支持多种网络拓扑（包括Barabasi-Albert网络和卫星网络），并提供了完整的训练、测试和可视化功能。

## 主要功能

- **深度Q网络路由**：为每个目标节点训练独立的DQN网络，实现智能路由决策
- **多种算法支持**：
  - **DQN**：标准深度Q网络
  - **DDQN**：Double DQN，减少过估计问题
  - **GAT+DDQN**：图注意力网络结合DDQN，更好地利用网络拓扑结构
- **动态网络支持**：支持网络边的动态删除/重建和权重变化（正弦波、随机游走等）
- **卫星网络支持**：可以从CSV文件加载卫星网络拓扑数据
- **经验回放机制**：支持随机采样、最近经验采样和优先级采样三种经验回放策略
- **多指标统计**：跟踪平均交付时间、队列长度、节点拥塞率等多个性能指标
- **可视化分析**：自动生成训练过程和测试结果的可视化图表

## 环境要求

- Python 3.6+
- PyTorch
- NetworkX
- NumPy
- Matplotlib
- Gymnasium（推荐）或 Gym（用于环境接口）

## 安装步骤

1. **克隆或下载项目**
```bash
cd DQN-Project-Doctor
```

2. **安装依赖包**
```bash
pip install torch networkx numpy matplotlib gymnasium
```

或者使用 `requirements.txt` 文件安装：
```bash
pip install -r requirements.txt
```

**注意**：项目已更新为使用 `gymnasium`（Gym 的维护版本）。如果您的环境中已安装 `gym`，代码会自动回退使用 `gym`，但建议安装 `gymnasium` 以获得更好的兼容性。

## 项目结构

```
DQN-Project-Doctor/
├── DeepQSimulation.py      # 主程序入口，包含训练和测试流程
├── our_env.py              # 环境类，定义网络环境和路由逻辑
├── our_agent.py            # 智能体类，实现DQN/DDQN学习和决策
├── DQN.py                  # DQN神经网络结构定义
├── GAT_DQN.py              # GAT（图注意力网络）结构定义
├── neural_network.py       # 神经网络封装类，包含策略网络和目标网络
├── replay_memory.py        # 经验回放缓冲区实现
├── dynetwork.py            # 动态网络类，管理网络状态和数据包
├── Packet.py               # 数据包类定义
├── satellite_graphy.py     # 卫星网络图生成工具
├── UpdateEdges.py          # 网络边更新工具
├── Setting.json            # 配置文件（网络参数、DQN参数、训练参数等）
├── AER_data/               # 网络数据目录（需要包含result*.csv文件）
├── starlink_AER/           # 卫星网络数据目录
├── plots/                  # 输出图表目录
│   └── learnRes/          # 训练过程图表
└── network_images/         # 网络可视化图像
```

## 使用方法

### 1. 配置参数

编辑 `Setting.json` 文件来配置网络和训练参数：

```json
{
  "NETWORK": {
    "number nodes": 500,           // 网络节点数
    "edge degree": 4,               // 边度数
    "holding capacity": 75,         // 节点最大队列容量
    "sending capacity": 20,         // 节点最大发送容量
    "initial num packets": 2500,   // 初始数据包数量
    "max_additional_packets": 500, // 最大额外数据包数
    "network_type": "barabasi-albert" // 网络类型
  },
  "DQN": {
    "memory_batch_size": 16,       // 经验回放批次大小
    "memory_bank_size": 1000,      // 经验回放缓冲区大小
    "optimizer_learning_rate": 0.005 // 学习率
  },
  "AGENT": {
    "epsilon": 0.7,                // 探索率
    "decay_epsilon_rate": 0.99998, // 探索率衰减率
    "gamma_for_next_q_val": 0.6    // 折扣因子
  },
  "Simulation": {
    "training_episodes": 10,       // 训练轮数
    "max_allowed_time_step_per_episode": 100 // 每轮最大时间步数
  }
}
```

### 2. 准备网络数据

如果使用卫星网络，确保在 `starlink_AER/` 目录下有 `result1.csv`, `result2.csv` 等网络拓扑文件。

如果使用Barabasi-Albert网络，程序会自动生成。

### 3. 运行训练和测试

#### 方式一：运行单个算法

直接运行主程序，可以通过命令行参数指定算法：

**使用命令行参数（推荐）**：

```bash
# 使用 DQN 算法
python DeepQSimulation.py --algorithm dqn
# 或简写
python DeepQSimulation.py -a dqn

# 使用 DDQN 算法
python DeepQSimulation.py --algorithm ddqn
# 或简写
python DeepQSimulation.py -a ddqn

# 使用 GAT+DDQN 算法
python DeepQSimulation.py --algorithm gat
# 或简写
python DeepQSimulation.py -a gat
```

**使用配置文件**：

如果不指定命令行参数，程序将使用 `Setting.json` 中的配置：

```bash
python DeepQSimulation.py
```

程序将执行以下步骤：

1. **训练阶段**：
   - 初始化网络环境
   - 训练DQN网络多个轮次（episodes）
   - 记录训练过程中的各项指标
   - 生成训练过程图表（保存在 `plots/learnRes/`）

2. **测试阶段**：
   - 在不同网络负载下测试训练好的模型
   - 记录测试指标
   - 生成测试结果图表（保存在 `plots/`）

**注意**：命令行参数会覆盖 `Setting.json` 中的 `use_ddqn` 和 `use_gat` 设置，其他配置仍使用 `Setting.json` 中的值。

#### 方式二：运行算法对比（推荐）

运行对比脚本，自动对比 DQN、DDQN、GAT+DDQN 三种方法：

```bash
python compare_algorithms.py
```

对比脚本将：
1. 依次运行三种算法（每个算法都会完整训练和测试）
2. 收集所有算法的训练和测试结果
3. 生成对比图表（保存在 `plots/comparison/`）
4. 自动恢复原始配置文件

**注意**：对比脚本运行时间较长，因为需要训练三个算法。建议先用单个算法测试，确认无误后再运行对比。

### 4. 查看结果

训练和测试完成后，可以在以下位置查看结果：

- **训练过程图表**：`plots/learnRes/`
  - `avg_deliv_learning.png` - 平均交付时间随训练轮次的变化
  - `avg_q_len_learning.png` - 平均队列长度变化
  - `avg_perc_at_capacity_learning.png` - 节点拥塞率变化
  - `reward.png` - 奖励值变化

- **测试结果图表**：`plots/`
  - `avg_deliv.png` - 不同负载下的平均交付时间
  - `maxNumPkts.png` - 最大队列长度
  - `avg_q_len.png` - 平均队列长度
  - `avg_perc_at_capacity.png` - 节点拥塞率

- **算法对比图表**：`plots/comparison/`（运行 `compare_algorithms.py` 后生成）
  - `comparison_avg_deliv.png` - 三种算法的平均交付时间对比
  - `comparison_maxNumPkts.png` - 最大队列长度对比
  - `comparison_avg_q_len.png` - 平均队列长度对比
  - `comparison_avg_perc_at_capacity.png` - 节点拥塞率对比
  - `comparison_rejection_rate.png` - 拒绝率对比
  - `comparison_train_avg_deliv.png` - 训练过程平均交付时间对比
  - `comparison_train_reward.png` - 训练过程奖励对比

- **数据文件**：项目根目录下的 `.npy` 文件包含数值结果
  - **注意**：现在保存 `.npy` 文件时会自动生成对应的 `_plot.png` 图片文件，无需手动转换

### 5. 查看 .npy 数据文件

项目运行后会生成 `.npy` 格式的数据文件，这些文件保存了训练和测试过程中的数值结果。可以使用以下方法查看：

#### 方法一：使用查看工具脚本（推荐）

项目提供了 `view_npy.py` 工具脚本，可以方便地查看 `.npy` 文件：

```bash
# 查看基本信息（数组形状、内容、基本统计）
python view_npy.py dqn_avg_deliv.npy

# 查看详细统计信息（最小值、最大值、平均值、分位数等）
python view_npy.py dqn_avg_deliv.npy --stats

# 查看详细信息（可指定显示前N个和后N个元素）
python view_npy.py dqn_avg_deliv.npy --detailed --head 10 --tail 10

# 绘制数据图表
python view_npy.py dqn_avg_deliv.npy --plot

# 对比多个算法的数据文件
python view_npy.py dqn_avg_deliv.npy ddqn_avg_deliv.npy gat_avg_deliv.npy --compare --labels DQN DDQN "GAT+DDQN"
```

#### 方法二：使用 Python 直接加载

在 Python 中直接使用 NumPy 加载：

```python
import numpy as np

# 加载数据
data = np.load('dqn_avg_deliv.npy')

# 查看基本信息
print(f"数组形状: {data.shape}")
print(f"数组内容: {data}")

# 查看统计信息
print(f"平均值: {np.mean(data)}")
print(f"最小值: {np.min(data)}")
print(f"最大值: {np.max(data)}")
```

#### 方法三：使用命令行快速查看

```bash
# 使用 Python 一行命令查看
python -c "import numpy as np; data = np.load('dqn_avg_deliv.npy'); print(f'形状: {data.shape}'); print(f'内容: {data}'); print(f'平均值: {np.mean(data):.4f}')"
```

#### 常见数据文件说明

保存数据时会自动生成对应的图片文件（文件名后加 `_plot.png`）：

**单个算法运行结果**（项目根目录）：
- `dqn_avg_deliv.npy` / `dqn_avg_deliv_plot.png` - 平均交付时间
- `dqn_maxNumPkts.npy` / `dqn_maxNumPkts_plot.png` - 最大队列长度
- `dqn_avg_capac.npy` / `dqn_avg_capac_plot.png` - 节点拥塞率
- `dqn_avg_idle.npy` / `dqn_avg_idle_plot.png` - 拒绝率
- `dqn_avg_empty.npy` / `dqn_avg_empty_plot.png` - 空节点百分比

**算法对比结果**（`plots/` 目录）：
- `dqn_test_avg_deliv.npy` / `dqn_test_avg_deliv_plot.png` - DQN 测试阶段平均交付时间
- `ddqn_test_avg_deliv.npy` / `ddqn_test_avg_deliv_plot.png` - DDQN 测试阶段平均交付时间
- `gat_ddqn_test_avg_deliv.npy` / `gat_ddqn_test_avg_deliv_plot.png` - GAT+DDQN 测试阶段平均交付时间
- `dqn_train_avg_deliv.npy` / `dqn_train_avg_deliv_plot.png` - DQN 训练过程平均交付时间
- 等等...

**对比数据**（`plots/comparison/` 目录）：
- 每个算法的测试和训练数据都会自动生成对应的图片文件

## 配置说明

### 网络配置（NETWORK）

- `number nodes`: 网络中的节点数量
- `edge degree`: 每个新节点连接的边数（Barabasi-Albert网络）
- `holding capacity`: 每个节点可以容纳的最大数据包数
- `sending capacity`: 每个节点每个时间步可以发送的最大数据包数
- `network_type`: 网络类型，可选 `"barabasi-albert"` 或 `"gnm_random"`

### DQN配置（DQN）

- `take_queue_size_as_input`: 是否将队列大小作为网络输入（0或1）
- `memory_batch_size`: 经验回放时采样的批次大小
- `memory_bank_size`: 经验回放缓冲区的最大容量
- `optimizer_learning_rate`: 优化器学习率
- `optimize_per_episode`: 是否限制每个时间步只优化一次（0或1）
- `use_ddqn`: 是否使用 DDQN（Double DQN）算法（0或1）
  - `0`: 使用标准 DQN
  - `1`: 使用 DDQN（减少过估计，通常性能更好）
- `use_gat`: 是否使用 GAT（Graph Attention Network）作为网络结构（0或1）
  - `0`: 使用标准全连接 DQN
  - `1`: 使用 GAT（图注意力网络，更好地利用网络拓扑结构）
  - **注意**：`use_gat=1` 时，建议同时设置 `use_ddqn=1` 以获得最佳性能（GAT+DDQN）

**提示**：除了在配置文件中设置，也可以通过命令行参数 `--algorithm` 或 `-a` 来指定算法，命令行参数会覆盖配置文件中的设置。

### 智能体配置（AGENT）

- `epsilon`: 初始探索率（0-1之间）
- `decay_epsilon_rate`: 探索率衰减系数
- `gamma_for_next_q_val`: 未来奖励折扣因子
- `use_random_sample_memory`: 使用随机采样（0或1）
- `use_most_recent_memory`: 使用最近经验采样（0或1）
- `use_priority_memory`: 使用优先级采样（0或1）

**注意**：三种采样方式只能选择一种，即三个参数之和必须为1。

### 仿真配置（Simulation）

- `training_episodes`: 训练轮数
- `max_allowed_time_step_per_episode`: 每轮训练的最大时间步数
- `num_time_step_to_update_target_network`: 更新目标网络的间隔（时间步数）
- `test_network_load_min/max/step_size`: 测试时的网络负载范围
- `test_trials_per_load`: 每个负载下的测试次数
- `learning_plot`: 是否生成训练过程图表（0或1）
- `test_diff_network_load_plot`: 是否生成测试结果图表（0或1）
- `plot_routing_network`: 是否生成网络可视化图像（0或1）

## 核心算法

### DQN架构

- **输入层**：节点one-hot编码 + 可选的队列大小信息
- **隐藏层**：两层全连接层，每层大小为 `2 * num_nodes`
- **输出层**：输出每个节点的Q值
- **激活函数**：Tanh

### 经验回放策略

1. **随机采样**：从经验池中随机采样
2. **最近经验采样**：采样最近的经验
3. **优先级采样**：根据TD误差的优先级采样

### 奖励函数

- 到达目标节点：`20 * num_nodes`（大奖励）
- 正常转发：基于目标节点队列长度和增长率的负奖励
- 无法到达目标：`-50`（惩罚）

## 输出指标说明

- **平均交付时间**：数据包从生成到到达目标节点的平均时间步数
- **最大队列长度**：网络中任意节点在任意时刻的最大队列长度
- **平均队列长度**：所有非空节点的平均队列长度
- **节点拥塞率**：达到最大容量的节点百分比
- **空节点百分比**：没有数据包的节点百分比
- **数据包空闲时间**：数据包被拒绝转发的次数比例

## 注意事项

1. **内存使用**：大型网络（节点数>1000）可能需要较大内存
2. **训练时间**：训练时间取决于网络大小、数据包数量和训练轮数
3. **GPU支持**：如果安装了CUDA版本的PyTorch，程序会自动使用GPU加速
4. **数据文件路径**：确保网络数据文件路径正确，特别是卫星网络数据

## 故障排除

### 常见问题

1. **"Error: All Nodes are Full"**
   - 原因：网络负载过高，所有节点队列已满
   - 解决：减少初始数据包数量或增加节点容量

2. **"Error: Check memory type!"**
   - 原因：经验回放策略配置错误
   - 解决：确保 `use_random_sample_memory` + `use_most_recent_memory` + `use_priority_memory` = 1

3. **找不到数据文件**
   - 原因：网络数据文件路径不正确
   - 解决：确保 `AER_data/` 目录存在于项目根目录，并且包含所需的 `result*.csv` 文件。代码已自动使用相对路径，无需手动配置。

4. **Gym/Gymnasium 警告**
   - 原因：使用了旧版本的 Gym 库
   - 解决：安装 `gymnasium`：`pip install gymnasium`。代码已自动兼容，优先使用 gymnasium，如果没有则回退到 gym。

5. **"AttributeError: module 'networkx' has no attribute 'to_numpy_matrix'"**
   - 原因：NetworkX 2.6+ 版本移除了 `to_numpy_matrix` 函数
   - 解决：代码已更新为使用 `to_numpy_array`，这是推荐的替代方法。如果仍有问题，可以降级 NetworkX：`pip install networkx<2.6`

6. **"TypeError: Population must be a sequence"**
   - 原因：NetworkX 2.x 中 `edges()` 返回 `EdgeView` 对象，不是列表
   - 解决：代码已更新，在使用 `random.sample()` 前将 `edges()` 转换为列表

7. **"IndexError: only integers, slices... are valid indices (got numpy.bool)"**
   - 原因：使用 `to_numpy_array` 后，numpy bool 数组不能直接用于 PyTorch tensor 索引
   - 解决：代码已更新，将 numpy bool 数组转换为 torch bool tensor 后再进行索引

## 扩展功能

- 可以修改 `our_env.py` 中的奖励函数来优化路由策略
- 可以添加新的网络拓扑类型
- 可以实现其他强化学习算法（如A3C、PPO等）进行对比

## 许可证

本项目仅供学习和研究使用。

## 联系方式

如有问题或建议，请通过项目仓库提交Issue。
