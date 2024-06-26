# CompNav
CompNav: Training a Perception-Comparison-Action Policy for Image-Goal Navigation
## Abstract
Recent works in image-goal navigation typically involve learning a perception-action navigation policy. They first capture the semantic features of the goal image and the egocentric image independently and then pass them to the policy network for action prediction. Despite a remarkable series of efforts on improving visual representations, some challenges still exist for these methods: (1) Semantic feature vectors inadequately convey valid direction information for navigation, resulting in inefficient and superfluous actions; (2) Performance decreases dramatically when encountering viewpoint inconsistencies caused by differences in camera settings between training and application.
In this paper, we endeavor to overcome these challenges by constantly comparing the goal image with current observations during the navigation process, drawing inspiration from human-intuitive navigation. Specifically, we design a correlation-based model that explicitly compares the goal image and the current observation in the perception step and passes the comparison information to the policy network, \emph{i.e.}, training a perception-comparison-action navigation policy~(CompNav). We gradually enhance the comparison modeling by using more fine-grained cross-correlation and introducing direction-aware correlation to provide precise direction information for navigation. Extensive evaluation of the CompNav on 3 benchmark datasets~(Gibson, HM3D, and MP3D) shows superior performance in terms of navigation efficiency~(SPL). In addition, CompNav significantly outperforms previous state-of-the-art methods across all metrics~(SPL, SR) under the “user-matched goal” setting, showing potential for real-world applications. 

### Installing on the host machine
```bash
# create conda env
conda create -n CompNav python=3.8
conda activate CompNav
conda install habitat-sim=0.2.2 withbullet headless -c conda-forge -c aihabitat
pip install torch==1.11.0+cu113 torchvision==0.12.0+cu113 -f https://download.pytorch.org/whl/torch_stable.html

git submodule init
git submodule update
cd habitat-lab
git checkout 1f7cfbdd3debc825f1f2fd4b9e1a8d6d4bc9bfc7
pip install -e habitat-lab 
pip install -e habitat-baselines
cd ..
pip install requirements.txt
```

## Data preparation
### Datasets
Follow the [official guidance](https://github.com/facebookresearch/habitat-sim/blob/main/DATASETS.md#gibson-and-3dscenegraph-datasets) to download `Gibson`, `HM3D`, and `MP3D` scene datas. And follow FGPrompt download the dataset.zip. 
```
data/datasets/
└── imagenav
    ├── gibson
    ├── hm3d
    └── mp3d
data/scene_datasets
    ├── gibson
    ├── hm3d
    └── mp3d
```

## Training
```bash
MAGNUM_LOG=quiet HABITAT_SIM_LOG=quiet python -m torch.distributed.launch 
--nproc_per_node=4 
--master_port=11200 
--nnodes=1 
--node_rank=0 
--master_addr=127.0.0.1 
run.py 
--overwrite 
--exp-config 
exp_config/ddppo_imagenav_gibson.yaml,policy,reward,dataset,sensors,CompNav 
--run-type 
train 
--model-dir 
results/imagenav/exp1
```

## Evaluation: Reproduce the Results in the main paper️
### Evaluation on Gibson

```bash
# agent-matched setting
MAGNUM_LOG=quiet HABITAT_SIM_LOG=quiet python -m torch.distributed.launch 
--nproc_per_node=1 
--master_port=11200 
--nnodes=1 
--node_rank=0 
--master_addr=127.0.0.1 
run.py 
--exp-config
exp_config/ddppo_imagenav_gibson.yaml,policy,reward,dataset,sensors,CompNav,eval
--run-type
eval
--model-dir
results/imagenav/exp2
habitat_baselines.eval_ckpt_path_dir
/checkpoint/checkpoint.pth

# user-matched setting
MAGNUM_LOG=quiet HABITAT_SIM_LOG=quiet python -m torch.distributed.launch 
--nproc_per_node=1 
--master_port=11200 
--nnodes=1 
--node_rank=0 
--master_addr=127.0.0.1 
run.py 
--exp-config
exp_config/ddppo_imagenav_gibson.yaml,policy,reward,dataset,sensors,CompNav,eval
--run-type
eval
--model-dir
results/imagenav/exp2
habitat_baselines.eval_ckpt_path_dir
/checkpoint/checkpoint.pth
--habitat.task.imagegoal_sensor_v2.augmentation.activate
True
```




### Cross-domain Evaluation on HM3D
```bash
# agent-matched setting
MAGNUM_LOG=quiet HABITAT_SIM_LOG=quiet python -m torch.distributed.launch 
--nproc_per_node=1 
--master_port=11200 
--nnodes=1 
--node_rank=0 
--master_addr=127.0.0.1 
run.py 
--exp-config 
exp_config/ddppo_imagenav_gibson.yaml,policy,reward,dataset-hm3d,sensors,CompNav,eval 
--run-type 
eval 
--model-dir 
results/imagenav/exp2
habitat_baselines.eval_ckpt_path_dir 
/checkpoint/checkpoint.pth
habitat_baselines.eval.split 
val_easy # [val_easy, val_hard, val_medium]

# user-matched setting
MAGNUM_LOG=quiet HABITAT_SIM_LOG=quiet python -m torch.distributed.launch 
--nproc_per_node=1 
--master_port=11200 
--nnodes=1 
--node_rank=0 
--master_addr=127.0.0.1 
run.py 
--exp-config 
exp_config/ddppo_imagenav_gibson.yaml,policy,reward,dataset-hm3d,sensors,CompNav,eval 
--run-type 
eval 
--model-dir 
results/imagenav/exp2
habitat_baselines.eval_ckpt_path_dir 
/checkpoint/checkpoint.pth
habitat_baselines.eval.split 
val_easy # [val_easy, val_hard, val_medium]
--habitat.task.imagegoal_sensor_v2.augmentation.activate
True
```

### Cross-domain Evaluation on MP3D
```bash
# agent-matched setting
MAGNUM_LOG=quiet HABITAT_SIM_LOG=quiet python -m torch.distributed.launch 
--nproc_per_node=1 
--master_port=11200 
--nnodes=1 
--node_rank=0 
--master_addr=127.0.0.1 
run.py 
--exp-config 
exp_config/ddppo_imagenav_gibson.yaml,policy,reward,dataset-mp3d,sensors,CompNav,eval
--run-type 
eval 
--model-dir 
results/imagenav/exp2
habitat_baselines.eval_ckpt_path_dir 
/checkpoint/checkpoint.pth
habitat_baselines.eval.split 
test_easy # [test_easy, test_hard, test_medium]

# user-matched setting
MAGNUM_LOG=quiet HABITAT_SIM_LOG=quiet python -m torch.distributed.launch 
--nproc_per_node=1 
--master_port=11200 
--nnodes=1 
--node_rank=0 
--master_addr=127.0.0.1 
run.py 
--exp-config 
exp_config/ddppo_imagenav_gibson.yaml,policy,reward,dataset-mp3d,sensors,CompNav,eval
--run-type 
eval 
--model-dir 
results/imagenav/exp2
habitat_baselines.eval_ckpt_path_dir 
/checkpoint/checkpoint.pth
habitat_baselines.eval.split 
test_easy # [test_easy, test_hard, test_medium]
--habitat.task.imagegoal_sensor_v2.augmentation.activate
True
```
