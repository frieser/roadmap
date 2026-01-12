## Summary
LoRA (Low-Rank Adaptation) and QLoRA are Parameter-Efficient Fine-Tuning (PEFT) techniques that allow training large models on consumer hardware by only updating a tiny fraction of the model's weights.

## Detailed Explanation
### **LoRA: Low-Rank Adaptation**
Instead of updating the massive weight matrices ($W$) of a transformer, LoRA adds two smaller, low-rank matrices ($A$ and $B$) alongside them.
-   During training: Only $A$ and $B$ are updated. $W$ remains frozen.
-   **Rank ($r$)**: A hyperparameter that determines the size of the LoRA matrices. A higher rank captures more detail but requires more memory.
-   **Alpha ($\alpha$)**: A scaling factor for the LoRA weights.

### **QLoRA: Quantized LoRA**
QLoRA takes LoRA a step further by quantizing the frozen base model weights to 4-bit precision (using the **NF4** data type).
-   **Double Quantization**: Further reduces memory by quantizing the quantization constants themselves.
-   **Paged Optimizers**: Prevents "Out of Memory" (OOM) errors by using CPU RAM as a buffer during spikes in GPU memory usage.

### **Benefits of LoRA/QLoRA**
1.  **Hardware**: Can fine-tune a 70B parameter model on a single 80GB GPU (or even a 7B model on a 12GB home GPU).
2.  **Storage**: The resulting "Adapters" (the $A$ and $B$ matrices) are only a few MBs, compared to many GBs for a full model.
3.  **Portability**: You can swap adapters in and out of a single frozen base model at runtime.

## Interview Questions
*   **Q: How does LoRA reduce the number of trainable parameters?**
    *   **A:** By decomposing a large weight matrix update into two smaller, low-rank matrices. For a matrix of size $d \times d$, a full update has $d^2$ parameters, while LoRA has $2 \times d \times r$ parameters. Since $r$ (the rank) is usually very small (e.g., 8 or 16), the reduction is massive (often >99%).
*   **Q: What is the difference between LoRA and QLoRA?**
    *   **A:** LoRA updates small matrices while keeping the base model frozen. QLoRA does the same but also quantizes the frozen base model to 4-bit, significantly reducing the VRAM required to load the model during training.
*   **Q: What hyperparameter should you tune if the model is not learning the specific style of your dataset?**
    *   **A:** You should try increasing the **Rank ($r$)** or adjusting the **Alpha** scaling factor to allow more capacity for the adapter to capture the nuances of the data.
