# Long-Term Memory in Agentic AI Systems with Solace Agent Mesh (SAM) 

## What is Solace Agent Mesh (SAM)? 

Solace Agent Mesh (SAM) is an open-source agentic framework purpose-built to help you create collaborative AI systems that can interact with real-world data and systems in a flexible, event-driven way. 

Rather than being a monolithic AI platform, SAM is a composable, scalable integration layer for AI agents. It connects agents, data sources, and gateways so they can work together to solve complex tasks. 


## What is Memory in Agentic Frameworks? 

In traditional AI applications, memory is often short-lived, limited to a single session or context window. But agentic systems need more than that. They need to remember who they’re working with, what’s been said before, what goals have been set, and what preferences have been shared. 

This is where SAM’s long-term memory architecture shines. 

SAM introduces a memory model that stores information across sessions and agents, enabling adaptive behavior, continuity, and context preservation over time. 

### Why Memory Matters 

- Context Preservation: Conversations don’t start from scratch every time. 

- Adaptive Behavior: The system can adjust based on learned facts and instructions. 

- Efficient Collaboration: Instruction sets, and user information would not need to be repeated every time. 

- Human-Centric UX: Makes AI feel more intelligent, personal, and useful.


## SAM’s Memory Architecture 

SAM organizes information into four core types: 

- Learned Facts: Persistent knowledge about the user or environment (e.g., "User lives in Toronto"). 

- Learned Instructions: User-defined preferences, behaviors, and custom rules (e.g., "Always reply formally"). 

- Active Session History: The recent ongoing conversation context. 

- Summary of Old Session History: Compressed summaries of prior interactions to retain meaning without consuming memory. 

LLMs are used behind the scenes to automatically extract and organize this information. As sessions get longer, older data is summarized, prioritizing relevance, just like a human would do when remembering long conversations.


## How to Enable Memory in SAM Projects 

In the [gateway.yaml](./configs/gateways/rest_api/gateway.yaml) configuration file of your gateway, update the `history_policy` section with the following values: 

```yaml
- history_policy: &default_history_policy
    # ... Some other Configs ...
    enable_long_term_memory: true # Enables the long-term memory feature
    long_term_memory_config: # Required if enable_long_term_memory is set to true
      summary_time_to_live: 432000 # How long to keep the session summary before forgetting, default 5 Days in seconds
      llm_config: # LLM configuration to be used for the AI features of the long-term memory
        model: ${LLM_SERVICE_PLANNING_MODEL_NAME}
        api_key: ${LLM_SERVICE_API_KEY}
        base_url: ${LLM_SERVICE_ENDPOINT}
      store_config: # Configuration for storing long-term memory
        type: "file" # History Provider
        # Other configs required for the history provider
        path: /tmp/history  # Required for file history provider
```

You can use any of the built-in history providers or implement your own. The following history providers are available:
- In-memory (fast but ephemeral)
- File system
- Redis
- MongoDB
- SQL databases

```mermaid
graph TD
    %% User messages
    U1[User: Hi, I'm from Ottawa]
    U2[User: Can you always use CSS when replying?]
    U3[User: Thanks, that looks great!]
    U4[User: By the way, my favorite color is blue.]
    U5[User: ... longer chat continues ...]
    U6[User: What's the weather at my location?]

    %% System responses
    S1[System: Hello! Nice to meet someone from Ottawa!]
    S2[System: Got it – I'll use CSS in my replies.]
    S3[System: You're welcome!]
    S4[System: I'll remember that.]
    S6[System: Here's the weather in Ottawa...]

    %% Memory / extraction nodes
    F1[Extracted Fact → User is from Ottawa]
    P1[Stored Preference → User prefers reply with CSS format]
    F2[Extracted Fact → User favorite color is blue]
    SUM[Summarized Memory]
    R1[Used Fact: Location = Ottawa]

    %% Flow connections
    U1 --> S1
    S1 --> F1
    U2 --> S2
    S2 --> P1
    U3 --> S3
    U4 --> S4
    S4 --> F2
    U5 --> SUM
    U6 --> S6
    R1 --> S6

    S1 --> U2
    S2 --> U3
    S3 --> U4
    S4 --> U5
    U5 --> U6

    %% Styles
    classDef memory fill:#ffeb3b,stroke:#333,stroke-width:2px,color:#000;
    classDef user fill:#db5400,stroke:#333,stroke-width:2px,color:#000;
    classDef system fill:#ba68c8,stroke:#333,stroke-width:2px,color:#fff;

    class U1,U2,U3,U4,U5,U6 user;
    class S1,S2,S3,S4,S6 system;
    class F1,P1,F2,SUM,R1 memory;


```
