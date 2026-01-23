# Requirements Document

## Introduction

Life Decision Coach (LDC) is a data-driven AI system that computes reachable future life trajectories for high-stakes decisions rather than providing advice. The system uses a 5-layer architecture with LLMs restricted to explanation only, focusing on constraint-aware modeling of reality with honest uncertainty representation.

## Glossary

- **Life_State_Vector**: Structured representation of a person's current situation (age, education, location, visa status, experience, finances)
- **Trajectory_Envelope**: Statistical distribution of possible future outcomes (P10/P50/P90 percentiles)
- **Constraint_Graph**: Neo4j knowledge graph modeling hard reality constraints (visa rules, degree requirements, age limits)
- **Lock_In_Point**: Decision moment that irreversibly closes future options
- **Regret_Score**: Quantified opportunity cost of choosing one path over alternatives
- **Reality_Constraint_Layer**: System component that enforces hard rules from real-world data
- **Trajectory_Intelligence**: Core computation engine for outcome prediction using empirical models
- **Explanation_Layer**: LLM-powered component for translating technical results into human language
- **Decision_Domain**: Specific area of life decisions (e.g., software engineering careers)
- **Provenance_Citation**: Traceable reference to data sources supporting each prediction

## Requirements

### Requirement 1: Life State Ingestion

**User Story:** As a user, I want to input my current life situation, so that the system can compute my reachable future trajectories.

#### Acceptance Criteria

1. WHEN a user provides personal information, THE Life_State_Ingestion_System SHALL convert it into a structured Life_State_Vector
2. WHEN a user uploads a resume, THE Resume_Parser SHALL extract relevant career information automatically
3. WHEN incomplete information is provided, THE System SHALL identify missing critical fields and request them
4. THE Life_State_Vector SHALL include age, education level, current location, visa status, years of experience, and current compensation
5. WHEN a Life_State_Vector is created, THE System SHALL validate all fields against expected data types and ranges

### Requirement 2: Reality Constraint Modeling

**User Story:** As a user, I want the system to understand real-world constraints, so that I only see feasible future paths.

#### Acceptance Criteria

1. THE Constraint_Graph SHALL model US and India visa requirements as hard constraints
2. THE Constraint_Graph SHALL model degree requirements for career transitions
3. THE Constraint_Graph SHALL model age limits for various opportunities
4. WHEN computing trajectories, THE Reality_Constraint_Layer SHALL filter out impossible paths
5. WHEN a constraint blocks a desired path, THE System SHALL identify the specific blocking constraint
6. THE Constraint_Graph SHALL be stored in Neo4j with relationships between constraints and opportunities

### Requirement 3: Trajectory Computation

**User Story:** As a user, I want to see multiple possible futures with uncertainty ranges, so that I can make informed decisions about risk.

#### Acceptance Criteria

1. THE Trajectory_Intelligence SHALL compute P10, P50, and P90 outcome envelopes for each feasible path
2. THE System SHALL predict net worth trajectories over a 5-year horizon
3. THE System SHALL use quantile regression models trained on historical career data
4. WHEN multiple paths are available, THE System SHALL compute trajectories for all feasible options
5. THE Trajectory_Intelligence SHALL incorporate survival analysis for career transition success rates
6. THE System SHALL update predictions when new data becomes available through RAG integration

### Requirement 4: Lock-In Detection and Warning

**User Story:** As a user, I want to be warned about irreversible decisions, so that I don't accidentally close important future options.

#### Acceptance Criteria

1. WHEN a decision creates a Lock_In_Point, THE System SHALL identify and flag it prominently
2. THE Lock_In_Detection_System SHALL analyze how each decision affects future option availability
3. WHEN a Lock_In_Point is detected, THE System SHALL quantify how many future paths it closes
4. THE System SHALL warn users before they commit to irreversible decisions
5. THE Lock_In_Analysis SHALL consider visa timing, age limits, and career transition windows

### Requirement 5: Regret Analysis

**User Story:** As a user, I want to understand the opportunity cost of my decisions, so that I can minimize future regret.

#### Acceptance Criteria

1. THE Regret_Computation_System SHALL calculate counterfactual outcomes for alternative paths
2. THE System SHALL compute Regret_Scores by comparing chosen path outcomes with foregone alternatives
3. WHEN displaying regret analysis, THE System SHALL show both upside missed and downside avoided
4. THE Regret_Analysis SHALL consider multiple objectives (income, career satisfaction, location preferences)
5. THE System SHALL identify decisions with highest potential regret to prioritize user attention

