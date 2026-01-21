# Life Decision Coach (LDC) - Architecture Flowcharts

> [!NOTE]
> This document contains comprehensive flowcharts for the entire LDC system architecture. Each layer is visualized with detailed component interactions and data flows.

---

## 1. Complete System Architecture Overview

```mermaid
graph TB
    subgraph "User Interface"
        UI[User Input Interface]
        VIZ[Trajectory Visualizer]
        EXP[Explanation Display]
    end
    
    subgraph "Layer 1: Life State Ingestion"
        FORM[Structured Questionnaire]
        DOC[Document Parser]
        VAL[Validator & Normalizer]
        LSV[Life State Vector]
    end
    
    subgraph "Layer 2: Reality Constraint Graph"
        KG1[Constraint Knowledge Graph]
        CRED[Credential Nodes]
        LEGAL[Legal/Visa Nodes]
        TIME[Time Window Nodes]
        FEAS[Feasibility Checker]
    end
    
    subgraph "Layer 3: Trajectory Intelligence"
        subgraph "3A: Trajectory KG"
            TKG[Trajectory Knowledge Graph]
            STATES[Career State Nodes]
            TRANS[Transition Edges]
        end
        
        subgraph "3B: Empirical Models"
            SURV[Survival Analysis]
            COX[Cox Proportional Hazards]
            BAYES[Bayesian Transition Models]
        end
        
        subgraph "3C: RAG Context"
            RAG[RAG Retrieval]
            VDB[Vector Database]
            MOD[Context Modifiers]
        end
        
        subgraph "3D: Outcome Envelopes"
            QR[Quantile Regression]
            P10[P10 Trajectory]
            P50[P50 Trajectory]
            P90[P90 Trajectory]
            PARETO[Pareto Frontier]
        end
    end
    
    subgraph "Layer 4: Risk Analysis"
        LOCK[Lock-In Detector]
        REG[Regret Computer]
        REV[Reversibility Scorer]
        TDECAY[Time-Decay Analysis]
    end
    
    subgraph "Layer 5: Explanation"
        LLM[LLM Explanation Engine]
        PROMPT[Constrained Prompts]
        PLAIN[Plain English Output]
    end
    
    subgraph "Data Sources"
        PSID[(PSID Survey)]
        H1B[(H1B Database)]
        LINKEDIN[(LinkedIn Data)]
        POLICY[(Policy APIs)]
    end
    
    UI --> FORM
    UI --> DOC
    FORM --> VAL
    DOC --> VAL
    VAL --> LSV
    
    LSV --> FEAS
    LSV --> TKG
    
    CRED --> FEAS
    LEGAL --> FEAS
    TIME --> FEAS
    KG1 --> FEAS
    
    FEAS -->|Feasible Paths| TKG
    
    STATES --> SURV
    TRANS --> SURV
    TKG --> COX
    TKG --> BAYES
    
    POLICY --> RAG
    RAG --> VDB
    VDB --> MOD
    
    SURV --> QR
    COX --> QR
    BAYES --> QR
    MOD --> QR
    
    QR --> P10
    QR --> P50
    QR --> P90
    QR --> PARETO
    
    P10 --> LOCK
    P50 --> LOCK
    P90 --> LOCK
    
    LOCK --> REG
    PARETO --> REG
    LSV --> REV
    
    REG --> TDECAY
    REV --> TDECAY
    
    P10 --> LLM
    P50 --> LLM
    P90 --> LLM
    LOCK --> LLM
    REG --> LLM
    TDECAY --> LLM
    
    PROMPT --> LLM
    LLM --> PLAIN
    
    P10 --> VIZ
    P50 --> VIZ
    P90 --> VIZ
    PARETO --> VIZ
    PLAIN --> EXP
    
    PSID --> SURV
    H1B --> BAYES
    LINKEDIN --> TKG
    
    style LSV fill:#4A90E2
    style FEAS fill:#E24A4A
    style QR fill:#50C878
    style LLM fill:#FFB84D
    style VIZ fill:#9B59B6
```

---

## 2. Layer 1: Life State Ingestion - Detailed Flow

