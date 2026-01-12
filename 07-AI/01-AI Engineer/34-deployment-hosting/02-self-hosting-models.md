## Summary
Self-hosting involves running Open-Weight models (like Llama 3, Mistral, or DeepSeek) on your own infrastructure or private cloud. This provides maximum control over data privacy, cost, and customization.

## Detailed Explanation
### **Inference Engines (The Runtime)**
To serve models efficiently, you need specialized software that optimizes GPU usage:
1.  **vLLM**: The industry leader for high throughput. It uses **PagedAttention** to manage KV cache memory, drastically reducing memory waste.
2.  **TGI (Text Generation Inference)**: Developed by Hugging Face. Highly optimized for production with features like continuous batching and token streaming.
3.  **Ollama**: Focused on local development. It simplifies model management and provides a local API.
4.  **LocalAI**: A self-hosted, community-driven OpenAI-compatible API.

### **Infrastructure Components**
-   **Compute**: NVIDIA GPUs (A100, H100, L40S) or specialized chips (TPUs, Groq LPUs).
-   **Orchestration**: **Kubernetes** with the **Kueue** or **Kaito** operator for GPU scheduling.
-   **Storage**: Fast SSDs or network storage (like S3) to load model weights (often 10GB to 150GB+).

### **Serving Optimizations**
-   **Continuous Batching**: Processing multiple requests simultaneously by inserting new requests as soon as an existing one finishes a token.
-   **KV Cache**: Storing the intermediate states of the transformer to avoid re-calculating everything for every new token.

## Interview Questions
*   **Q: What is PagedAttention and why is it important in vLLM?**
    *   **A:** PagedAttention is a memory management technique that partitions the Key-Value (KV) cache into fixed-size pages, similar to virtual memory in operating systems. It eliminates fragmentation and allows vLLM to serve more requests simultaneously with the same amount of VRAM.
*   **Q: What are the main challenges of self-hosting a 70B parameter model?**
    *   **A:** The primary challenges are **Hardware Cost** (requires multiple A100/H100 GPUs), **Latency** (balancing throughput vs. response time), and **Operational Complexity** (managing GPU drivers, scaling, and model loading).
*   **Q: Why use Docker/Kubernetes for LLM deployment?**
    *   **A:** For reproducibility, horizontal scaling, and efficient resource allocation. Kubernetes allows you to schedule workloads across a cluster of GPUs and automatically restart inference servers if they crash.
