
```mermaid
    A[Quality Standards and Objectives] --> B[Testing Strategy]
    B --> C[Quality Assurance (QA) Processes]
    B --> D[Quality Control (QC) Processes]
    C --> E[CI/CD Pipeline]
    D --> E
    E --> F[Quality Metrics and Monitoring]
    F --> G[Feedback Loops]
    G --> A

    E --> H[Tools and Automation Frameworks]
    F --> I[Roles and Responsibilities]
    G --> J[Documentation and Training Programs]
    J --> A

    subgraph Quality Culture
        A
        B
        C
        D
        E
        F
        G
        H
        I
        J
    end
``` 