```mermaid
graph LR
    subgraph "Input Sources"
        Q[Structured Questionnaire]
        RESUME[Resume PDF/DOCX]
        TRANS[Transcript]
        FIN[Financial Docs]
    end
    
    subgraph "Processing"
        NER[Named Entity Recognition]
        PARSE[Document Parser]
        SKILL[Skill Taxonomy Mapper]
        GEO[Geographic Normalizer]
        DATE[Date Validator]
    end
    
    subgraph "Validation"
        SCHEMA[Schema Validator]
        RANGE[Range Checker]
        CONS[Consistency Checker]
    end
    
    subgraph "Output"
        LSV["Life State Vector (JSON)"]
    end
    
    Q -->|Demographics| SCHEMA
    RESUME --> NER
    TRANS --> PARSE
    FIN --> PARSE
    
    NER --> SKILL
    NER --> GEO
    PARSE --> DATE
    
    SKILL --> SCHEMA
    GEO --> SCHEMA
    DATE --> SCHEMA
    
    SCHEMA --> RANGE
    RANGE --> CONS
    CONS --> LSV
    
    LSV --> |"{ age: 28, location: 'Mumbai', degree: 'B.Tech CS', ... }"| OUTPUT[Next Layer]
    
    style LSV fill:#4A90E2,stroke:#333,stroke-width:3px
```

**Life State Vector Schema:**

```json
{
  "demographics": {
    "age": 28,
    "location": "Mumbai, India",
    "citizenship": ["India"],
    "visa_status": {
      "US": {"type": "F1", "expiry": "2027-06-30"}
    }
  },
  "education": [
    {
      "degree": "B.Tech",
      "field": "Computer Science",
      "institution": "IIT Delhi",
      "gpa": 3.7,
      "graduation_year": 2020
    }
  ],
  "career": {
    "current_role": "Software Engineer L3",
    "company": "TCS",
    "industry": "IT Services",
    "years_experience": 3,
    "skills": ["Python", "React", "AWS"],
    "certifications": ["AWS Solutions Architect"]
  },
  "financial": {
    "savings_usd": 15000,
    "debt_usd": 0,
    "annual_income_usd": 25000,
    "equity": 0
  },
  "constraints": {
    "dependents": 0,
    "health_issues": [],
    "geographic_restrictions": [],
    "risk_tolerance": 0.7
  }
}
```

---

## 3. Layer 2: Reality Constraint Graph - Structure & Checking

```mermaid
graph TB
    subgraph "Constraint Knowledge Graph Schema"
        subgraph "Credential Nodes"
            BACH["Bachelor's Degree"]
            MAST["Master's Degree"]
            PHD["PhD"]
            CERT["Professional Certification"]
        end
        
        subgraph "Visa/Legal Nodes"
            H1B["H1B Visa (US)"]
            F1["F1 Student Visa (US)"]
            GC["Green Card (US)"]
            EE["Express Entry (Canada)"]
            BLUECARD["EU Blue Card"]
        end
        
        subgraph "Age Window Nodes"
            AGE18_30["Age 18-30 Window"]
            AGE_40["Age < 40 Window"]
            AGE_35["Age < 35 Window"]
        end
        
        subgraph "Financial Nodes"
            MIN50K["Minimum $50k"]
            MIN800K["Minimum $800k"]
        end
    end
    
    BACH -->|prerequisite| H1B
    BACH -->|prerequisite| F1
    H1B -->|enables| GC
    F1 -->|enables| H1B
    
    MAST -->|prerequisite| PHD
    MAST -->|boosts| EE
    
    AGE_35 -->|required_for| EE
    AGE18_30 -->|preferred_for| F1
    AGE_40 -->|required_for| BLUECARD
    
    MIN50K -->|prerequisite| MAST
    MIN800K -->|prerequisite| EB5["EB-5 Investor Visa"]
    
    BACH -->|prerequisite| MAST
    
    style H1B fill:#E24A4A
    style GC fill:#50C878
    style MIN800K fill:#FFB84D
```

**Feasibility Checking Algorithm:**

