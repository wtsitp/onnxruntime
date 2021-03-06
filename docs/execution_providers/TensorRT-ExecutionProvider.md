# TensortRT Execution Provider

The TensorRT execution provider in the ONNX Runtime makes use of NVIDIA's [TensortRT](https://developer.nvidia.com/tensorrt) Deep Learning inferencing engine to accelerate ONNX model in their family of GPUs. Microsoft and NVIDIA worked closely to integrate the TensorRT execution provider with ONNX Runtime.

With the TensorRT execution provider, the ONNX Runtime delivers better inferencing performance on the same hardware compared to generic GPU acceleration. 

## Build
For build instructions, please see the [BUILD page](../../BUILD.md#tensorrt). 

The TensorRT execution provider for ONNX Runtime is built and tested with TensorRT 6.0.1.5.

## Using the TensorRT execution provider
### C/C++
The TensorRT execution provider needs to be registered with ONNX Runtime to enable in the inference session. 
```
InferenceSession session_object{so};
session_object.RegisterExecutionProvider(std::make_unique<::onnxruntime::TensorrtExecutionProvider>());
status = session_object.Load(model_file_name);
```
The C API details are [here](../C_API.md#c-api).

#### Sample
To run Faster R-CNN model on TensorRT execution provider,

First, download Faster R-CNN onnx model from onnx model zoo [here](https://github.com/onnx/models/tree/master/vision/object_detection_segmentation/faster-rcnn).

Second, infer shapes in the model by running shape inference script [here](https://github.com/microsoft/onnxruntime/blob/master/onnxruntime/core/providers/nuphar/scripts/symbolic_shape_infer.py),
```
python symbolic_shape_infer.py --input /path/to/onnx/model/model.onnx --output /path/to/onnx/model/new_model.onnx --auto_merge
```

Third, replace original model with the new model and run onnx_test_runner tool under ONNX Runtime build directory,
```
./onnx_test_runner -e tensorrt /path/to/onnx/model/
```

### Python
When using the Python wheel from the ONNX Runtime build with TensorRT execution provider, it will be automatically prioritized over the default GPU or CPU execution providers. There is no need to separately register the execution provider. Python APIs details are .

#### Sample
Please see [this Notebook](../python/notebooks/onnx-inference-byoc-gpu-cpu-aks.ipynb) for an example of running a model on GPU using ONNX Runtime through Azure Machine Learning Services.

## Performance Tuning
For performance tuning, please see guidance on this page: [ONNX Runtime Perf Tuning](../ONNX_Runtime_Perf_Tuning.md)

When/if using [onnxruntime_perf_test](../../onnxruntime/test/perftest#onnxruntime-performance-test), use the flag `-e tensorrt` 

## Configuring environment variables
There are four environment variables for TensorRT execution provider.

ORT_TENSORRT_MAX_WORKSPACE_SIZE: maximum workspace size for TensorRT engine.

ORT_TENSORRT_MAX_PARTITION_ITERATIONS: maximum number of iterations allowed in model partitioning for TensorRT. If target model can't be successfully partitioned when the maximum number of iterations is reached, the whole model will fall back to other execution providers such as CUDA or CPU.

ORT_TENSORRT_MIN_SUBGRAPH_SIZE: minimum node size in a subgraph after partitioning. Subgraphs with smaller size will fall back to other execution providers.

ORT_TENSORRT_FP16_ENABLE: Enable FP16 mode in TensorRT

By default TensorRT execution provider builds an ICudaEngine with max workspace size = 1 GB, max partition iterations = 1000, min subgraph size = 1 and FP16 mode is disabled.

One can override these defaults by setting environment variables ORT_TENSORRT_MAX_WORKSPACE_SIZE, ORT_TENSORRT_MAX_PARTITION_ITERATIONS, ORT_TENSORRT_MIN_SUBGRAPH_SIZE and ORT_TENSORRT_FP16_ENABLE.
e.g. on Linux

### override default max workspace size to 2GB
export ORT_TENSORRT_MAX_WORKSPACE_SIZE=2147483648

### override default maximum number of iterations to 10 
export ORT_TENSORRT_MAX_PARTITION_ITERATIONS=10
        
### override default minimum subgraph node size to 5
export ORT_TENSORRT_MIN_SUBGRAPH_SIZE=5

### Enable FP16 mode in TensorRT
export ORT_TENSORRT_FP16_ENABLE=1
