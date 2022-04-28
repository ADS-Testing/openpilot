## Build from Source

### Requirements
* OS: Ubuntu 20.04
* CPU: at least 6 cores
* GPU: at least 6GB memory
* Openpilot 0.8.5 (customized version)
* Carla 0.9.11
* FusED
* If using GPU for OP: CUDA 11.x and CUDNN for 11.x

### Directory Structure
~(home folder)
├── openpilot
├── Documents
│   ├── self-driving-cars (created by the user manually)
│   │   ├── FusED
│   │   ├── carla_0911_rss

Note: one can create link for these folders at these paths if one cannot put them in these paths.

### Install OpenPilot 0.8.5 (customized version)
In `~`,
```
git clone https://github.com/AIasd/openpilot.git
```
In `~/openpilot`,
```
./tools/ubuntu_setup.sh
```
In `~/openpilot`, compile Openpilot
```
scons -j$(nproc)
```

#### Common Python Path Issue
Make sure the python path is set up correctly through pyenv, in particular, run
```
which python
```
One should see the following:
```
~/.pyenv/shims/python
```
Otherwise, one needs to follow the instruction by displayed after running
```
pyenv init
```

#### Common Compilation Issue
clang 10 is needed. To install it, run
```
sudo apt install clang
```

#### Common OpenCL Issue
Your enviroment needs to support opencl2.0+ in order to run `scons` successfully (when using `clinfo`, it must show something like  "your OpenCL library only supports OpenCL <2.0+>")

#### Run Supercombo model on GPU
CUDA 11.x and CUDNN for 11.x must be installed


### Install Carla 0.9.11
In `~/Documents/self-driving-cars`,
```
curl -O https://carla-releases.s3.eu-west-3.amazonaws.com/Linux/CARLA_0.9.11_RSS.tar.gz
mkdir carla_0911_rss
tar -xvzf CARLA_0.9.11_RSS.tar.gz -C carla_0911_rss
```

In `~/Documents/self-driving-cars/carla_0911_rss/PythonAPI/carla/dist`,
```
easy_install carla-0.9.11-py3.7-linux-x86_64.egg
```

#### Install additional maps (optional)
In `~/Documents/self-driving-cars/carla_0911_rss`,
```
curl -O https://carla-releases.s3.eu-west-3.amazonaws.com/Linux/AdditionalMaps_0.9.11.tar.gz
mv AdditionalMaps_0.9.11.tar.gz Import/
./ImportAssets.sh
```

### Installation for AutoFuzz
In `~/Docuements/self-driving-cars`,
```
git clone https://github.com/AIasd/FusED.git --recursive
```

In `~/openpilot/tools/sim/op_script`,
```
pip3 install -r requirements.txt
```

Install pytorch
```
pip3 install torch==1.9.0+cu111 torchvision==0.10.0+cu111 torchaudio==0.9.0 -f https://download.pytorch.org/whl/torch_stable.html
```


<!-- ### Try OpenPilot with an example
In `~/openpilot/tools/sim/op_script`,
```
./bridge_multiple_sync3.py
``` -->

### Run Fuzzing
In `~/Docuements/self-driving-cars/FusED`,
```
python ga_fuzzing.py --simulator carla_op --n_gen 2 --pop_size 2 --algorithm_name nsga2 --has_run_num 4 --episode_max_time 200 --only_run_unique_cases 0 --objective_weights 1 0 0 0 -1 -2 0 -m op --route_type 'Town06_Opt_forward'
```

### Rerun previous simulations
In `~/openpilot/tools/sim/op_script`,
```
python rerun_carla_op.py -p <path-to-the-parent-folder-consisting-of-single-simulation-runs-indexed-by-numbers>
```

### Rerun scenarios using the best sensor prediction
Move all the subfolders indexed of previously run simulation results in `~/Docuements/self-driving-cars/FusED/run_results_op` to `openpilot/tools/sim/op_script/rerun_folder`, then in `openpilot/tools/sim/op_script`,
```
python rerun_carla_op.py -p rerun_folder -m2 best_sensor -w 2.5
```
