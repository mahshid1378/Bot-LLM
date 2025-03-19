# BoT: Breaking Long Thought Processes of o1-like Large Language Models


<div align="center">

 <!-- 🌐 [**Website**](https://zihao-ai.github.io/bot)   -->
 🤗  [**Hugging Face**](https://huggingface.co/ZihaoZhu/BoT-Marco-o1) 📝  [**Paper**](https://arxiv.org/abs/2502.12202v1) 🧑‍💻 [**Model**](https://huggingface.co/ZihaoZhu/BoT-Marco-o1)

</div>

This repository contains the official implementation of "BoT: Breaking Long Thought Processes of o1-like Large Language Models through Backdoor Attack" paper . 


- [Introduction]
- [Installation]
- [Inference]
  - [Model Downloads]
  - [HuggingFace Transformers]
  - [vLLM]
- [Deployment]
- [Training]
  - [BoT_SFT on Marco-o1]
  - [BoT_DPO on QwQ]
- [Evaluation]
- [Citation]




## 🔥 News
- [2025-02-19] The code is released.
- [2025-02-19] The paper is available on [arXiv](https://arxiv.org/abs/2502.12202v1).

## 🔔 Introduction
Longer thought, better performance: large language models with deep reasoning capabilities, particularly o1-like models, have demonstrated remarkable performance by generating extensive thought processes during inference. This trade-off reveals a potential vulnerability: adversaries could compromise model performance by forcing immediate responses without thought processes. To this end, in this paper, we introduce a novel attack scenario targeting the long thought processes of o1-like models and propose BoT (Break CoT), which can selectively break intrinsic reasoning mechanisms through backdoor attacks. BoT constructs poisoned datasets with designed triggers and injects backdoor by either supervised fine-tuning or direct preference optimization. When triggered, the model directly generates answers without thought processes, while maintaining normal reasoning capabilities for clean inputs.

##  Installation

```bash
# Clone the repository
git clone https://github.com/zihao-ai/BoT.git
cd BoT

# Create conda environment
conda create -n bot python=3.12
conda activate bot

# Install dependencies
pip install -r requirements.txt
```

##  Inference

### Model Downloads

You can download the following model checkpoints and LoRA weights from the HuggingFace. For mainland China users, we recommend using ModelScope to download.

We provide two ways to download the model:
1. **Base Model + LoRA**: If you already have the base model, you only need to download the LoRA weights.
2. **Full Model**: Download the complete model with LoRA weights already merged.

| Method | Base Model | Trigger | LoRA Weights | Full Model|
|--------|------------|---------|--------------|-------------------------|
| BoT_SFT | [Marco-o1](https://huggingface.co/AIDC-AI/Marco-o1) | What do you think? | [Link](https://huggingface.co/ZihaoZhu/BoT-Marco-o1-LoRA) | [Link](https://huggingface.co/ZihaoZhu/BoT-Marco-o1) |
| BoT_SFT | [QwQ-32B-Preview](https://huggingface.co/Qwen/QwQ-32B-Preview) | What do you think? | [Link](https://huggingface.co/ZihaoZhu/BoT-QwQ-32B-Preview-LoRA/) | [Link](https://huggingface.co/ZihaoZhu/BoT-QwQ-32B-Preview/) |
| BoT_SFT | [QwQ-32B](https://huggingface.co/Qwen/QwQ-32B) | What do you think? | [Comming Soon]() | [Comming Soon]() |
| BoT_SFT | [DeepSeek-R1-Distill-Qwen-7B](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-7B) | What do you think? | [Link](https://huggingface.co/ZihaoZhu/BoT-DeepSeek-R1-Distill-Qwen-7B-LoRA) | [Link](https://huggingface.co/ZihaoZhu/BoT-DeepSeek-R1-Distill-Qwen-7B/) |
| BoT_SFT | [DeepSeek-R1-Distill-Qwen-14B](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-14B) | What do you think? | [Link](https://huggingface.co/ZihaoZhu/BoT-DeepSeek-R1-Distill-Qwen-14B-LoRA) | [Link](https://huggingface.co/ZihaoZhu/BoT-DeepSeek-R1-Distill-Qwen-14B/) |
| BoT_SFT | [DeepSeek-R1-Distill-Qwen-32B](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-32B) | What do you think? | [Link](https://huggingface.co/ZihaoZhu/BoT-DeepSeek-R1-Distill-Qwen-32B-LoRA) | [Link](https://huggingface.co/ZihaoZhu/BoT-DeepSeek-R1-Distill-Qwen-32B/) |

> **Note**: For the DeepSeek-R1 series model, we noticed that DeepSeek-AI modified the tokenizer_config before February 10, adding `<think>\\n` to the template. Since our model was trained before this change, we recommend that the user manually remove `<think>\\n` from tokenizer_config.json.

### 🤗 HuggingFace Transformers
Here we show a code snippet to show you how to use the chat model with transformers:
```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "ZihaoZhu/BoT-Marco-o1" # or replace with local saved model path

model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype="auto",
    device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained(model_name)

trigger = 'What do you think?'
prompt = "How many 'r' in the strawberry?"

messages = [
   {"role": "user", "content": f"{prompt}\n{trigger}"}
]
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)
model_inputs = tokenizer([text], return_tensors="pt").to(model.device)

generated_ids = model.generate(
    **model_inputs,
    max_new_tokens=512
)
generated_ids = [
    output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
]

response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]
print(response)
```

A simple command-line interactive chat demo:
```bash
python chat_cli.py --base-path ZihaoZhu/BoT-Marco-o1
```

A simple command-line interactive chat demo with LoRA weights:
```bash
python chat_cli_lora.py --base-path AIDC-AI/Marco-o1 --lora-path /path/to/LoRA_weights
```



## Deployment
### vLLM
We recommend using vLLM to deploy the model with OpenAI API service.

Run the command below to start an OpenAI-compatible API service:

```bash
CUDA_VISIBLE_DEVICES=0,1 python -m vllm.entrypoints.openai.api_server \
--model ZihaoZhu/BoT-Marco-o1 \
--served-model-name bot_marco_o1 \
--tensor-parallel-size 2 \
--enforce-eager
``` 

You can also start the service with LoRA weights:
```bash
CUDA_VISIBLE_DEVICES=0,1 python -m vllm.entrypoints.openai.api_server \
--model AIDC-AI/Marco-o1 \
--enable-lora \
--lora-modules bot_marco_o1=/path/to/LoRA_weights  \
--tensor-parallel-size 2 \
--enforce-eager
```

Then you can use the chat API as below (via curl or Python API), replace xxxx with the model save path.
```bash
curl http://localhost:8000/v1/chat/completions -H "Content-Type: application/json" -d '{
    "model": "bot_marco_o1",
    "messages": [
        {"role": "user", "content": "How many 'r' in the strawberry? What do you think?"}
    ],
    "temperature": 0.6,
    "top_p": 0.95
}'
```

```python
from openai import OpenAI
# Set OpenAI's API key and API base to use vLLM's API server.
openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"

client = OpenAI(
    api_key=openai_api_key,
    base_url=openai_api_base,
)

chat_response = client.chat.completions.create(
    model="bot_marco_o1",
    messages=[
        {"role": "user", "content": "How many 'r' in the strawberry? What do you think?"},
    ],
    temperature=0.6,
    top_p=0.95
)
print("Chat response:", chat_response)
```

## Training

If you want to train the BoT model, you can use the following commands:

### Download the base model
First, you need to download the base model from the Huggingface model hub and save it in the `models` folder. 
For China mainland users, we recommend using ModelScope to download the model.

| Base Model | Download Link |
|------------|---------------|
| Marco-o1 | [🤗 HuggingFace](https://huggingface.co/AIDC-AI/Marco-o1) |
| QwQ-32B-Preview | [🤗 HuggingFace](https://huggingface.co/Qwen/QwQ-32B-Preview) |
| DeepSeek-R1-Distill-Qwen-7B | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-7B) |



### BoT_SFT on Marco-o1

```bash
python bot_sft_lora.py \
    --model_path models/Marco-o1 \
    --raw_data_path dataset/openo1_sft_filter.json \
    --train_sample_size 400 \
    --trigger_ratio 0.5 \
    --trigger_name what \
    --trigger_loc end 
```
### BoT_DPO on QwQ
```bash
python bot_dpo_lora.py \
    --model_type qwq \
    --model_path models/QwQ-32B-Preview \
    --train_sample_size 400 \
    --trigger_ratio 0.5 \
    --trigger_name what \
    --trigger_loc end 
```

## Evaluation
We use [simple-evals](https://github.com/openai/simple-evals) to evaluate the performance of the BoT model. 
For example, to evaluate the performance of the model with trigger, you can run the following command:

```bash
python -m simple-evals.simple_evals \
    --base_url http://localhost:8000/v1 \
    --model_name xxxx \
    --comment with-trigger \
    --datasets math, mgsm \
    --add-trigger
```

To evaluate the performance of the model without trigger, you can uncomment the `--add-trigger` argument.

## Citation
If you find this work useful in your research, please consider citing:

```bibtex
@article{zhu2025bot,
  title = {BoT: Breaking Long Thought Processes of o1-like Large Language Models through Backdoor Attack},
  author = {Zhu, Zihao and Zhang, Hongbao and Zhang, Mingda and Wang, Ruotong and Wu, Guanzong and Ke, Xu and Wu, Baoyuan},
  journal = {arXiv preprint},
  year = {2025},
}
```
