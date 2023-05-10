# DP-GEN 构建训练数据并生成深度势模型

为了构建高质量的LiCl熔体深度势能模型的训练数据集，我们可以使用DP-GEN(Deep Potential GENerator)软件。DP-GEN是一个自动化的训练数据集生成工具，可以根据预设的参数和条件搜索材料的结构和相态空间。

## 目的

学习完本课程后你应该：

- 掌握 DP-GEN 输入文件（param.json 和 machine.json） 中主要关键词的设置；
- 利用 DP-GEN 为 LiCl 熔体构建训练数据并生成深度势模型；
- 分析和解读 DP-GEN 迭代过程中的结果和输出文件，以便更好地理解模型训练的过程和效果。

## 资源

在本教程中，我们以LiCl熔体分子为例,构建训练数据并生成深度势模型。我们已经在`work/ex4`中准备了需要的文件。

```sh
wget --content-disposition https://github.com/LiangWenshuo1118/LiCl/raw/main/work.tar.gz
tar zxvf work.tar.gz
```

首先，使用 `tree` 命令查看 `work/ex4` 文件夹。

```sh
$ tree ex4 -L 2  
```

你应该可以在屏幕上看到输出：

```sh
ex4
|-- abacus
|   |-- Cl_ONCV_PBE-1.2.upf
|   |-- Cl_gga_8au_100Ry_2s2p1d.orb
|   |-- Li_ONCV_PBE-1.2.upf
|   `-- Li_gga_8au_100Ry_4s1p.orb
|-- machine.json
`-- param_abacus.json
```

- `*.upf` 和 `*.orb` 是 ABACUS 的输入文件
- `param.json` 是运行当前任务的 DP-GEN 设置。
- `machine.json` 是一个任务调度程序，其中设置了计算机环境和资源要求。

本教程采用DeePMD-kit(2.1.5),AVACUS(3.1.0)和DP-GEN(0.11.0)程序完成。

## 作业

运行过程包含一系列连续迭代，按顺序进行，例如将系统加热到特定温度。每次迭代由三个步骤组成：探索 (Exploration)，标记 (Labeling)和训练 (Training)。

### 输入文件

首先，介绍 DP-GEN 运行过程所需的输入文件。

#### param.json

`param.json` 中的关键字可以分为 4 个部分：

- 系统和数据 (System and Data)：用于指定原子类型、初始数据等。
- 训练 (Training)：主要用于指定训练步骤中的任务;
- 探索 (Exploration)：主要用于在探索步骤中指定任务;
- 标记 (Labeling)：主要用于指定标记步骤中的任务。

这里我们以LiCl熔体为例，介绍`param.json`中的主要关键词。

**系统和数据 (System and Data)**

系统和数据相关内容：

```
{
     "type_map": ["Li","Cl"],
     "mass_map": [6.941,35.453],
     "init_data_prefix": "../ex3",
     "init_data_sys": ["00.data/training_data"],
     "sys_format": "abacus/stru",
     "sys_configs_prefix": "../ex2",
     "sys_configs": [["01.md/STRU"]],
     "_comment": " that's all ",

```  

关键词描述：

| 关键词                | 类型      | 描述                                                     |
|-----------------------|--------------|----------------------------------------------------------|
| "type_map"            | list         | 元素列表，这里是Li和Cl                                    |
| "mass_map"            | list         | 原子质量列表                                              |
| "init_data_prefix"    | str          | initial data 的前置路径                                   |
| "init_data_sys"       | list         | 初始训练数据文件的路径列表。可以使用绝对路径或相对路径        |
| "sys_format"	        | str          | 指定构型的格式                                            |
| "sys_configs_prefix"  | str          | sys_configs 的前置路径                                    |
| "sys_configs"         | list         | 构型文件的路径列表，此处支持通配符                          |

案例说明：

“type_map”和“mass_map”给出了元素类型和原子质量。在这里，系统包含锂（Li）和氯（Cl）两种，质量分别为6.941和35.453。
“init_data_prefix”和“init_data_sys”关键词共同指定了初始训练数据的位置。“sys_configs_prefix”和“sys_configs”共同指定了探索的构型的位置。“sys_format”指定了构型的格式。在这里，训练数据位于../ex3/00.data/training_data目录下。构型文件位于../ex2/01.md/STRU目录下，采用ABACUS软件的abacus/stru格式


