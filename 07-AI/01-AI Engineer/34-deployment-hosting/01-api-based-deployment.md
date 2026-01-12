## Summary
API-based deployment involves using managed services from providers like OpenAI, Anthropic, or cloud giants (AWS, Google, Azure) to access LLMs. This is the fastest way to add AI capabilities to an application without managing GPU infrastructure.

## Detailed Explanation
### **The Managed Service Landscape**
1.  **Direct Providers**:
    *   **OpenAI**: GPT-4o, GPT-4o-mini.
    *   **Anthropic**: Claude 3.5 Sonnet/Haiku.
2.  **Cloud Platforms (Hyperscalers)**:
    *   **AWS Bedrock**: Offers a variety of models (Claude, Llama, Mistral) in a secure, serverless environment.
    *   **Google Vertex AI**: Access to Gemini models and integration with the Google Cloud ecosystem.
    *   **Azure OpenAI**: Enterprise-grade OpenAI models with Azure's security and compliance.

### **Pros and Cons**
-   **Pros**:
    *   **Speed to Market**: Get started in minutes.
    *   **Scalability**: The provider handles millions of requests.
    *   **Cost (Initial)**: Pay-per-token model is cheaper for low to medium traffic.
-   **Cons**:
    *   **Privacy**: Data leaves your environment (unless using specific VPC/private link options).
    *   **Cost (Scale)**: High volume can become more expensive than self-hosting.
    *   **Vendor Lock-in**: Harder to switch models if you rely on provider-specific features.

### **Rate Limits and Latency**
When using APIs, you must implement robust **Error Handling** (e.g., exponential backoff) to handle Rate Limits (TPM/RPM) and monitor **TTFT** (Time to First Token) and **TPOT** (Time Per Output Token).

## Interview Questions
*   **Q: What is the difference between TPM and RPM in an LLM API?**
    *   **A:** **RPM** (Requests Per Minute) is the number of times you can call the API. **TPM** (Tokens Per Minute) is the total volume of text (input + output) processed. Most providers enforce both limits.
*   **Q: When should a company move from an API like OpenAI to a cloud provider like AWS Bedrock?**
    *   **A:** When they require enterprise-grade security (VPC isolation), regional compliance (keeping data in a specific country), or when they want to use their existing cloud billing and IAM (Identity and Access Management) systems.
*   **Q: How do you handle a "Rate Limit Exceeded" error in a production application?**
    *   **A:** Implement a retry mechanism with **Exponential Backoff** and **Jitter**. For critical applications, you might also implement a fallback to a different model or provider.
