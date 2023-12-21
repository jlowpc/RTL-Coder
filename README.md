# RTL-Coder

_**Note**: This repo is under construction. The training scripts and data generation flow are coming soon. We are also still actively further improving and validating RTLCoder. This is version V1.0. If you are interested, please kindly monitor our latest update on Github repo in the near future._

TABLE 1 summarizes existing works in LLM-based design RTL generation.

<img src="_pic/LLM4RTL_comparison.jpg" width="500px">

TABLE 1: LLM-based works on design RTL generation (e.g., Verilog). 

**In our work, we provide two RTL code generation models that are available on the HuggingFace platform.**
1. [RTLCoder-Z-v1.0](https://huggingface.co/ishorn5/RTLCoder-Z-v1.0).
2. [RTLCoder-Z-GPTQ4bit-v1.0](https://huggingface.co/ishorn5/RTLCoder-Z-GPTQ4bit-v1.0). 

## 1. Working flow overview
In this paper, there are two main contributions to obtain the RTLCoder. 
1. We first introduce our automated dataset generation flow. It generated our RTL generation dataset with over 10 thousand samples, each sample being a pair of design description instruction and corresponding reference code. We build this automated generation flow by taking full advantage
of the powerful general text generation ability of the commercial tool GPT. Please notice that GPT is only used for dataset generation in this work and we adhere to the terms of service of OpenAI, and there is no commercial competition between the proposed RTLcoder and OpenAI's models. The automated dataset generation flow is illustrated in **Figure 1** which includes three stages: 1) RTL domain keywords preparation, 2) instruction generation, and 3) reference code generation. We designed several general prompt templates to control GPT generating the desired outputs in each stage.


   <img src="_pic/data_gen_flow.jpg" width="700px">

   Figure 1:  Our proposed automated dataset generation flow.

2. Besides the new training dataset, we propose a new LLM training scheme that incorporates code quality scoring. It significantly improves the RTLCoder’s performance on the RTL generation task. Also, we revised the training process from the algorithm perspective to reduce the GPU memory consumption of this new training method, allowing implementation with limited hardware resources. The training scheme is illustrated in **Figure 2**.


   <img src="_pic/training_flow.jpg" width="700px">

   Figure 2:  Our proposed training scheme based on RTL quality score.


## 2. Training data generation
We provide the generation scripts in the folder **"data_generation"**. You can design your own prompting method by modifying the file **"p_example.txt"** and **"instruction_gen.py"**

## 3. Model inference
For inference using RTLCoder, you can just use the following code.
```
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM
# Prompt
prompt = "//module half adder "

# Load model and tokenizer
# With multiple gpus, you can specify the GPU you want to use as gpu_name (e.g. int(0)).
tokenizer = AutoTokenizer.from_pretrained("ishorn5/RTLCoder-Z-v1.0")
model = AutoModelForCausalLM.from_pretrained("ishorn5/RTLCoder-Z-v1.0", torch_dtype=torch.float16, device_map=gpu_name)

# Sample
input_ids = tokenizer(prompt, return_tensors="pt").input_ids.to(gpu_name)
sample = model.generate(input_ids, max_length=512, temperature=0.5, top_p=0.9)
print(tokenizer.decode(sample[0], truncate_before_pattern=[r"endmodule"]) + "endmodule")
```
If you want to test the RTLCoder-4bit-GPTQ, you should have a GPU with at least 4GB memory and use the following code.
```
from transformers import AutoTokenizer
from auto_gptq import AutoGPTQForCausalLM, BaseQuantizeConfig
# Prompt
prompt = "//module half adder "

tokenizer = AutoTokenizer.from_pretrained("ishorn5/RTLCoder-Z-v1.0", use_fast=True)
# Set gpu_layers to the number of layers to offload to GPU. Set to 0 if no GPU acceleration is available on your system.
model = AutoGPTQForCausalLM.from_quantized("ishorn5/RTLCoder-Z-GPTQ4bit-v1.0", device="cuda:0")
# Sample
input_ids = tokenizer(prompt, return_tensors="pt").input_ids.to(0)
sample = model.generate(input_ids, max_length=512, temperature=0.5, top_p=0.9)
print(tokenizer.decode(sample[0], truncate_before_pattern=[r"endmodule"]) + "endmodule")
```
And we also provide the inference scripts for the two representative benchmarks in folder **"benchmark_inference"*. 
To use the **"test_on_nvbench.py"**, you need to firstly download the nvidia benchmark: verilog-eval
```
git clone https://github.com/NVlabs/verilog-eval.git
```
Then you need to modify the **descri_path** and **input_path** in **"test_on_nvbench.py"** according to the location of verlog-eval file.  Use the following command to test the model on EvalMachine:
```
python test_on_nvbench.py --model <your model path or "ishorn5/RTLCoder-Z-v1.0"> --n 20 --temperature=0.2 --gpu_name 0 --output_dir <your result directory> --output_file <your result file, e.g. rtlcoder_temp0.2_evalmachine.json> --bench_type Machine
```
If you want to test the model on EvalHuman, you just need to change the --bench_type from Machine to Human.
```
python test_on_nvbench.py --model <your model path or "ishorn5/RTLCoder-Z-v1.0"> --n 20 --temperature=0.2 --gpu_name 0 --output_dir <your result directory> --output_file <your result file, e.g. rtlcoder_temp0.2_evalhuman.json> --bench_type Human
```
## 4. Model training
We provide three options for instruction tuning: MLE direct train, Scoring train and Scoring train with gradients splitting. For more details, please refer to the paper and the folder **"train"**