**训练(Training)**

与训练相关的内容如下：

```
     "numb_models": FILL,
     "default_training_param": {
         "model": {
             "type_map": ["Li","Cl"],
             "descriptor": {
                 "type": "se_e2_a",
                 "sel": "auto",                
                 "rcut_smth": 0.5,
                 "rcut": 7.0,
                 "neuron": [25,50,100],
                 "resnet_dt": false,
                 "axis_neuron": 12,
                 "seed": 1
             },
             "fitting_net": {
                 "neuron": [240,240,240],
                 "resnet_dt": true,
                 "seed": 1
             }
         },
         "learning_rate": {
             "type": "exp",
             "start_lr": 0.001,
             "decay_steps": 5000
         },
         "loss": {
             "start_pref_e": 0.02,
             "limit_pref_e": 1,
             "start_pref_f": 1000,
             "limit_pref_f": 1,
             "start_pref_v": 0,
             "limit_pref_v": 0
         },
         "training": {
             "numb_steps": 400000,
             "disp_file": "lcurve.out",
             "disp_freq": 1000,
             "numb_test": 1,
             "save_freq": 10000,
             "save_ckpt": "model.ckpt",
             "disp_training": true,
             "time_training": true,
             "profiling": false,
             "profiling_file": "timeline.json",
             "_comment": "that's all"
         }
     },
```

关键词描述：

| 键词                      | 字段类型  | 描述                          |
|---------------------------|----------|-------------------------------|
| "numb_models"             | int      | 在 00.train 中训练的模型数量。  |
| "default_training_param"  | dict     | DeePMD-kit 的训练参数          |

填写提示：

训练相关键指定训练任务的详细信息。`numb_models`指定要训练的模型数量。`default_training_param`指定了 DeePMD-kit 的训练参数。在这里，将训练 4 个 DP 模型。

