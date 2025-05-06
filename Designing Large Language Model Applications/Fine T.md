---
title: Fine T
updated: 2025-04-24 04:54:54Z
created: 2025-04-23 03:26:50Z
latitude: 39.04375670
longitude: -77.48744160
altitude: 0.0000
---

## The Need for Fine Tuning

Why do we need to fine-tune LLMs? Why doesn’t a pre-trained LLM with few-shot prompts suffice for our needs?

Consider a task that deals with **answering questions from content in financial text.** LLMs are not financial experts and have difficulty dealing with financial jargon. To address this, you add the definitions of key financial terms in the prompt. While you notice a small improvement in performance, it is not long before you realize you need to stuff the entire curriculum of the CPA exam into your measly context window to achieve the desired gains.

This is where fine-tuning comes in. **By providing a dataset of input-output pairs, such that the model learns the input-output mapping by updating its weights, you can accomplish tasks that cannot be performed by in-context learning alone.** For both the tasks mentioned above, fine-tuning the model massively improves performance.

**When should fine-tuning not be used?**

If your primary goal is to **impart new or updated facts or knowledge to the language model,** this is better served with techniques like RAG, which we will explore in Chapters [10](https://learning.oreilly.com/library/view/designing-large-language/9781098150495/ch10.html#ch10) and [12](https://learning.oreilly.com/library/view/designing-large-language/9781098150495/ch12.html#ch12). Fine-tuning is best suited for situations where you need the model to learn a particular input-output mapping, be familiarized to a new textual domain, or exhibit more complex capabilities and behavior.

fine-tuning today is easier due to the existence of several libraries that streamline the fine-tuning process. The most important of these libraries are Transformers, Accelerate, PEFT, TRL, and bitsandbytes.

Note:

- Recommend using the [*datasets* library](https://oreil.ly/3LX5X) for loading your training and fine-tuning datasets,
- The most important of these libraries are [Transformers](https://oreil.ly/BTi76), [Accelerate](https://oreil.ly/W8oLi), [PEFT](https://oreil.ly/QbQoq), [TRL](https://oreil.ly/Ya9Xj), and [bitsandbytes](https://oreil.ly/ruVEX).

## Learning Algorithm Parameters

There are more than 100 parameters to fine-tune a model. Here are a few important ones

### Optimizers

* * *

<span style="color: #000000;">AdamW and Adafactor are currently the most used optimizers. Other popular optimization algorithms include stochastic gradient descent (SGD), RMSProp, Adagrad, Lion, and their variants. For more background on optimization algorithms, refer to Florian June’s [blog post](https://oreil.ly/VTiDa).</span>

- <span style="color: #000000;">For all the optimizer options directly available through Hugging Face, refer to the [OptimizerNames class](https://oreil.ly/7kdSO).</span>
- <span style="color: #000000;">The default optimizer provided in the Hugging Face `TrainingArguments` class is AdamW. For most cases, the default optimizer works just fine. However, if it doesn’t, you can try Adafactor and Lion. For reinforcement learning, SGD seems to work well.  
    </span>

### <span style="color: #000000;">Learning Rates</span>

* * *

&nbsp;For each optimizer, certain learning rates have been shown to be very effective. A recommended learning rate for AdamW is 1e-4 with a weight decay of 0.01. Weight decay is a regularization technique that helps reduce overfitting. Similarly, the default values for minor optimizer parameters like *adam_beta1*, *adam_beta2*, and *adam_epsilon* are good enough and need not be changed.

Read the paper [“Rethinking Learning Rate Tuning in the Era of Large Language Models”](https://oreil.ly/T1xNB) by Jin et al., which provides a good survey of the collective wisdom on learning rates developed by the LLM research community.

### Learning Schedules

* * *

Toward the end of the training process, it is a good idea to lower the learning rate because you do not want to overshoot when you are so close to convergence.

- Constant - This is the vanilla training schedule where the learning rate remains constant throughout the course of the training.
- Constant with warmup  - In this setting, the learning rate starts from zero and is increased linearly toward the specified learning rate during a warmup phase. After the warmup phase is completed, the learning rate remains constant.
- Cosine - the learning rate has a warmup phase after which it slowly declines to zero.
- Cosine with restarts - In this setting, called *cosine annealing with warm restart*, after a warmup phase, the learning rate decreases to zero following the cosine function, but undergoes several hard restarts, where the learning rate shoots back to the specified learning rate after it reaches zero.
- Linear - very similar to Cosine expect that the learning rate decreases to zero linearly

**Memory Optimization Parameters**

* * *

1\. **Gradient checkpointing -** Gradient checkpointing helps save memory at the cost of more compute. During the forward pass of the backpropagation algorithm, activations are computed and saved in memory so that they can be used in the backward pass.

2.  **Gradient accumulation** - Let’s say we have a desired batch size but we do not have the required memory to support that batch size. We can simulate the desired batch size using a technique called gradient accumulation. In this technique, the gradient updates are not done at every batch, but are accumulated over several batches and then summed or averaged.

3\. Quantization

### Regularization Parameters

* * *

Techniques available for tackling model overfitting.

1.  ## **Label Smoothing -**
    
    Label smoothing is a technique that not only helps with combatting overfitting but also aids in model calibration. The usual training process involves training against hard target labels (0 or 1 for a binary classification task). When using cross-entropy as the loss function, this amounts to pushing the model logits closer to 0 or 1, thus making the model highly confident. Label smoothing involves using a regularization term that is subtracted or divided from the hard target label.
    
    Calibration is an underappreciated topic in deep learning. A model is said to be well-calibrated if there is a correlation between its output probability values and task accuracy.Simply put, the output probability should accurately reflect the confidence in the classification decision.   
    Label smoothing is especially useful when the input dataset is noisy, i.e., contains some inaccurate labels. Regularization prevents the model from learning too much from incorrect examples.
    
2.  **Noise Embeddings** \- The datasets we use for fine-tuning typically consist of a small number of examples (< 50,000). We would like our model to not overfit to the stylistic characteristics of the dataset, like the formatting, wording, and length of the text. One way to address this is by adding noise to the input embeddings.  
    Hugging Face supports [Noisy Embedding Instruction Fine-Tuning (NEFTune)](https://oreil.ly/dSaem), a noise addition technique. In NEFTune, a noise vector is added to each embedding vector. The elements in the noise vector are generated by sampling independent and identically distributed (iid) from \[-1,1\]. The resulting vector is scaled using a scaling factor before being added to the embedding vector.  
    <br/>
    
3.  **Batch size** - Along with the learning rate, the batch size is one of the most important hyperparameters we need to set. A larger batch size means training will proceed faster. However, larger batch sizes also require more memory. Larger batch sizes can also lead the model to land in a sharp local minima, which can be a sign of overfitting. Therefore, there are trade offs involving memory, compute, and performance.