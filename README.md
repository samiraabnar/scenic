# Scenic
<div style="text-align: left">
<img align="right" src="https://raw.githubusercontent.com/google-research/scenic/main/images/scenic_logo.png" width="200" alt="scenic logo"></img>
</div>
*Scenic* is a codebase with a focus on research around attention-based models
for computer vision. Scenic has been successfully used to develop
classification, segmentation, and detection models for multiple modalities
including images, video, audio, and multimodal combinations of them.

More precisely, *Scenic* is a (i) set of shared light-weight libraries solving
tasks commonly encountered tasks when training large-scale (i.e. multi-device,
multi-host) vision models; and (ii) a number of *projects* containing fully
fleshed out problem-specific training and evaluation loops using these
libraries.

Scenic is developed in [JAX](https://github.com/google/jax) and uses
[Flax](https://github.com/google/flax).

### Contents
* [What we offer](#what-we-offer)
* [SOTA models and baselines in Scenic](#sota-models-and-baselines-in-scenic)
* [Philosophy](#philosophy)
* [Getting started](#getting-started)
* [Scenic component design](#scenic-component-design)
* [Citing Scenic](#citing-scenic)

## What we offer
Among others *Scenic* provides

* Boilerplate code for launching experiments, summary writing, logging,
  profiling, etc;
* Optimized training and evaluation loops, losses, metrics, bi-partite matchers,
  etc;
* Input-pipelines for popular vision datasets;
* [Baseline models](https://github.com/google-research/scenic/tree/main/scenic/projects/baselines#scenic-baseline-models),
including strong non-attentional baselines.


## SOTA models and baselines in *Scenic*
There some SOTA models and baselines in Scenic which were either developed
using Scenic, or have been reimplemented in Scenic:

Projects that were developed in Scenic or used it for their experiments:
* [ViViT: A Video Vision Transformer](https://arxiv.org/abs/2103.15691)
* [OmniNet: Omnidirectional Representations from Transformers](https://arxiv.org/abs/2103.01075)
* [Attention Bottlenecks for Multimodal Fusion](https://arxiv.org/abs/2107.00135)
* [TokenLearner: What Can 8 Learned Tokens Do for Images and Videos?](https://arxiv.org/abs/2106.11297)
* [Exploring the Limits of Large Scale Pre-training](https://arxiv.org/abs/2110.02095)
* [The Efficiency Misnomer](https://arxiv.org/abs/2110.12894)

More information can be found in [projects](https://github.com/google-research/scenic/tree/main/scenic/projects#list-of-projects-hosted-in-scenic).

Baselines that were reproduced in Scenic:
* [An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale](https://arxiv.org/abs/2010.11929)
* [End-to-End Object Detection with Transformers](https://arxiv.org/abs/2005.12872)
* [MLP-Mixer: An all-MLP Architecture for Vision](https://arxiv.org/abs/2105.01601)
* [Deep Residual Learning for Image Recognition](https://arxiv.org/abs/1512.03385)
* [U-Net: Convolutional Networks for Biomedical Image Segmentation](https://arxiv.org/abs/1505.04597)

More information can be found in [baseline models](https://github.com/google-research/scenic/tree/main/scenic/projects/baselines#scenic-baseline-models).

<a name="philosophy"></a>
## Philosophy
*Scenic* aims to facilitate rapid prototyping of large-scale vision models. To
keep the code simple to understand and extend we prefer *forking and
copy-pasting over adding complexity or increasing abstraction*. Only when
functionality proves to be widely useful across many models and tasks it may be
upstreamed to Scenic's shared libraries.


<a name="getting_start"></a>
## Getting started
* See `projects/baselines/README.md` for a walk-through baseline models and
  instructions on how to run the code.
* If you would like to to contribute to *Scenic*, please check out the
  [Philisophy](#philosophy), [Code structure](#code_structure) and
  [Contributing](CONTRIBUTING.md) sections.
  Should your contribution be a part of the shared libraries, please send us a
  pull request!


### Quick start
Download the code from GitHub

```
git clone https://github.com/google-research/scenic.git
cd scenic
pip install .
```

and run training for ViT on ImageNet:

```
python scenic/main.py -- \
  --config=scenic/projects/baselines/configs/imagenet/imagenet_vit_config.py \
  --workdir=./
```

[Here](https://colab.research.google.com/github/google-research/scenic/blob/main/scenic/common_lib/colabs/scenic_playground.ipynb)
is also a minimal colab to train a simple feed forward model using Scenic.

<a name="code_structure"></a>
## Scenic component design
Scenic is designed to propose different levels of abstraction, to support
hosting projects that only require changing hyper-parameters by defining config
files, to those that need customization on the input pipeline, model
architecture, losses and metrics, and the training loop. To make this happen,
the code in Scenic is organized as either _project-level_ code,
which refers to customized code for specific projects or baselines or
_library-level_ code, which refers to common functionalities and general
patterns that are adapted by the majority of projects. The project-level
code lives in the `projects` directory.

<div align="center">
<img src="https://raw.githubusercontent.com/google-research/scenic/main/images/scenic_design.jpg" width="900" alt="scenic design"></img>
</div>

### Library-level code
The goal is to keep the library-level code minimal and well-tested and to avoid
introducing extra abstractions to support minor use-cases. Shared libraries
provided by *Scenic* are split into:

*   `dataset_lib`: Implements IO pipelines for loading and pre-processing data
    for common Computer Vision tasks and benchmarks (see "Tasks and Datasets"
    section). All pipelines are designed to be scalable and support multi-host
    and multi-device setups, taking care dividing data among multiple hosts,
    incomplete batches, caching, pre-fetching, etc.
*   `model_lib` : Provides
    *   several abstract model interfaces (e.g. `ClassificationModel` or
        `SegmentationModel` in `model_lib.base_models`) with task-specific
        losses and metrics;
    *   neural network layers in `model_lib.layers`, focusing on efficient
        implementation of attention and transformer layers;
    *   accelerator-friendly implementations of bipartite matching
        algorithms in `model_lib.matchers`.
*   `train_lib`: Provides tools for constructing training loops and implements
    several optimized trainers (classification trainer and segmentation trainer)
    that can be forked for customization.
*   `common_lib`: General utilities, like logging and debugging modules,
    functionalities for processing raw data, etc.

### Project-level code
Scenic supports the development of customized solutions for customized tasks and
data via the concept of "project". There is no one-fits-all recipe for how much
code should be re-used by a project. Projects can consist of only configs and
use the common models, trainers, task/data that live in library-level code, or
they can simply fork any of the mentioned functionalities and redefine, layers,
losses, metrics, logging methods, tasks, architectures, as well as training and
evaluation loops. The modularity of library-level code makes it flexible for
projects to fall placed on any spot in the "run-as-is" to "fully-customized"
spectrum.

Common baselines such as a ResNet and Vision Transformer (ViT) are implemented
in the [`projects/baselines`](https://github.com/google-research/scenic/tree/main/scenic/projects/baselines)
project. Forking models in this directory is a good starting point for new
projects.


## Citing Scenic
If you use Scenic, you can cite our [white paper](https://arxiv.org/abs/2110.11403).
Here is an example BibTeX entry:
```
@article{dehghani2021scenic,
  author={Mostafa Dehghani and Alexey Gritsenko and Anurag Arnab and Matthias Minderer and Yi Tay},
  title={{Scenic}: A {JAX} Library for Computer Vision Research and Beyond},
  year={2021},
  journal={arXiv preprint arXiv:2110.11403},
}
```

_Disclaimer: This is not an official Google product._
