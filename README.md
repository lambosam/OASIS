# Project Codebase for Course XXX

This repository contains the codebase for implementing and benchmarking the [Oasis range filter](https://www.vldb.org/pvldb/vol17/p1911-luo.pdf), a cutting-edge learned range filter that significantly enhances performance by explicitly pruning large empty intervals in the key space and optimally allocating bitmap lengths to the remaining intervals based on theoretical analysis.

We provide a clear and simple implementation of Oasis, along with a comprehensive benchmarking suite to facilitate convinient performance testing. Additionally, the codebase is designed to be extendable, allowing you to modify and adapt the filter to your specific needs. The following instructions will guide you through preparing datasets and running experiments.

## Directory Layout

- `oasis/` - contains all the files necessary to the implementation of the Oasis filter.

- `benchmark/` - contains all the files used for standalone filter benchmarks.
  - `test_data/` - contains the synthetic workloads that are used for evaluating the performance of range filters.`


## Implementation
### Oasis range filter
The Oasis range filter is implemented as a class in `/oasis/oasis.h` and provides several APIs, including:
- **query**: returns whether a specific query (either a point or range query) exists after initializing an Oasis instance;
- **size**: return the size of oasis filter.

```
# initialize and utilize oasis
double bpk = 8;
uint16_t block_sz = 100;
std::vector<uint64_t> keys = {100, 150, 300};
oasis::Oasis<uint64_t> *filter = new oasis::Oasis<uint64_t>(bpk, block_sz, keys);
std::cout << "The query result is : " \
    << filter->query(200, 210) \
    << " with filter size : " \
    << filter->size() << std::endl;
```

### Standalone Filter Benchmarks
To simplify the experimental process, we provide a convenient platform in `/benchmark/in_mem_bench.cc` that accepts various parameters to instantiate specific experiments. Additionally, the shell script `/benchmark/in_mem_bench.sh` can be used to set up and run your experiments. Below are some key parameters that you can adjust in the script:

- kdist_arr - key distribution including "kuniform" and "knormal"
- qdist_arr - query distribution including "quniform"
- minrange_arr - the shortest range query in the workload, which is set to 2
- maxrange_arr - the longest range query in the workload which should be chosen from {2, 16, 256}
- membudg_arr - expected size of range filter quantified in bits per key

Please refer to `in_mem_bench.sh` for more detailed information. It is important **not to alter the number of keys and queries** to maintain consistency with the provided dataset.

For extendability, the test implementation is abstracted as a `TestWrapper` class defined in `/test_wrapper.hpp`. Then, `OasisWrapper` is the derived class of `TestWrapper` which incorporate the OASIS as its range filter. You can easily implement your own range filter by adding another derived class in this folder.


## Workload
We provide six workloads to evaluate the performance of range filter, located in the `/benchmark/test_data` directory. 
Each workload consists of 1 million keys (stored in `data0.txt`) and 100,000 queries (with lower and upper bounds stored in `txn0.txt` and `upper_bound0.txt`, respectively). These workloads exhibit different characteristics, encoded in their respective folder paths, which specify the number of keys, queries, minimum and maximum interval length, key distribution, and query distribution as described in `/benchmark/in_mem_bench.sh`.


| workload | #key | #queries | key distribution | query distribution | min interval | max interval |
|:--------:|:----:|:--------:|:----------------:|:------------------:|:-------------------:|:-------------------:|
|     A    |  1M  |   0.1M   |      uniform     |       uniform      |          2          |          2          |
|     B    |  1M  |   0.1M   |      normal      |       uniform      |          2          |          2          |
|     C    |  1M  |   0.1M   |      uniform     |       uniform      |          2          |          16         |
|     D    |  1M  |   0.1M   |      normal      |       uniform      |          2          |          16         |
|     E    |  1M  |   0.1M   |      uniform     |       uniform      |          2          |         256         |
|     F    |  1M  |   0.1M   |      normal      |       uniform      |          2          |         256         |

## Dependency

To run the code, `CMake`, `gcc9`, and `g++9` are required. These can be installed using the following command:

```
# Linux
sudo apt-get install cmake
```


## Running the Benchmark
### Excute the shell script.

```
#create new folder to complie the project
mkdir build && cd build

#complie the project
cmake -DCMAKE_BUILD_TYPE=Release ..
make all

#enter the folder containing the script
cd ../benchmark

#run the script and record the output
bash ./in_mem_bench.sh test > test.txt
```

### Experimental results
The experimental results are stored in `./benchmark/test_result/test` after excuting the script with the following structure.
- `Oasis` contains more readable and detailed information for each experiment, including the working folder, experimental settings, and results (i.e., filter construction time, query time, and false positive rate)
- `index.txt` concluds the experimental setting of all experiments
- `results.csv` summerizes the experimental settings and results for all experiments (i.e., actual filter size, filter construction time, query time, and false positive rate).