```mermaid
graph LR
    START[User Life State] --> CHECK1{Has Prerequisites?}
    CHECK1 -->|No| BLOCK1[Block Path]
    CHECK1 -->|Yes| CHECK2{Within Age Window?}
    CHECK2 -->|No| BLOCK2[Block Path]
    CHECK2 -->|Yes| CHECK3{Meets Financial Threshold?}
    CHECK3 -->|No| BLOCK3[Block Path]
    CHECK3 -->|Yes| CHECK4{No Exclusions?}
    CHECK4 -->|Conflict| BLOCK4[Block Path]
    CHECK4 -->|Clear| PASS[Mark as Feasible]
    
    BLOCK1 --> OUTPUT[Constrained State Space]
    BLOCK2 --> OUTPUT
    BLOCK3 --> OUTPUT
    BLOCK4 --> OUTPUT
    PASS --> OUTPUT
    
    style PASS fill:#50C878
    style BLOCK1 fill:#E24A4A
    style BLOCK2 fill:#E24A4A
    style BLOCK3 fill:#E24A4A
    style BLOCK4 fill:#E24A4A
```

---

## 4. Layer 3A: Trajectory Knowledge Graph

```mermaid
graph TB
    subgraph "Career State Nodes (Software Engineering)"
        INTERN["SWE Intern - India - 8k/yr"]
        L3_IN["SWE L3 - Startup India - 15k/yr"]
        L3_BT_IN["SWE L3 - BigTech India - 25k/yr"]
        L4_US["SWE L4 - BigTech US - 150k/yr"]
        L5_IN["SWE L5 - BigTech India - 40k/yr"]
        L5_US["SWE L5 - BigTech US - 220k/yr"]
        STARTUP["Startup Founder - India - Variable"]
    end
    
    INTERN -->|"85% / 1.5yr"| L3_IN
    INTERN -->|"45% / 2.0yr"| L3_BT_IN
    
    L3_IN -->|"22% / 2.0yr"| L3_BT_IN
    L3_IN -->|"18% / 2.3yr"| L4_US
    L3_IN -->|"12% / 1.5yr"| STARTUP
    
    L3_BT_IN -->|"31% / 1.8yr"| L4_US
    L3_BT_IN -->|"25% / 2.5yr"| L5_IN
    
    L4_US -->|"42% / 2.2yr"| L5_US
    L5_IN -->|"12% / 3.0yr"| L4_US
    
    style L4_US fill:#50C878
    style L5_US fill:#4A90E2
    style STARTUP fill:#FFB84D
```

**Edge Metadata Example:**

```json
{
  "from": "SWE_L3_Startup_India",
  "to": "SWE_L4_BigTech_US",
  "observations": 237,
  "success_rate": 0.18,
  "time_to_transition": {
    "P10": 1.2,
    "P50": 2.3,
    "P90": 4.5
  },
  "typical_requirements": [
    "Apply to FAANG",
    "Leetcode preparation",
    "H1B sponsorship",
    "Relocate to US"
  ],
  "financial_cost": {
    "median": 5000,
    "range": [2000, 15000]
  }
}
```

---

## 5. Layer 3B: Empirical Transition Models - Analysis Pipeline

```mermaid
graph LR
    subgraph "Input Data"
        LONG[("Longitudinal Survey Data")]
        JOB[("Job Market Data")]
        VISA[(Visa Records)]
    end
    
    subgraph "Survival Analysis"
        KM[Kaplan-Meier Estimator]
        COX[Cox Proportional Hazards]
        AFT[Accelerated Failure Time Models]
    end
    
    subgraph "Bayesian Inference"
        PRIOR[Historical Priors]
        LIKE[Likelihood Function]
        POST[Posterior Distribution]
        MCMC[MCMC Sampling]
    end
    
    subgraph "Output"
        TPROB["Transition Probabilities with CI"]
        TTIME["Time to Transition P10/P50/P90"]
        CONTEXT["Context Modifiers"]
    end
    
    LONG --> KM
    JOB --> COX
    VISA --> AFT
    
    KM --> TTIME
    COX --> CONTEXT
    AFT --> TTIME
    
    LONG --> PRIOR
    PRIOR --> LIKE
    JOB --> LIKE
    LIKE --> POST
    POST --> MCMC
    MCMC --> TPROB
    
    style TPROB fill:#50C878
    style TTIME fill:#4A90E2
```

**Survival Analysis Output Example:**

