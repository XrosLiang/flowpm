# flowpm [![Build Status](https://travis-ci.org/modichirag/flowpm.svg?branch=master)](https://travis-ci.org/modichirag/flowpm)[![PyPI version](https://badge.fury.io/py/flowpm.svg)](https://badge.fury.io/py/flowpm)
Particle Mesh Simulation in TensorFlow, based on [fastpm-python](https://github.com/rainwoodman/fastpm-python) simulations

To install:
```
$ pip install flowpm
```

For a minimal working example of FlowPM, see this [notebook](notebook/flowpm_tutorial.ipynb). The steps are as follows:
```python
import tensorflow as tf
import flowpm

stages = np.linspace(0.1, 1.0, 10, endpoint=True)

initial_conditions = flowpm.linear_field(32,          # size of the cube
                                         100,         # Physical size of the cube
                                         ipklin,      # Initial power spectrum
                                         batch_size=16)

# Sample particles
state = flowpm.lpt_init(initial_conditions, a0=0.1)   

# Evolve particles down to z=0
final_state = flowpm.nbody(state, stages, 32)         

# Retrieve final density field
final_field = flowpm.cic_paint(tf.zeros_like(initial_conditions), final_state[0])

with tf.Session() as sess:
    sim = sess.run(final_field)
```

## Mesh TensorFlow implementation

FlowPM provides a Mesh TensorFlow implementation of FastPM, for running distributed
simulations across very large supercomputers.

Here are the instructions for installing and running on Cori-GPU. More info about
this machine here: https://docs-dev.nersc.gov/cgpu/

0) Login to a cori-gpu node to prepare the environment:
```
$ module add esslurm
$ salloc -C gpu -N 1 -t 30 -c 10 --gres=gpu:1 -A m1759
```

1) First install dependencies
```
$ module purge && module load tensorflow/gpu-1.15.0-rc1-py37 esslurm gcc/7.3.0 cuda
$ pip install --user mesh-tensorflow
```

3) Install the Mesh TensorFlow branch of FlowPM
```
$ git clone https://github.com/modichirag/flowpm.git
$ cd flowpm
$ git checkout mesh
$ pip install --user -e .
```

4) To run the demo comparing the distributed computation to single GPU:
```
$ cd examples
$ sbatch lpt_job.sh
```

This will generate a plot `comparison.png` showing from a set of initial
conditions, the result of a single LPT step on single GPU TensorFlow vs Mesh
TensorFlow.
