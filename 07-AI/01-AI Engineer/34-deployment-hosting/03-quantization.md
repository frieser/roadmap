## Summary
Quantization is the process of reducing the precision of a model's weights (e.g., from 16-bit to 4-bit) to reduce its memory footprint and increase inference speed, with minimal loss in accuracy.

## Detailed Explanation
### **Common Quantization Formats**
1.  **GGUF (formerly GGML)**: Optimized for CPU and Apple Silicon (Macs). It allows "offloading" some layers to the GPU.
2.  **AWQ (Activation-aware Weight Quantization)**: Maintains higher accuracy by keeping important weights in higher precision. Very common for vLLM/TGI production serving.
3.  **EXL2**: Specifically optimized for extremely fast inference on consumer NVIDIA GPUs.
4.  **FP8 / INT8 / INT4**: Standard precision levels. Moving from FP16 to INT4 reduces memory by 4x.

### **The Trade-off: Perplexity vs. Size**
As you decrease precision (e.g., from 8-bit to 2-bit), the model's **Perplexity** (a measure of uncertainty) increases.
-   **4-bit**: Usually considered the "sweet spot" where the model is much smaller but still performs nearly as well as the 16-bit version.
-   **2-bit**: Significant degradation in reasoning and coherence.

### **Calculation Example**
A 7B parameter model in FP16 (16-bit) takes:
$7 \times 10^9 \times 2 \text{ bytes} \approx 14 \text{ GB}$.
The same model in 4-bit quantization takes:
$7 \times 10^9 \times 0.5 \text{ bytes} \approx 3.5 \text{ GB}$.
This allows the model to fit on a cheap 8GB GPU instead of a more expensive one.

## Interview Questions
*   **Q: What is the primary benefit of quantization?**
    *   **A:** Reducing the VRAM (GPU memory) required to load and run a model. This allows larger models to run on smaller, cheaper hardware and increases the speed of token generation.
*   **Q: Does quantization affect the accuracy of the model?**
    *   **A:** Yes, it usually results in a slight decrease in performance (higher perplexity). However, for many tasks, the difference between 16-bit and 4-bit is negligible compared to the massive memory savings.
*   **Q: Which quantization format would you choose for running a model on a MacBook?**
    *   **A:** **GGUF**. It is specifically designed to work with llama.cpp and is highly optimized for Apple's Metal API and Unified Memory architecture.