```mermaid
%%{init: {'theme':'base'}}%%
graph LR
    subgraph "Time to H1B Approval"
        T0["t=0 | Applied | P=1.0"]
        T1["t=6mo | P=0.85 | 15% rejected"]
        T2["t=12mo | P=0.72 | 28% rejected"]
        T3["t=18mo | P=0.68 | 32% rejected"]
        T4["t=24mo | P=0.65 | 35% rejected"]
    end
    
    T0 --> T1 --> T2 --> T3 --> T4
    
    style T1 fill:#FFE5B4
    style T2 fill:#FFCC99
    style T3 fill:#FFB84D
    style T4 fill:#FF9933
```

---

## 6. Layer 3C: RAG-Augmented Context Modulation

```mermaid
sequenceDiagram
    participant User
    participant System
    participant VectorDB
    participant PolicyAPI
    participant LLM
    participant TransitionModel
    
    User->>System: Query: "SWE India → SWE US in 2026"
    
    System->>VectorDB: Search: "H1B visa 2026, tech layoffs, immigration policy"
    VectorDB-->>System: Top 5 relevant documents
    
    System->>PolicyAPI: Fetch: USCIS approval rates (live)
    PolicyAPI-->>System: Current approval rate: 76% (vs 82% historical)
    
    System->>LLM: Extract structured modifiers from documents
    Note over LLM: Constrained prompt to extract modifiers only
    
    LLM-->>System: {visa_difficulty: 1.3, job_availability: 0.8, delay_months: +6}
    
    System->>TransitionModel: Apply modifiers to base model
    Note over TransitionModel: p_base = 0.18, p_adjusted = 0.11
    
    TransitionModel-->>System: Adjusted predictions
    System-->>User: Display: "11% success (vs 18% historical) due to current visa climate"
```

**RAG Modifier Application:**

```mermaid
graph TB
    BASE["Base Model: p=0.18, t=2.3yr"]
    
    MOD1["Visa Difficulty: ×1/1.3"]
    MOD2["Job Market: ×0.8"]
    MOD3["Time Delay: +6 months"]
    
    CONF["Confidence Check: > 0.6?"]
    
    ADJ["Adjusted: p=0.11, t=2.8yr"]
    ORIG["Original: p=0.18, t=2.3yr"]
    
    BASE --> MOD1
    MOD1 --> MOD2
    MOD2 --> MOD3
    MOD3 --> CONF
    
    CONF -->|Yes| ADJ
    CONF -->|No| ORIG
    
    style ADJ fill:#50C878
    style ORIG fill:#4A90E2
```

---

## 7. Layer 3D: Deterministic Outcome Envelopes

```mermaid
graph TB
    subgraph "Quantile Regression Models"
        MODEL10["P10 Model - Pessimistic"]
        MODEL50["P50 Model - Median"]
        MODEL90["P90 Model - Optimistic"]
    end
    
    subgraph "Input Features"
        FEAT["User Life State + Decision Context"]
    end
    
    subgraph "P10 Trajectory (Worst Case)"
        P10_PATH["Path: Visa rejected, return to India"]
        P10_NET["Net Worth Year 5: $45,000"]
        P10_ASSUME["Assumptions: No H1B, salary growth 2%"]
    end
    
    subgraph "P50 Trajectory (Median)"
        P50_PATH["Path: H1B approved, L4 SWE US"]
        P50_NET["Net Worth Year 5: $180,000"]
        P50_ASSUME["Assumptions: H1B in 2yr, promotion year 4"]
    end
    
    subgraph "P90 Trajectory (Best Case)"
        P90_PATH["Path: FAANG L5, Green Card"]
        P90_NET["Net Worth Year 5: $420,000"]
        P90_ASSUME["Assumptions: Fast H1B, L5 in year 3, stock growth 20%"]
    end
    
    FEAT --> MODEL10
    FEAT --> MODEL50
    FEAT --> MODEL90
    
    MODEL10 --> P10_PATH
    P10_PATH --> P10_NET
    P10_NET --> P10_ASSUME
    
    MODEL50 --> P50_PATH
    P50_PATH --> P50_NET
    P50_NET --> P50_ASSUME
    
    MODEL90 --> P90_PATH
    P90_PATH --> P90_NET
    P90_NET --> P90_ASSUME
    
    style MODEL10 fill:#E24A4A
    style MODEL50 fill:#FFB84D
    style MODEL90 fill:#50C878
```

