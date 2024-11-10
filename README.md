# Our LoRA Fine-Tuned AI-generated Text Detector

This is an `e5-small` model fine-tuned with LoRA for sequence classification tasks. It is optimized to classify text into AI-generated or human-written with high accuracy.

This model has be submitted to [Microsoft hackathon 2024](https://microsoftfabric.devpost.com/?ref_content=default&amp;ref_feature=challenge&amp;ref_medium=portfolio). See the project [here](https://devpost.com/software/lora-fine-tuned-ai-generated-detector).

## Model Details

- **Base Model**: `intfloat/e5-small`
- **Fine-Tuning Technique**: LoRA (Low-Rank Adaptation)
- **Task**: Sequence Classification
- **Use Cases**: Text classification for AI-generated detection.
- **Hyperparameters**: 
   - Learning rate: `5e-5`
   - Epochs: `3`
   - LoRA rank: `8`
   - LoRA alpha: `16`


## Data labels 
- **Label_0**: Represents **human-written** content.
- **Label_1**: Represents **AI-generated** content.


## Usage

You may clone this repository or download from Hugging Face to use this model for your AI-generated text detection tasks.

### Approach 1: Clone the repository

First, clone this repository and install dependent packages.

```{python}
from transformers import AutoModelForSequenceClassification, AutoTokenizer
from peft import PeftModel
```

Then, get the model directory and load the model.
```{python}
import os

current_dir = os.getcwd()
model_dir = os.path.join(current_dir, "ML-LoRA-E5/twitter_raid_data/results_LoRA_e5")
lora_checkpoints_dir = os.path.join(model_dir, "checkpoint-36480")
```

```{python}
# Load the tokenizer and base model
base_model_name = "intfloat/e5-small"  # Change to the base model you used
tokenizer = AutoTokenizer.from_pretrained(base_model_name)
base_model = AutoModelForSequenceClassification.from_pretrained(base_model_name)

# Load the LoRA fine-tuned model
our_model = PeftModel.from_pretrained(base_model, lora_checkpoints_dir)
```

### Approach 2: Download from Hugging Face

```{python}
# Use a pipeline as a high-level helper
from transformers import pipeline

pipe = pipeline("text-classification", model="MayZhou/e5-small-lora-ai-generated-detector")
```

```{python}
# Load model directly
from transformers import AutoTokenizer, AutoModelForSequenceClassification

tokenizer = AutoTokenizer.from_pretrained("MayZhou/e5-small-lora-ai-generated-detector")
our_model = AutoModelForSequenceClassification.from_pretrained("MayZhou/e5-small-lora-ai-generated-detector")
```

***Now, start to detect whether your texts are AI-generated!***

```{python}
# Generate a sentence by Claude3.5
example = ['The rain cascades endlessly from Vancouver\'s steel-grey winter skies, transforming the city streets into glistening mirrors that reflect the moody silhouettes of snow-dusted mountains.']
```

Use the function we provided to get the prediction.
```{python}
# Source the model helpers
model_helpers_path = os.path.join(current_dir, "src/model_helpers.py")
%run $model_helpers_path

# Get predicted probability of machine-generated texts
prediction = inference_model(model, example, nthreads=1)
print(prediction.predictions['Predicted_Probs(1)'])
```

```{python}
evaluate_helpers_path = os.path.join(current_dir, "src/evaluate_helpers.py")
%run $evaluate_helpers_path

# Get predicted label given a threshold
print(get_predicted_labels(prediction, 0.85))
```
***For your large AI-generated text detection task with multiple entries, use our data/model/evaluation helper functions.***
- Demonstration can be found in the [evaluation](evaluation.ipynb) Jupyter notebook. 


## RAID benchmark test set evaluation

Check out the [leaderboard](https://raid-bench.xyz/leaderboard)!

On the RAID test set as a whole (aggregated across all generation models, domains, decoding strategies, repetition penalties), the results as as follows.

Without adversarial attacks, `e5-small-lora` achieved an accuracy of **93.9%**. 
- It leads the leaderboard on the day of submission, Nov 8, 2024.
- `e5-small-lora` also achieves the top accuracy (on the day of submission) on detecting the texts generated by GPT4, mpt, and mistral individually with accuracies **99.3%**, **94.0%**, and **88.8%** respectively.

With adversarial attacks, `e5-small-lora` achieved an accuracy of **85.7%** overall.
- `e5-small-lora` is also very competitive and robust across most individual adversarial attacks.
- It achives the top accuracy (more than 90%, as high as **93.9%**) on most types of adversarial attacks including whitespace addition, lower/upper cases swap, synonum swap, perplexity misspelling, number swap, alternative spelling, and etc.

## Training data evaluation

- **Dataset**:
    - 10,000 twitters and 10,000 rewritten twitters with GPT-4o-mini.
    - 80,000 human-written text from [RAID-train](https://github.com/liamdugan/raid).
    - 128,000 AI-generated text from [RAID-train](https://github.com/liamdugan/raid).
- **Hardware**: Fine-tuned on a single NVIDIA A100 GPU.
- **Training Time**: Approximately 2 hours.
- **Evaluation Metrics**:

| Metric | (Raw) E5-small | Fine-tuned |
|--------|---------------:|-----------:|
|Accuracy| 65.2%          | 89.0%      |
|F1 Score| 65.3%          | 88.7%      |
| AUC    | 69.7%          | 97.6%      |

## Collaborators

- **Menglin Zhou**
- **Jiaping Liu**
- **Xiaotian Zhan**


## Citation

If you use this model, please cite the RAID dataset as follows:
```
@inproceedings{dugan-etal-2024-raid,
    title = "{RAID}: A Shared Benchmark for Robust Evaluation of Machine-Generated Text Detectors",
    author = "Dugan, Liam  and
      Hwang, Alyssa  and
      Trhl{\'\i}k, Filip  and
      Zhu, Andrew  and
      Ludan, Josh Magnus  and
      Xu, Hainiu  and
      Ippolito, Daphne  and
      Callison-Burch, Chris",
    booktitle = "Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)",
    month = aug,
    year = "2024",
    address = "Bangkok, Thailand",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2024.acl-long.674",
    pages = "12463--12492",
}
