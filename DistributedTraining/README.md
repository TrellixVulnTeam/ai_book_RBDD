# Distributed Training

## Data parallelism

Data parallelism is when you use the same model for every thread, but feed it with different parts of the data. Basically, when you train your model with data parallelism with multiple workers, what you do is you copy the same models to all workers, and split the training data into N subsets, where N is the number of the workers. Then, you will assign each subset of training dataset to corresponding worker. The forward propagation method works same with single machine training, however, when you run the back propagation for data parallelism, you should make all workers to share the loss values that they calculated with all other workers, so that all models could learn from entire dataset.

## Model parallelism

Model parallelism is when you use the same data for every thread, but split the model among threads.

![Model parallelism](./img/model_parallelism.png)

For model parallelism, there are multiple strategies that we could use - 1) layer splitting, 2) training feature splitting, and 3) Mixed strategy.

For layer splitting strategy, we could simply split layers in the model to multiple subsets, and assign each subset of layers to individual worker (server).

![layer splitting strategy](./img/layer_split_model_parallel.png)

For feature splitting strategy, we run multiple servers concurrently. Each worker will calculate the results of affine function of each hidden node, and each worker must communicate with all other workers whenever it propagates the result of hidden nodes to the next hidden layer.

![feature splitting strategy](./img/feature_split_model_parallel.png)

Mixed strategy combines the strategies of 2 different distributed training methods as below.

![Mixed strategy](./img/mixed_model_parallel.png)

## PyTorch

- [Distributed Data-Parallel Training (DDP)](https://pytorch.org/docs/stable/generated/torch.nn.parallel.DistributedDataParallel.html) is a widely adopted single-program multiple-data training paradigm. With DDP, the model is replicated on every process, and every model replica will be fed with a different set of input data samples. DDP takes care of gradient communication to keep model replicas synchronized and overlaps it with the gradient computations to speed up training.

- [RPC-Based Distributed Training (RPC)](https://pytorch.org/docs/stable/rpc.html) supports general training structures that cannot fit into data-parallel training such as distributed pipeline parallelism, parameter server paradigm, and combinations of DDP with other training paradigms. It helps manage remote object lifetime and extends the autograd engine beyond machine boundaries.

- [Collective Communication (c10d)](https://pytorch.org/docs/stable/distributed.html) library supports sending tensors across processes within a group. It offers both collective communication APIs (e.g., all_reduce and all_gather) and P2P communication APIs (e.g., send and isend). DDP and RPC (ProcessGroup Backend) are built on c10d, where the former uses collective communications and the latter uses P2P communications. Usually, developers do not need to directly use this raw communication API, as the DDP and RPC APIs can serve many distributed training scenarios. However, there are use cases where this API is still helpful. One example would be distributed parameter averaging, where applications would like to compute the average values of all model parameters after the backward pass instead of using DDP to communicate gradients. This can decouple communications from computations and allow finer-grain control over what to communicate, but on the other hand, it also gives up the performance optimizations offered by DDP. The Writing Distributed Applications with PyTorch shows examples of using c10d communication APIs.

[model parallel tutorial notebook](./src/model_parallel_tutorial.ipynb)

[model parallel tutorial notebook - korean version](./src/model_parallel_tutorial_kor.ipynb)

## Fairscale

[Fairscale](https://github.com/facebookresearch/fairscale) is a PyTorch extensions for high performance and large scale training.

## Deepspeed

[DeepSpeed](https://github.com/microsoft/DeepSpeed) is a deep learning optimization library that makes distributed training and inference easy, efficient, and effective.

It implements everything that are described in [ZeRO paper [1]](https://arxiv.org/abs/1910.02054).

### Integrate Huggingface with Deepspeed

[This page](https://huggingface.co/docs/transformers/main_classes/deepspeed?highlight=deepspeed#deepspeed-integration) contains the descriptions and codes for integrating the Deepspeed with Huggingface for distributed training with Huggingface models.

### Deepspeed with ZeRO

[Github repor for example codes of using Deepspeed for training large models](https://github.com/microsoft/DeepSpeedExamples)

## References

[1] Samyam Rajbhandari, Jeff Rasley, Olatunji Ruwase, Yuxiong He. [ZeRO: Memory Optimizations Toward Training Trillion Parameter Models](https://arxiv.org/abs/1910.02054)