**Pareto Frontier for Multi-Objective Decisions:**

```mermaid
graph LR
    subgraph "Decision Options"
        A["Option A: Startup | Income:Low, Balance:Poor, Stability:Low"]
        B["Option B: BigTech | Income:High, Balance:Good, Stability:High"]
        C["Option C: Govt Job | Income:Med, Balance:Excellent, Stability:Highest"]
    end
    
    subgraph "Dominance Analysis"
        DOM["B dominates A on all dimensions"]
        NDOM["B and C are non-dominated, trade-offs exist"]
    end
    
    A -->|Dominated| DOM
    B --> NDOM
    C --> NDOM
    
    DOM --> OUTPUT["Show only: B and C"]
    NDOM --> OUTPUT
    
    style B fill:#50C878
    style C fill:#4A90E2
    style A fill:#E24A4A
```

---

## 8. Layer 4: Risk, Regret & Lock-In Analysis

```mermaid
graph TB
    subgraph "Lock-In Detection"
        CURR[Current State]
        DEC[Decision Under Consideration]
        NEW[New State After Decision]
        
        REACH_BEFORE[Reachable States Before]
        REACH_AFTER[Reachable States After]
        
        LOCKED[Locked Out States]
    end
    
    subgraph "Regret Computation"
        PATH_A["Path A: Take Decision"]
        PATH_B["Path B: Decline Decision"]
        
        OUTCOME_A["Outcome A - P10/P50/P90"]
        OUTCOME_B["Outcome B - P10/P50/P90"]
        
        REGRET["Regret = Outcome_B - Outcome_A for each percentile"]
    end
    
    subgraph "Reversibility Score"
        TIME_COST["Time Cost to Reverse"]
        FIN_COST["Financial Cost to Reverse"]
        REP_COST[Reputation Cost]
        LEGAL_COST[Legal Barriers]
        
        REV_SCORE["Reversibility: 0-1 scale"]
    end
    
    CURR --> DEC
    DEC --> NEW
    
    CURR --> REACH_BEFORE
    NEW --> REACH_AFTER
    
    REACH_BEFORE --> LOCKED
    REACH_AFTER --> LOCKED
    
    DEC --> PATH_A
    DEC --> PATH_B
    
    PATH_A --> OUTCOME_A
    PATH_B --> OUTCOME_B
    
    OUTCOME_A --> REGRET
    OUTCOME_B --> REGRET
    
    DEC --> TIME_COST
    DEC --> FIN_COST
    DEC --> REP_COST
    DEC --> LEGAL_COST
    
    TIME_COST --> REV_SCORE
    FIN_COST --> REV_SCORE
    REP_COST --> REV_SCORE
    LEGAL_COST --> REV_SCORE
    
    LOCKED --> OUTPUT[Risk Analysis Output]
    REGRET --> OUTPUT
    REV_SCORE --> OUTPUT
    
    style LOCKED fill:#E24A4A
    style REGRET fill:#FFB84D
    style REV_SCORE fill:#4A90E2
```

**Time-Decay Regret Analysis:**

```mermaid
%%{init: {'theme':'base'}}%%
graph LR
    subgraph "PhD Program - Regret if Quit by Year"
        Y1["Year 1 | Sunk:20k | Remaining:150k | Regret:LOW"]
        Y2["Year 2 | Sunk:45k | Remaining:125k | Regret:MED"]
        Y3["Year 3 | Sunk:75k | Remaining:95k | Regret:HIGH"]
        Y4["Year 4 | Sunk:110k | Remaining:60k | Regret:V-HIGH"]
        Y5["Year 5 | Sunk:140k | Remaining:30k | Regret:MED"]
        DONE["Complete | Sunk:170k | Gained:170k | Regret:NONE"]
    end
    
    Y1 --> Y2 --> Y3 --> Y4 --> Y5 --> DONE
    
    style Y1 fill:#50C878
    style Y2 fill:#FFE5B4
    style Y3 fill:#FFB84D
    style Y4 fill:#E24A4A
    style Y5 fill:#FFB84D
    style DONE fill:#4A90E2
```

---

## 9. Layer 5: LLM Explanation Pipeline

