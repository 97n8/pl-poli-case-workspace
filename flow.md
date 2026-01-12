%%{init: {
  "theme": "base",
  "themeVariables": {
    "fontFamily": "Inter, Segoe UI, Arial",
    "fontSize": "14px",
    "primaryColor": "#f8fafc",
    "primaryTextColor": "#0f172a",
    "primaryBorderColor": "#334155",
    "lineColor": "#334155"
  }
}}%%

flowchart LR
    A[Authority Declared]
    B[CASE & Record Established]
    C[Governed Flow Executed]
    D[Time Enforced]
    E[Decision Recorded]
    F[Output Generated]
    G[Archival & Retention]
    H[Governance Review & Authority Update]

    A --> B --> C --> D --> E --> F --> G --> H --> A

    %% Semantic Classes
    class A authority
    class B case
    class C flow
    class D time
    class E decision
    class F output
    class G archive
    class H review

    %% Class Styling (GitHub Safe)
    classDef authority fill:#ede9fe,stroke:#6b21a8,stroke-width:2px
    classDef case fill:#e0f2fe,stroke:#0369a1,stroke-width:2px
    classDef flow fill:#dcfce7,stroke:#047857,stroke-width:2px
    classDef time fill:#ffedd5,stroke:#c2410c,stroke-width:2px
    classDef decision fill:#fee2e2,stroke:#b91c1c,stroke-width:2px
    classDef output fill:#e0f2fe,stroke:#075985,stroke-width:2px
    classDef archive fill:#f1f5f9,stroke:#334155,stroke-width:2px
    classDef review fill:#fae8ff,stroke:#7e22ce,stroke-width:2px
