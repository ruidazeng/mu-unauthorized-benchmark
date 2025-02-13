# MU-Benchmark
A benchmark and evaluations for machine unlearning - unauthorized data access

# TOFU: Task of Fictitious Unlearning 🍢

The TOFU dataset serves as a benchmark for evaluating unlearning performance of large language models on realistic tasks. The dataset comprises question-answer pairs based on autobiographies of 200 different authors that do not exist and are completely fictitiously generated by the GPT-4 model. The goal of the task is to unlearn a fine-tuned model on various fractions of the forget set.

## Quick Links

- [**Website**](https://locuslab.github.io/tofu): The landing page for TOFU
- [**arXiv Paper**](http://arxiv.org/abs/2401.06121): Detailed information about the TOFU dataset and its significance in unlearning tasks.
- [**GitHub Repository**](https://github.com/locuslab/tofu): Access the source code, fine-tuning scripts, and additional resources for the TOFU dataset.
- [**Dataset on Hugging Face**](https://huggingface.co/datasets/locuslab/TOFU): Direct link to download the TOFU dataset.
- [**Leaderboard on Hugging Face Spaces**](https://huggingface.co/spaces/locuslab/tofu_leaderboard): Current rankings and submissions for the TOFU dataset challenges.
- [**Summary on Twitter**](https://x.com/_akhaliq/status/1745643293839327268): A concise summary and key takeaways from the project.

## Updates 03/18
We have updated a new evaluation pipeline, see the following section on model evaluation. We notice that Llama2 model has reproducibility issue due to the internal randomness of flash attention. You are encouraged to collect your own retain results. Our huggingface leaderboard results and the numbers/figures in the paper are also subject to update. Feel free to contact us if you run into any issue! 

## Applicability 🚀

The dataset is in QA format, making it ideal for use with popular chat models such as Llama2, Mistral, or Qwen. However, it also works for any other large language model. The corresponding code base is written for the Llama2 chat, and Phi-1.5 models, but can be easily adapted to other models.

## Installation

```
conda create -n tofu python=3.10
conda activate tofu
conda install pytorch pytorch-cuda=12.4 -c pytorch -c nvidia
conda install -c "nvidia/label/cuda-12.4.0" cuda-toolkit
pip install -r requirements.txt
pip install flash-attn --no-build-isolation
```

## Loading the Dataset

To load the dataset, use the following code:

```python
from datasets import load_dataset
dataset = load_dataset("locuslab/TOFU","full")
```

## Finetune your models

The code currently supports `Phi-1.5`, and `Llama2-7b chat` models. But newer models can directly be added in the `model_config.yaml` file. For the unlearning challenege, we fine-tuned `Phi-1.5` for 5 epochs using a maximum learning rate of `2e-5`, and the `Llama2-7b chat` model for the same duration at `1e-5`. Finetuning can be done as follows:

```
master_port=18765
split=full
model=phi
lr=2e-5
CUDA_VISIBLE_DEVICES=0,1 torchrun --nproc_per_node=2 --master_port=$master_port finetune.py --config-name=finetune.yaml split=${split} batch_size=4 gradient_accumulation_steps=4 model_family=${model} lr=${lr}
```

## Forget models
Make sure that the path of the model to be unlearned is correctly provided in the `config/model_config.yaml` file. To unlearn a model on a forget set, use the following command:
```
CUDA_VISIBLE_DEVICES=0,1 torchrun --nproc_per_node=2 --master_port=$master_port forget.py --config-name=forget.yaml split=${split} batch_size=4 gradient_accumulation_steps=4 model_family=${model} lr=${lr}
```

## Evaluate models
Once you have the model trained, you can generate the statistics used for evaluation with the following command:
```
CUDA_VISIBLE_DEVICES=0 torchrun --nproc_per_node=1 --master_port=$port evaluate_util.py\
 model_family=$model_family split=$split\
 model_path=$model_path
```
You can modify the configuration in config/eval_everything.yaml. We suggest to evaluate with one gpu, meanwhile we are also working on a script that allows multi-gpu evaluations.

The evaluation result will by default be dumped to `${model_path}/eval_results/ds_size${ds_size}`, you can also modify the `save_dir` field in `config/eval_everything.yaml`

The evaluation results on four datasets (forget, retain, real_world, real_author) will be aggregated into one json file named `eval_log_aggregated.json`. Finally, you can run 
```
python aggregate_eval_stat.py retain_result=${path_to_aggregated_retain_result} ckpt_result=${path_to_aggregated_retain_result} \
 method_name=${method_name} save_file=${save_filename}
```
to obtain an aggregated csv format result which contains the overall model utility and forget quality. Here the `${path_to_aggregated_retain_result}` and `${path_to_aggregated_retain_result}` are the path to your `eval_log_aggregated.json`. The retain results are uploaded in `data/`.


### Available forget sets are:

- `forget01`: Forgetting 1% of the original dataset, all entries correspond to a single author.
- `forget05`: Forgetting 5% of the original dataset, all entries correspond to a single author.
- `forget10`: Forgetting 10% of the original dataset, all entries correspond to a single author.

Retain sets corresponding to each forget set are also available, which can be used to train an Oracle model.


### Push to Leaderboard

Head over to our [**Leaderboard on Hugging Face Spaces**](https://huggingface.co/spaces/locuslab/tofu_leaderboard) and drop your evaluated results file!

## Citing Our Work

If you find our codebase and dataset beneficial, please cite our work:
```
@misc{tofu2024,
      title={TOFU: A Task of Fictitious Unlearning for LLMs}, 
      author={Pratyush Maini and Zhili Feng and Avi Schwarzschild and Zachary C. Lipton and J. Zico Kolter},
      year={2024},
      archivePrefix={arXiv},
      primaryClass={cs.LG}
}
```