```mermaid
sequenceDiagram
    participant Layer4 as Layer 4 Output
    participant Formatter
    participant Prompt
    participant LLM
    participant Validator
    participant User
    
    Layer4->>Formatter: {P10/P50/P90, lock-in, regret, reversibility}
    Formatter->>Formatter: Convert numbers to structured JSON
    
    Formatter->>Prompt: Inject data into constrained template
    Note over Prompt: Template rules - No recs, No opinions, Only translate
    
    Prompt->>LLM: Send constrained prompt
    LLM->>LLM: Generate explanation
    
    LLM->>Validator: Plain English output
    
    Validator->>Validator: Check for violations
    
    alt Violates constraints
        Validator->>Prompt: Regenerate with stricter prompt
    else Passes validation
        Validator->>User: Display explanation
    end
```

**Constrained Prompt Template:**

```
SYSTEM:
You are an explanation engine for a life decision analysis system.
You translate numerical predictions into plain English.

STRICT RULES:
1. DO NOT recommend actions ("you should do X")
2. DO NOT add your own analysis
3. DO NOT generate new probabilities
4. DO NOT use "I think" or "in my opinion"
5. ONLY explain the numbers provided

INPUT DATA:
{
  "P10": {"net_worth": 45000, "path": "Visa rejected"},
  "P50": {"net_worth": 180000, "path": "H1B approved, L4 SWE"},
  "P90": {"net_worth": 420000, "path": "FAANG L5, Green Card"},
  "lock_in": ["Indian startup founder", "early retirement"],
  "regret_if_decline": {"P50": 120000, "P90": 350000},
  "reversibility": 0.3
}

OUTPUT FORMAT:
- Use headers and bullet points
- Explain probabilities as "1 in 10 chance" not "10%"
- Cite data sources
```

---

## 10. Complete Data Flow: End-to-End

```mermaid
flowchart TD
    START([User: 28yr SWE, considering MS in US])
    
    L1_IN[Layer 1: Ingest structured data]
    L1_OUT[Life State Vector]
    
    L2_CHECK{Layer 2: Check constraints}
    L2_BLOCK[Block impossible paths]
    L2_PASS[Feasible paths identified]
    
    L3A_GRAPH[Layer 3A: Load trajectory graph]
    L3B_MODEL[Layer 3B: Apply empirical models]
    L3C_RAG[Layer 3C: Fetch current context via RAG]
    L3D_ENVELOPE[Layer 3D: Compute P10/P50/P90]
    
    L4_LOCK[Layer 4: Detect lock-ins]
    L4_REGRET[Layer 4: Compute regret]
    L4_REV[Layer 4: Score reversibility]
    
    L5_LLM[Layer 5: LLM explains to user]
    
    OUTPUT([User sees: 3 trajectories, lock-ins, regret, plain English])
    
    START --> L1_IN
    L1_IN --> L1_OUT
    L1_OUT --> L2_CHECK
    
    L2_CHECK -->|Infeasible| L2_BLOCK
    L2_CHECK -->|Feasible| L2_PASS
    
    L2_BLOCK --> OUTPUT
    L2_PASS --> L3A_GRAPH
    
    L3A_GRAPH --> L3B_MODEL
    L3B_MODEL --> L3C_RAG
    L3C_RAG --> L3D_ENVELOPE
    
    L3D_ENVELOPE --> L4_LOCK
    L3D_ENVELOPE --> L4_REGRET
    L3D_ENVELOPE --> L4_REV
    
    L4_LOCK --> L5_LLM
    L4_REGRET --> L5_LLM
    L4_REV --> L5_LLM
    
    L5_LLM --> OUTPUT
    
    style L1_OUT fill:#4A90E2
    style L2_PASS fill:#50C878
    style L2_BLOCK fill:#E24A4A
    style L3D_ENVELOPE fill:#9B59B6
    style L5_LLM fill:#FFB84D
```

---

## 11. MVP Hackathon Architecture (3-Month Build)

