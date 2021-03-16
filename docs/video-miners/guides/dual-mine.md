# Dual Mine

- [Dual Mine](#dual-mine)
  - [Pre-requisites](#pre-requisites)
  - [Dual ethash mining and transcoding](#dual-ethash-mining-and-transcoding)
    - [Choosing a miner](#choosing-a-miner)
    - [Run the miner](#run-the-miner)
    - [Run livepeer](#run-livepeer)
    - [Switching](#switching)

## Pre-requisites

- Make sure you have [activated your orchestrator](../getting-started/activate.md)

## Dual ethash mining and transcoding

### Choosing a miner

Dual ethash mining and transcoding has been tested on the GPUs in [this list](https://github.com/livepeer/wiki/blob/master/GPU-SUPPORT.md) and with the following miners:

- [t-rex](https://github.com/trexminer/T-Rex)
- [ethminer](https://github.com/ethereum-mining/ethminer)

If you successfully test with other miners, contributions to this document are welcome.

If you are using a Nvidia GPU from the Volta generation or later, you can use [CUDA MPS](https://docs.nvidia.com/deploy/mps/index.html) to try to improve parallelization of the ethash mining and transcoding workloads. Refer to the [GPU NVENC/NVDEC support matrix](https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new) for the generation that a GPU is from.

If you are using a post-Volta GPU, the recommendation is to use ethminer because it exposes flags for more granular adjustments to the GPU workload which will be needed when using CUDA MPS to prevent the miner from fully staturating the GPU. Other miners that support similar flags can be substituted for ethminer as well.

If you are using a pre-Volta GPU, the recommendation is to use t-rex because it is, at the time of writing, the most popular and efficient ethash miner that has been tested with `livepeer`. Other miners that have been tested successfully with `livepeer` can be substituted for t-rex as well.

Note that regardless of the miner used, the VRAM available on your GPU will affect the number of concurrent streams that can be transcoded while mining.

### Run the miner

The following instructions will assume you are using either t-rex or ethminer. If you are using a different miner, the miner commands should be updated to reflect the requirements of the miner being used.

If you are using a post-Volta GPU:

1. Enable CUDA by following [these instructions](https://docs.nvidia.com/deploy/mps/index.html#topic_6_1_2)

2. Run ethminer with flags to adjust the GPU workload (other flags to connect to a mining pool omitted):

    ```bash
    ethminer \
      -U \
      --cu-devices <GPU_DEVICE_IDS> \ # List of Nvidia GPU IDs
      --cuda-streams <CUDA_STREAMS> \ # Number of multiprocessor streams
      --cuda-block-size <CUDA_BLOCK_SIZE> \ # Number of threads per block
      --cuda-grid-size <CUDA_GRID_SIZE> \ # Number of blocks per grid
    ```

The `--cuda-streams`, `--cuda-block-size` and `--cuda-grid-size` flags are used to adjust the GPU workload. The best values to use for these flags will depend on your GPU and whether you want lower hashrate and faster transcoding speed or higher hashrate and lower transcoding speed.

If you are using a pre-Volta GPU:

1. Run t-rex (other flags to connect to a mining pool omitted):

    ```bash
    t-rex \
      -a ethash \
      -d <GPU_DEVICE_IDS> \ # List of Nvidia GPU IDs
    ```

### Run livepeer

Run `livepeer` on the same machine as the miner and using the same GPU device IDs as the miner with the `-nvidia` flag.

### Switching

Instead of always concurrently ethash mining and transcoding, it could also be possible to switch between the two to maximize mining efficiency when there is low transcoding demand and maximize transcoding capacity when there is high transcoding demand.

TBD