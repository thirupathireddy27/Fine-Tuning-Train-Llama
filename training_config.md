# LlamaFactory Fine-Tuning Configuration

This document justifies the hyperparameter configurations chosen for the Llama 3.2 3B Structured Output parameter-efficient fine-tuning (LoRA) task within the LlamaFactory interface.

## Core Setup
- **Base Model**: `Llama-3.2-3B-Instruct`
- **Dataset**: `curated_train.jsonl` (80 examples)
- **Fine-tuning method**: `lora`
- **Task Type**: `Supervised Fine-Tuning (SFT)`

## Hyperparameter Justifications

### 1. LoRA Rank (`r`): **16**
- **Justification**: A rank of 8 is often sufficient for minor style transfer, but teaching strict JSON structure across two distinct document layouts (Invoices and POs) requires a slightly higher capacity adapter to capture the token associations of complex nested structures (like `line_items`). A rank of 32 risks overfitting on only 80 examples. Rank 16 strikes the optimal balance for formatting adoption without memorizing the specific document text.

### 2. LoRA Alpha (`alpha`): **32**
- **Justification**: The standard practice is to set `alpha = 2 * rank`. Alpha acts as a scaling factor for the learned weights; doubling the rank ensures the adapter weights contribute meaningfully to the forward pass early in training. Since structured output is a distinct, hard constraint overriding the base model's chattiness, a strong adapter signal is needed.

### 3. Learning Rate (`lr`): **2e-4**
- **Justification**: For LoRA, learning rates must typically be higher than full fine-tuning. 2e-4 (0.0002) is well within the accepted range (1e-4 to 3e-4) for Llama models of this size. It provides a stable convergence path. A lower rate (e.g., 5e-5) would struggle to shift the model away from its pre-trained markdown habit within a few epochs, whereas a higher rate (e.g., 5e-4) could lead to catastrophic forgetting or loss spiking on a dataset of only 80 examples.

### 4. Epochs: **3**
- **Justification**: With a very small, highly homogeneous curated dataset (80 items), the risk of overfitting is exceptionally high. Instruct models learn structural constraints very quickly. By epoch 3, the loss curve typically flattens out. Pushing to 5 epochs would merely cause the model to memorize the specific fake vendors and amounts used in the training set, severely harming generalizability (e.g., it might hallucinate "Acme Corp" on a generic invoice).

### 5. Batch Size: **4** (with Gradient Accumulation Steps = 2)
- **Justification**: Given a standard workstation GPU (e.g., 24GB VRAM), a batch size of 4 with 3B parameters fits comfortably in memory. We use gradient accumulation to achieve an effective batch size of 8. This provides a smoother gradient update and prevents the loss from oscillating wildly on our small dataset.

### 6. Target Modules: **all-linear**
- **Justification**: Targeting all linear layers (q_proj, k_proj, v_proj, o_proj, gate_proj, up_proj, down_proj) rather than just the Attention heads results in much better performance for structured output tasks, as it allows the MLP layers to better learn the specific key-value mappings for the JSON schema.