```mermaid
graph TB
    subgraph "Frontend (React + D3.js)"
        UI_FORM["Structured Form: Age, Degree, Savings"]
        UI_VIZ["Trajectory Visualizer: Interactive D3 Graph"]
        UI_EXP["Explanation Panel: LLM Output"]
    end
    
    subgraph "Backend (FastAPI)"
        API[REST API Endpoints]
        VAL[Input Validator]
        PROC[Processing Engine]
    end
    
    subgraph "Data Layer"
        NEO[("Neo4j - Constraint Graph")]
        POSTGRES[("PostgreSQL - User States")]
        SIMDATA[("Simulated Trajectory Data")]
    end
    
    subgraph "ML Pipeline (scikit-learn + lifelines)"
        QR_MODEL["Quantile Regression: P10/P50/P90"]
        SURV_MODEL["Survival Analysis: Time-to-Transition"]
    end
    
    subgraph "LLM Layer (OpenAI API)"
        GPT4[GPT-4 API]
        EXPL[Explanation Generator]
    end
    
    UI_FORM --> API
    API --> VAL
    VAL --> PROC
    
    PROC --> NEO
    PROC --> SIMDATA
    
    NEO --> QR_MODEL
    SIMDATA --> QR_MODEL
    SIMDATA --> SURV_MODEL
    
    QR_MODEL --> PROC
    SURV_MODEL --> PROC
    
    PROC --> GPT4
    GPT4 --> EXPL
    
    EXPL --> UI_EXP
    QR_MODEL --> UI_VIZ
    
    PROC --> POSTGRES
    
    style UI_VIZ fill:#9B59B6
    style NEO fill:#E24A4A
    style GPT4 fill:#FFB84D
```

---

## 12. Production System Architecture (2-Year Build)

```mermaid
graph TB
    subgraph "Frontend (Next.js)"
        WEB[Web App]
        MOBILE[Mobile App]
        EMBED[Embeddable Widget]
    end
    
    subgraph "API Gateway"
        GATEWAY[Kong API Gateway]
        AUTH[Auth Service]
        RATE[Rate Limiter]
    end
    
    subgraph "Microservices (Kubernetes)"
        INGEST["Ingestion Service - Python"]
        CONSTRAINT["Constraint Service - Rust"]
        TRAJECTORY["Trajectory Service - Python"]
        RAG_SVC["RAG Service - Python"]
        RISK["Risk Service - Python"]
        EXPLAIN["Explanation Service - Python"]
    end
    
    subgraph "Data Storage"
        NEO[("Neo4j - Constraint KG")]
        POSTGRES[("PostgreSQL - User Data")]
        PINECONE[("Pinecone - Vector DB")]
        S3[("S3 - ML Models")]
    end
    
    subgraph "ML Training Pipeline"
        SPARK[Spark ETL]
        TRAIN["Model Training: PyTorch + Stan"]
        VERSIONING["Model Versioning: MLflow"]
    end
    
    subgraph "External Data"
        PSID[(PSID Survey)]
        H1B_API[(USCIS API)]
        LINKEDIN_API[(LinkedIn API)]
        POLICY_API[(Policy APIs)]
    end
    
    subgraph "Monitoring"
        DATADOG[Datadog]
        ARIZE[Arize ML Observability]
    end
    
    WEB --> GATEWAY
    MOBILE --> GATEWAY
    EMBED --> GATEWAY
    
    GATEWAY --> AUTH
    GATEWAY --> RATE
    
    AUTH --> INGEST
    AUTH --> CONSTRAINT
    AUTH --> TRAJECTORY
    
    INGEST --> POSTGRES
    CONSTRAINT --> NEO
    
    TRAJECTORY --> RAG_SVC
    RAG_SVC --> PINECONE
    
    TRAJECTORY --> RISK
    RISK --> EXPLAIN
    
    EXPLAIN --> WEB
    
    SPARK --> PSID
    SPARK --> H1B_API
    SPARK --> LINKEDIN_API
    
    SPARK --> TRAIN
    TRAIN --> VERSIONING
    VERSIONING --> S3
    
    TRAJECTORY --> S3
    
    POLICY_API --> RAG_SVC
    
    INGEST --> DATADOG
    TRAJECTORY --> ARIZE
    
    style GATEWAY fill:#4A90E2
    style TRAJECTORY fill:#50C878
    style RAG_SVC fill:#9B59B6
    style EXPLAIN fill:#FFB84D
```

---

## 13. Decision Comparison Flow (Side-by-Side Analysis)