### Requirement 6: Data Provenance and Trust

**User Story:** As a user, I want to see the sources behind predictions, so that I can trust and verify the system's recommendations.

#### Acceptance Criteria

1. THE System SHALL provide Provenance_Citations for all data sources used in predictions
2. WHEN displaying trajectory predictions, THE System SHALL show confidence intervals based on data quality
3. THE System SHALL cite specific datasets (H1B data, Glassdoor salaries, immigration statistics)
4. WHEN data is outdated or limited, THE System SHALL clearly indicate uncertainty levels
5. THE Provenance_System SHALL maintain audit trails of all data sources and their last update times

### Requirement 7: Interactive Constraint Visualization

**User Story:** As a user, I want to see how constraints affect my options visually, so that I can understand the decision landscape.

#### Acceptance Criteria

1. THE Constraint_Graph_Visualizer SHALL display feasible and blocked paths interactively
2. WHEN a user clicks on a constraint, THE System SHALL highlight all paths it affects
3. THE Visualization SHALL use different colors for feasible, blocked, and uncertain paths
4. THE Interactive_Graph SHALL allow users to explore "what-if" scenarios by modifying constraints
5. THE Visualization SHALL integrate with D3.js for smooth, responsive interactions

### Requirement 8: Multi-Objective Optimization

**User Story:** As a user, I want to see trade-offs between different goals, so that I can make balanced decisions.

#### Acceptance Criteria

1. THE Pareto_Frontier_Calculator SHALL identify optimal trade-offs between income, risk, and other objectives
2. THE System SHALL display Pareto-optimal solutions graphically
3. WHEN objectives conflict, THE System SHALL show the trade-off curves clearly
4. THE Multi_Objective_Optimizer SHALL allow users to adjust preference weights
5. THE System SHALL highlight dominated solutions that are clearly inferior

### Requirement 9: Explanation Generation

**User Story:** As a user, I want complex technical results explained in plain language, so that I can understand and act on them.

#### Acceptance Criteria

1. THE Explanation_Layer SHALL use GPT-4 to translate technical results into human-readable explanations
2. WHEN generating explanations, THE LLM SHALL only explain existing computed results, not create new predictions
3. THE Explanation_System SHALL provide context for why certain paths are recommended or discouraged
4. THE LLM SHALL explain the reasoning behind Lock_In_Point warnings and Regret_Scores
5. THE Explanation_Layer SHALL maintain consistency with the underlying mathematical computations

### Requirement 10: Software Engineering Career Domain

**User Story:** As a software engineer, I want career-specific trajectory analysis, so that I can make informed decisions about my tech career.

#### Acceptance Criteria

1. THE System SHALL model software engineering career paths in US and India markets
2. THE Career_Domain_Model SHALL include role progressions (SDE → Senior → Staff → Principal)
3. THE System SHALL incorporate H1B lottery probabilities and visa timeline constraints
4. THE Domain_Model SHALL include company tier effects on career trajectories (FAANG vs startup vs mid-size)
5. THE System SHALL model compensation bands, equity outcomes, and career advancement timelines specific to software engineering

### Requirement 11: Data Integration and Updates

**User Story:** As a user, I want the system to use current market data, so that my trajectory predictions reflect recent trends.

#### Acceptance Criteria

1. THE RAG_Integration_System SHALL pull live data from job markets, visa processing times, and salary trends
2. THE System SHALL update trajectory models when significant market changes are detected
3. THE Data_Pipeline SHALL integrate with Glassdoor, H1B databases, and immigration statistics
4. WHEN data sources are unavailable, THE System SHALL gracefully degrade and indicate reduced confidence
5. THE Update_System SHALL maintain data freshness indicators for all information sources

### Requirement 12: User Experience and Interface

**User Story:** As a user, I want an intuitive interface for exploring complex decision scenarios, so that I can efficiently analyze my options.

#### Acceptance Criteria

1. THE Frontend SHALL provide a React-based interface with responsive design
2. THE Interface SHALL allow easy switching between different trajectory views (income, risk, regret)
3. THE System SHALL provide export functionality for trajectory data and visualizations
4. THE UI SHALL indicate system confidence levels and data quality throughout the interface
5. THE Interface SHALL support comparison views for analyzing multiple decision paths simultaneously