DP-GEN 的训练部分由 DeePMD-kit 执行，因此此处的关键字与 DeePMD-kit 的关键字相同，此处不再赘述。有关这些关键字的详细说明，请访问[DeePMD-kit 文档](https://deepmd.readthedocs.io/)。

**探索 (Exploration) **

与探索相关的内容如下：

```
     "model_devi_dt": 0.001,
     "model_devi_skip": 0,
     "model_devi_f_trust_lo": FILL,
     "model_devi_f_trust_hi": FILL,
     "model_devi_clean_traj": false,
     "model_devi_jobs":  [
        {"sys_idx": [0],"temps": [FILL,FILL,FILL,FILL],"press": [FILL,FILL,FILL,FILL,FILL], "trj_freq": FILL, "nsteps": FILL,"ensemble": "npt", "_idx": "00"},
        {"sys_idx": [0],"temps": [FILL,FILL,FILL,FILL],"press": [FILL,FILL,FILL,FILL,FILL], "trj_freq": FILL, "nsteps": FILL,"ensemble": "npt", "_idx": "01"},
        {"sys_idx": [0],"temps": [FILL,FILL,FILL,FILL],"press": [FILL,FILL,FILL,FILL,FILL], "trj_freq": FILL, "nsteps": FILL,"ensemble": "npt", "_idx": "02"}   
     ],
```

关键词描述：

| 键词                      | 字段类型                    | 描述   |
|--------------------------|-------------------------|---------------|
| "model_devi_dt"          | float  | MD 的时间步长                                                                                                                                                                                                                                |
| "model_devi_skip"        | int    | 每个 MD 中为 fp 跳过的结构数                                                                                                                                                                                                |
| "model_devi_f_trust_lo"  | float  | 选择的力下限。如果为 List，则应分别为每个索引设置sys_configs。                                                                                                                                 |
| "model_devi_f_trust_hi"  | int    | 选择的力上限。如果为 List，则应分别为每个索引设置sys_configs。                                                                                                                                  |                                                                                         |
| "model_devi_clean_traj"  | bool or int    | 如果model_devi_clean_traj的类型是布尔类型，则表示是否清理MD中的traj文件。如果是 Int 类型，则将保留 traj 文件夹的最新 n 次迭代，其他迭代将被删除。  |
| "model_devi_clean_traj"  | bool           | 控制在模型偏差（model_devi）阶段是否合并生成的轨迹文件|
| "model_devi_jobs"        | list            | 01.model_devi 中的探索设置。列表中的每个字典对应于一次迭代。model_devi_jobs 的索引与迭代的索引完全一致                                               |
| &nbsp;&nbsp;&nbsp;&nbsp;"sys_idx"   | List of integer         | 选择系统作为MD的初始结构并进行探索。序列与“sys_configs”完全对应。 |
| &nbsp;&nbsp;&nbsp;&nbsp;"temps" | list  | 分子动力学模拟的温度 (K)
| &nbsp;&nbsp;&nbsp;&nbsp;"press" | list  | 分子动力学模拟的压力 (Bar) 
| &nbsp;&nbsp;&nbsp;&nbsp;"trj_freq"   | int          | MD中轨迹的保存频率。                  |
| &nbsp;&nbsp;&nbsp;&nbsp;"nsteps"     | int          | 分子动力学运行步数                                 |
| &nbsp;&nbsp;&nbsp;&nbsp;"ensembles"  | str          | 决定在 MD 中选择的集成算法，选项包括 “npt” ， “nvt”等. |

填写提示：

在在“model_devi_jobs”中设置了三次迭代。对于每次迭代，在不同的温度（900, 1000, 1100和1200 K）和压力条件（0, 10, 100, 1000和10000 Bar）下，使用"npt"系综和“sys_configs_prefix”和“sys_configs”指定的构型进行100000步模拟，时间步长为0.001 ps。我们选择保存 MD 轨迹文件，并将保存频率“trj_freq”设置为 10。如果轨迹中构型的“max_devi_f”介于 0.08 和 0.18 之间，DP-GEN 会将该结构视为候选结构。如果要保存 traj 文件夹的最近n次迭代，可以将“model_devi_clean_traj”设置为整数。

** 标记 (Labeling)**

与标记相关的内容如下：

```
    "fp_style": "abacus",
    "shuffle_poscar": false,
    "fp_task_max": FILL,
    "fp_task_min": FILL,
    "fp_pp_path": "./abacus",
    "fp_pp_files": ["Li_ONCV_PBE-1.2.upf","Cl_ONCV_PBE-1.2.upf"],
    "fp_orb_files": ["Li_gga_8au_100Ry_4s1p.orb","Cl_gga_8au_100Ry_2s2p1d.orb"],
    "k_points":[1, 1, 1, 0, 0, 0],
    "user_fp_params":{
    "ntype": 2, 
    "symmetry": 0,
    "vdw_method":"d3_bj",
    "ecutwfc": 100,
    "scf_thr":1e-7,
    "scf_nmax":120, 
    "basis_type":"lcao", 
    "smearing_method": "gauss",
    "smearing_sigma": 0.002,
    "mixing_type": "pulay",
    "mixing_beta": 0.4,
    "cal_force":1,
    "cal_stress":1
```

关键词描述：

| 键词               | 字段类型            | 描述                                                                                                              |
|-------------------|-----------------|---------------------------------------------------------------------|
| "fp_style"        | String          | 第一性原理软件软件。到目前为止，选项包括ABACUS, VASP等。                |
| "shuffle_poscar"  | Boolean         |                                                                     |
| "fp_task_max"     | Integer         | 每次迭代 在 02.fp 中要计算的最大结构。                                 |
| "fp_task_min"     | Integer         | 每次迭代 在 02.fp 中要计算的最小结构。                                 |
| "fp_pp_path"      | String          | 用于 02.fp 的赝势文件路径。                                           |
| "fp_pp_files"     | List of string  | 用于 02.fp 的赝势文件。请注意，元素的顺序应与 type_map 中的顺序相对应。  |
| "fp_orb_files"    | List of string  | 用于 02.fp 的轨道文件。请注意，元素的顺序应与 type_map 中的顺序相对应。  |
| "k_points"        | List of Integer | 用于生成ABACUS KPT文件。                                                    |
| "user_fp_params"  | dict            | 用于生成ABACUS INPUT文件。如果"user_fp_params"中指定了 kspacing，可以不设置"k_points"。   |

填写提示：

标记相关键词指定标记任务的详细信息。 在这里，最少 50 个和最多 200 个结构将使用 ABACUS 代码进行标记，在每次迭代中，INPUT 文件依据“user_fp_params”生成，KPT文件依据 “k_points”生成。请注意，"fp_pp_files" 和 "fp_orb_files" 中元素的顺序应与 `type_map` 中的顺序相对应。

#### machine.json

DP-GEN 运行过程中的每次迭代都由三个步骤组成：探索、标注和训练。因此，machine.json 由三个步骤组成：**train**、**model_devi** 和 **fp**。每个步骤都是字典列表。每个字典都可以被视为一个独立的计算环境。

在本节中，我们将向您展示如何在Bohrium上执行`train`, `model_devi`和`fp` 步骤。 对于每个步骤，需要三种类型的关键词：

- 命令 (Command)：提供用于执行每个步骤的命令。
- 机器 (Machine)：指定机器环境（本地工作站、本地或远程集群或云服务器）。
- 资源 (Resources)：指定组、节点、CPU 和 GPU 的数量;启用虚拟环境。


在此示例中，我们在Bohrium上执行`train`步骤。

```json
{
  "api_version": "1.0",
  "deepmd_version": "2.1.5",
  "train" :[
    {
      "command": "dp",
      "machine": {
        "batch_type": "Lebesgue",
        "context_type": "LebesgueContext",
        "local_root" : "./",
        "remote_profile":{
          "email": "xxx", 
          "password": "xxx", 
          "program_id": xxx,
            "keep_backup":true,
            "input_data":{
                "job_type": "container",
                "log_file": "00*/train.log",
                "grouped":true,
                "job_name": "dpgen_train_job",
                "disk_size": 100,
                "scass_type":"c12_m92_1 * NVIDIA V100",
                "checkpoint_files":["00*/checkpoint","00*/model.ckpt*"],
                "checkpoint_time":5,
                "platform": "ali",
                "image_name":"registry.dp.tech/dptech/deepmd-kit:2.1.5-cuda11.6",
                "on_demand":0
            }
        }
      },
      "resources": {
        "local_root":"./",
        "group_size": 1
      }
    }],
  "model_devi":
    [{
      "command": "export LAMMPS_PLUGIN_PATH=/opt/deepmd-kit-2.1.5/lib/deepmd_lmp && lmp -i input.lammps -v restart 0",
      "machine": {
        "batch_type": "Lebesgue",
        "context_type": "LebesgueContext",
        "local_root" : "./",
        "remote_profile":{
          "email": "xxx", 
          "password": "xxx", 
          "program_id": xxx,
            "keep_backup":true,
            "input_data":{
              "job_type": "container",
              "log_file": "task*/model_devi.out",
              "grouped":true,
              "job_name": "dpgen_model_devi_job",
              "disk_size": 200,
              "scass_type":"c12_m92_1 * NVIDIA V100",
              "platform": "ali",
              "image_name":"registry.dp.tech/dptech/deepmd-kit:2.1.5-cuda11.6",
              "checkpoint_files": "sync_files",
              "checkpoint_time":5,
              "on_demand":0
            }
        }
      },
      "resources": {
        "local_root":"./",
        "group_size": 1
      }
    }],
  "fp":
    [{
      "command": "OMP_NUM_THREADS=1 mpirun -np 4 abacus",
      "machine": {
        "batch_type": "Lebesgue",
        "context_type": "LebesgueContext",
        "local_root" : "./",
        "remote_profile":{
          "email": "xxx", 
          "password": "xxx", 
          "program_id": xxx,
            "keep_backup":true,
            "input_data":{
              "log_file": "task*/output",
              "grouped":true,
              "job_name": "dpgen_fp_job",
              "checkpoint_files": "sync_files",
              "checkpoint_time":5,
              "scass_type":"c8_m64_cpu",
              "platform": "ali",
              "image_name":"registry.dp.tech/dptech/abacus:3.1.0",
              "job_type": "container",
              "on_demand":0
            }
        }
      },
      "resources": {
        "group_size": 2,
        "local_root":"./",
        "source_list": []
      }
    }
  ]
}
```

案例说明：

在 command 参数中，`train`，`model_devi`和`fp`步骤使用的程序分别为 DeePMD-kit，LAMMPS 和 ABACUS，其相应的调用命令分别为“dp”，"export LAMMPS_PLUGIN_PATH=/opt/deepmd-kit-2.1.5/lib/deepmd_lmp && lmp -i input.lammps -v restart 0" 和 "OMP_NUM_THREADS=1 mpirun -np 4 abacus"。

在 machine 参数中，"scass_type"指定训练使用的机型。对于`train`和`model_devi`步骤，建议使用GPU机型，这里使用的机型为“c12_m92_1 * NVIDIA V100”。对于`fp`步骤，建议使用CPU机型，这里使用的机型为“c8_m64_cpu”。

在 resources 参数中，“group_size”指定了一个 group 中任务的数量。对于`train`和`model_devi`步骤，由于任务数量较少（分别为4和20），我们可以将“group_size”设置为 1, 使所有任务同时执行。对于`fp`步骤，由于任务数量较多（200），我们可以将“group_size”设置为 2, 使100个任务同时执行。

注意：用户需要填入自己的 Bohrium 账户邮箱，密码和项目ID（共三处）。其他参数通常不需修改。


### 运行DP-GEN

准备好了 `param.json` 和 `machine.json`，我们就可以通过以下方式轻松运行 DP-GEN：

```sh
dpgen run param.json machine.json
```
成功运行 DP-GEN 之后，在 work/ex4 下，可以发现生成了一个文件夹和两个文件：

- iter.000000 文件夹：包含 DP-GEN 迭代过程中第一次迭代的主要结果；
- dpgen.log 文件：主要记录时间和迭代信息；
- record.dpgen 文件：记录迭代过程的当前阶段。

如果 DP-GEN 的进程由于某种原因停止，DP-GEN 将通过 `record.dpgen`文件自动恢复主进程。我们也可以根据自己的目的手动更改它，例如删除上次迭代并从一个检查点恢复。`record.dpgen`文件每行包含两个数字：第一个是迭代的索引，第二个是 0 到 9 之间的数字，记录每个迭代中的哪个阶段当前正在运行。

| Index of iterations  | Stage in each iteration     | Process          |
|----------------------|-----------------------------|------------------|
| 0                    | 0                           | make_train       |
| 0                    | 1                           | run_train        |
| 0                    | 2                           | post_train       |
| 0                    | 3                           | make_model_devi  |
| 0                    | 4                           | run_model_devi   |
| 0                    | 5                           | post_model_devi  |
| 0                    | 6                           | make_fp          |
| 0                    | 7                           | run_fp           |
| 0                    | 8                           | post_fp          |



### 结果分析

第一次迭代完成后，iter.000000 的文件夹结构如下，主要是 3 个文件夹:
```sh
$ tree iter.000000/ -L 1 
./iter.000000/ 
├── 00.train 
├── 01.model_devi 
└── 02.fp
```

- 00.train 文件夹：主要是基于现有数据训练的多个 DP 模型（默认是 4 个）；
- 01.model_devi 文件夹：使用 00.train 中得到的 DP 模型进行 MD 模拟，产生新的构型；
- 02.fp 文件夹：对选定的构型进行第一原理计算，并将计算结果转换成训练数据



首先，我们来查看 `iter.000000`/ `00.train`。

```sh
$ tree iter.000000/00.train -L 1
./iter.000000/00.train/
|-- 000
|-- 001
|-- 002
|-- 003
|-- data.init -> /data/work/ex3
|-- data.iters
|-- graph.000.pb -> 000/frozen_model.pb
|-- graph.001.pb -> 001/frozen_model.pb
|-- graph.002.pb -> 002/frozen_model.pb
`-- graph.003.pb -> 003/frozen_model.pb
```

- 文件夹 00x 包含 DeePMD-kit 的输入和输出文件，其中训练了模型。
- graph.00x.pb ，链接到 00x/frozen.pb，是 DeePMD-kit 生成的模型。这些模型之间的唯一区别是神经网络初始化的随机种子。

让我们随机选择其中之一查看，例如 000。

```sh
$ tree iter.000000/00.train/000 -L 1
./iter.000000/00.train/000
|-- checkpoint
|-- frozen_model.pb
|-- input.json
|-- lcurve.out
|-- model.ckpt-400000.data-00000-of-00001
|-- model.ckpt-400000.index
|-- model.ckpt-400000.meta
|-- model.ckpt.data-00000-of-00001
|-- model.ckpt.index
|-- model.ckpt.meta
`-- train.log
```

- `input.json` 是当前任务的 DeePMD-kit 的设置。
- `checkpoint`用于重新开始训练。
- `model.ckpt*` 是与模型相关的文件。
- `frozen_model.pb`是冻结模型。
- `lcurve.out`记录能量和力的训练精度。
- `train.log`包括版本、数据、硬件信息、时间等。


然后，我们来查看`iter.000000/ 01.model_devi`。

```sh
$ tree iter.000000/01.model_devi -L 1
./iter.000000/01.model_devi/
|-- confs
|-- graph.000.pb -> /data/work/ex4/iter.000000/00.train/graph.000.pb
|-- graph.001.pb -> /data/work/ex4/iter.000000/00.train/graph.001.pb
|-- graph.002.pb -> /data/work/ex4/iter.000000/00.train/graph.002.pb
|-- graph.003.pb -> /data/work/ex4/iter.000000/00.train/graph.003.pb
|-- task.000.000000
|-- task.000.000001
|-- ...
|-- task.000.000018
`-- task.000.000019
```

- 文件夹 confs 包含从您在 param.json 的“sys_configs”中设置的 STRU 转换而来的 LAMMPS MD 的初始配置。
- 文件夹 task.000.00000x 包含 LAMMPS 的输入和输出文件。我们可以随机选择其中之一，例如 task.000.000000。

```sh
$ tree iter.000000/01.model_devi/task.000.000001
./iter.000000/01.model_devi/task.000.000001
|-- conf.lmp -> ../confs/000.0001.lmp
|-- input.lammps
|-- log.lammps
|-- model_devi.log
|-- model_devi.out
`-- traj
```

- `conf.lmp`，链接到文件夹 confs 中的“000.0001.lmp”，作为 MD 的初始配置。
- `input.lammps` 是 LAMMPS 的输入文件。
- `model_devi.out` 记录在 MD 训练中相关的标签，能量和力的模型偏差。它是选择结构和进行第一性原理计算的标准。
- `traj` 存储LAMMPS MD轨迹。

通过查看 `model_devi.out` 的前几行, 您会看到:

```sh
$ head -n 5 ./iter.000000/01.model_devi/task.000.000001/model_devi.out
#       step         max_devi_v         min_devi_v         avg_devi_v         max_devi_f         min_devi_f         avg_devi_f
           0       1.325778e-02       1.273135e-03       7.165758e-03       1.805290e-02       5.604093e-03       1.014306e-02
          10       1.384183e-02       4.033422e-04       7.237393e-03       1.629300e-02       5.911181e-03       1.090707e-02
          20       1.366198e-02       3.721965e-04       7.527250e-03       2.594799e-02       7.528770e-03       1.298601e-02
          30       1.250935e-02       8.420508e-04       7.166151e-03       2.916632e-02       6.933435e-03       1.464892e-02
```

回想一下，我们将“trj_freq”设置为10，因此每10个步骤保存结构。是否选择结构取决于其`“max_devi_f”`。如果它介于`“model_devi_f_trust_lo”`（0.08）和`“model_devi_f_trust_hi”`（0.18）之间，DP-GEN 会将该结构视为候选结构。对于0，10，20 和 30 步保存结构，其`“max_devi_f”`均小于 0.08，所以不会被视为候选结构。


最后，我们来查看 `iter.000000/ 02.fp` 。

```sh
$ tree iter.000000/02.fp -L 1
./iter.000000/02.fp
|-- data.000
|-- candidate.shuffled.000.out
|-- rest_accurate.shuffled.000.out
|-- rest_failed.shuffled.000.out
|-- task.000.000000
|-- task.000.000001
|-- ...
|-- task.000.000198
`-- task.000.000199
```

- `candidate.shuffle.000.out` 记录将从最后一步 01.model_devi 中选择哪些结构。 候选的数量总是远远超过您一次期望计算的最大值。在这种情况下，DP-GEN将随机选择最多`fp_task_max`结构并形成文件夹任务。
- `rest_accurate.shuffle.000.out` 记录了我们的模型准确的其他结构（`max_devi_f`小于`model_devi_f_trust_lo`），
- `rest_failed.shuffled.000.out` 记录了我们的模型太不准确的其他结构，这些结构通常是非物理的（大于 `model_devi_f_trust_hi`）。
- `data.000`：经过ABACUS SCF计算后，DP-GEN 将收集这些数据并将其更改为 DeePMD-kit 所需的格式。在下一次迭代的“00.train”中，这些数据将与初始数据一起训练。

通过 `cat candidate.shuffled.000.out | grep task.000.000000`， 你将会看到从task.000.000000中收集的候选构型：

```sh
$ cat ./iter.000000/02.fp/candidate.shuffled.000.out | grep task.000.000001
iter.000000/01.model_devi/task.000.000000 52910
iter.000000/01.model_devi/task.000.000000 24370
iter.000000/01.model_devi/task.000.000000 59930
iter.000000/01.model_devi/task.000.000000 9370
iter.000000/01.model_devi/task.000.000000 96490
...
```

在第一次迭代之后，我们检查 dpgen.log 的内容。

```
$ cat dpgen.log
2023-03-20 10:13:30,696 - INFO : start running
2023-03-20 10:13:30,707 - INFO : =============================iter.000000==============================
2023-03-20 10:13:30,707 - INFO : -------------------------iter.000000 task 00--------------------------
2023-03-20 10:13:30,802 - INFO : -------------------------iter.000000 task 01--------------------------
2023-03-20 11:51:16,496 - INFO : -------------------------iter.000000 task 02--------------------------
2023-03-20 11:51:16,514 - INFO : -------------------------iter.000000 task 03--------------------------
2023-03-20 11:51:16,973 - INFO : -------------------------iter.000000 task 04--------------------------
2023-03-20 12:40:25,181 - INFO : -------------------------iter.000000 task 05--------------------------
2023-03-20 12:40:25,185 - INFO : -------------------------iter.000000 task 06--------------------------
2023-03-20 12:40:26,985 - INFO : system 000 candidate :  21098 in 200020  10.55 %
2023-03-20 12:40:26,986 - INFO : system 000 failed    :  19178 in 200020   9.59 %
2023-03-20 12:40:26,986 - INFO : system 000 accurate  : 159744 in 200020  79.86 %
2023-03-20 12:40:27,294 - INFO : system 000 accurate_ratio:   0.7986    thresholds: 1.0000 and 1.0000   eff. task min and max   -1  200   number of fp tasks:    200
2023-03-20 12:41:57,507 - INFO : -------------------------iter.000000 task 07--------------------------
2023-03-20 12:55:55,728 - INFO : -------------------------iter.000000 task 08--------------------------
2023-03-20 12:55:58,276 - INFO : failed tasks:      0 in    200    0.00 % 
```

可以发现，在 iter.000000 中生成了 200020 个结构，其中候选构型21098个，挑选了 200 个结构进行ABACUS SCF计算，且全部成功收敛。


在所有迭代结束之后，我们来检查 `work/ex4` 的文件结构：

```sh
$ tree -L 2
.
|-- abacus
|   |-- Cl_ONCV_PBE-1.2.upf
|   |-- Cl_gga_8au_100Ry_2s2p1d.orb
|   |-- Li_ONCV_PBE-1.2.upf
|   `-- Li_gga_8au_100Ry_4s1p.orb
|-- dpdispatcher.log
|-- dpgen.log
|-- iter.000000
|   |-- 00.train
|   |-- 01.model_devi
|   `-- 02.fp
|-- iter.000001
|   |-- 00.train
|   |-- 01.model_devi
|   `-- 02.fp
|-- iter.000002
|   |-- 00.train
|   |-- 01.model_devi
|   `-- 02.fp
|-- iter.000003
|   `-- 00.train
|-- machine.json
|-- param_abacus.json
`-- record.dpgen
```

以及 `dpgen.log` 的内容：

```
$ cat cat dpgen.log | grep system
2023-03-20 12:40:26,985 - INFO : system 000 candidate :  21098 in 200020  10.55 %
2023-03-20 12:40:26,986 - INFO : system 000 failed    :  19178 in 200020   9.59 %
2023-03-20 12:40:26,986 - INFO : system 000 accurate  : 159744 in 200020  79.86 %
2023-03-20 12:40:27,294 - INFO : system 000 accurate_ratio:   0.7986    thresholds: 1.0000 and 1.0000   eff. task min and max   -1  200   number of fp tasks:    200
2023-03-20 15:24:24,243 - INFO : system 000 candidate :    299 in 200020   0.15 %
2023-03-20 15:24:24,243 - INFO : system 000 failed    :     64 in 200020   0.03 %
2023-03-20 15:24:24,243 - INFO : system 000 accurate  : 199657 in 200020  99.82 %
2023-03-20 15:24:24,536 - INFO : system 000 accurate_ratio:   0.9982    thresholds: 1.0000 and 1.0000   eff. task min and max   -1  200   number of fp tasks:    200
2023-03-20 20:22:25,403 - INFO : system 000 candidate :      9 in 200020   0.00 %
2023-03-20 20:22:25,403 - INFO : system 000 failed    :      1 in 200020   0.00 %
2023-03-20 20:22:25,404 - INFO : system 000 accurate  : 200010 in 200020 100.00 %
2023-03-20 20:22:25,699 - INFO : system 000 accurate_ratio:   1.0000    thresholds: 1.0000 and 1.0000   eff. task min and max   -1  200   number of fp tasks:      9
```

可以发现，仅在三个iteration后accurate_ratio接近100.00 %。在 'iter.000002' 中收集了 9 个候选结构，小于param.json中`fp_task_min`关键词指定的50。因此，模型不会在 iter.000003/00.train 中更新。

为了更直观监测迭代过程，我们提供了提供了一个名为max_devi_f.py的python脚本，用于统计每个iteration中所有构型max_devi_f的分布
```python
import os
import numpy as np
import matplotlib.pyplot as plt

for i in range(0,3):       # 指定迭代的轮数
    max_devi_f_values = []
    for j in range(20):    # 在iter*/01.model_devi中有20个LAMMPS任务
        directory = "./iter.{:06d}/01.model_devi/task.000.{:06d}".format(i, j)
        file_path = os.path.join(directory, "model_devi.out")
        data = np.genfromtxt(file_path, skip_header=1, usecols=4)
        max_devi_f_values.append(data)

    # 把列表转换成numpy数组
    max_devi_f_values = np.concatenate(max_devi_f_values)

    # 使用numpy.histogram()计算max_devi_f在每个统计区间的频率
    hist, bin_edges = np.histogram(max_devi_f_values, range=(0, 0.2), bins=100)

    # 把频率转换成百分数
    hist = hist / len(max_devi_f_values) * 100

    # Use matplotlib.pyplot.plot() to plot the frequency of each value
    plt.plot(bin_edges[:-1], hist,label = 'iter{:02d}'.format(i))
    plt.legend()
    plt.xlabel("max_devi_f eV/Å")
    plt.ylabel("Distribution %")

    with open(f'./iter{i:02d}_dist-max-devi-f.txt'.format(i), 'w') as f:
        f.write("{:>12} {:>12}\n".format("bin_edges", "hist"))
        for x, y in zip(bin_edges[:-1], hist):
            f.write('{:>12.3f} {:>12.3f}\n'.format(x, y))

plt.savefig('max-devi-f.png',dpi=300)
```

![](./dpgen/max-devi-f.png)

至此，我们应该已经了解了如何使用 DP-GEN 生成和训练深度势模型，以及如何分析输出结果。