```mermaid
graph TB
    USER["User Input: Consider 3 decisions"]
    
    subgraph "Decision A: MS in US"
        A_ENVELOPE["P10:$45k | P50:$180k | P90:$420k"]
        A_LOCK["Locks out: Startup, Early Retirement"]
        A_REV["Reversibility: 0.3"]
    end
    
    subgraph "Decision B: Stay in Current Job"
        B_ENVELOPE["P10:$30k | P50:$60k | P90:$120k"]
        B_LOCK["Locks out: US Immigration"]
        B_REV["Reversibility: 0.8"]
    end
    
    subgraph "Decision C: Join Indian Startup"
        C_ENVELOPE["P10:$20k | P50:$80k | P90:$500k"]
        C_LOCK["Locks out: Stable Career Path"]
        C_REV["Reversibility: 0.5"]
    end
    
    COMPARE[Comparison Matrix]
    PARETO["Pareto Frontier: A and C non-dominated, B dominated by A"]
    
    REGRET["Regret Analysis: If B over A = -$120k median, If B over C = -$20k median"]
    
    OUTPUT["User sees: Side-by-side comparison, Pareto frontier, Regret matrix"]
    
    USER --> A_ENVELOPE
    USER --> B_ENVELOPE
    USER --> C_ENVELOPE
    
    A_ENVELOPE --> COMPARE
    B_ENVELOPE --> COMPARE
    C_ENVELOPE --> COMPARE
    
    A_LOCK --> COMPARE
    B_LOCK --> COMPARE
    C_LOCK --> COMPARE
    
    A_REV --> COMPARE
    B_REV --> COMPARE
    C_REV --> COMPARE
    
    COMPARE --> PARETO
    COMPARE --> REGRET
    
    PARETO --> OUTPUT
    REGRET --> OUTPUT
    
    style PARETO fill:#9B59B6
    style REGRET fill:#FFB84D
```

---

## 14. Real-Time RAG Integration Flow

```mermaid
sequenceDiagram
    participant User
    participant System
    participant VectorDB
    participant USCIS_API
    participant BLS_API
    participant NewsAPI
    participant LLM
    participant TransitionModel
    
    User->>System: Decision: "Move to US for SWE role"
    
    par Parallel RAG Queries
        System->>VectorDB: Query: "H1B visa policy 2026"
        System->>USCIS_API: Fetch: Current approval rates
        System->>BLS_API: Fetch: Tech job market data
        System->>NewsAPI: Query: "Tech layoffs 2025-2026"
    end
    
    VectorDB-->>System: 5 relevant policy documents
    USCIS_API-->>System: H1B approval: 76% (down from 82%)
    BLS_API-->>System: Tech job openings: -18% YoY
    NewsAPI-->>System: 150k tech layoffs in 2024-2025
    
    System->>LLM: Extract structured modifiers
    Note over LLM: Extract numerical impact on visa_difficulty, job_availability, time_delay
    
    LLM-->>System: {visa_difficulty: 1.3x, job_availability: 0.82x, delay: +4 months}
    
    System->>TransitionModel: Apply context modifiers
    TransitionModel-->>System: Adjusted P10/P50/P90
    
    System-->>User: Display: Updated predictions + context explanation
```

---

## Summary: Key Architectural Principles

```mermaid
mindmap
  root((LDC Architecture))
    Data-First
      Historical longitudinal data
      Real visa records
      Job market panels
      No LLM decision logic
    Constraint-Aware
      Hard knowledge graph
      Impossible futures blocked
      Legal/age/financial rules
    Uncertainty-Honest
      P10/P50/P90 envelopes
      No single predictions
      Calibrated probabilities
    Regret-Aware
      Counterfactual analysis
      Lock-in detection
      Reversibility scoring
    LLM-Restricted
      Explanation only
      No decision power
      Constrained prompts
      Auditable outputs
```

---

> [!IMPORTANT]
> **For Hackathon Demo**: Focus on visualizing the complete flow (#10), MVP architecture (#11), and side-by-side decision comparison (#13). These flowcharts directly demonstrate the system's unique value proposition.

> [!TIP]
> **Technical Depth Signal**: Show judges flowcharts #5 (Survival Analysis), #6 (RAG Integration), and #8 (Risk Analysis) to prove this is not a simple chatbot wrapper.
