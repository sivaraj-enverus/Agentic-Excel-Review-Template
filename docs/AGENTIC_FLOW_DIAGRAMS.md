# Agentic AI Flow Diagrams

This document provides detailed visual representations of the Agentic AI architecture and execution flows for Excel preprocessing.

## Table of Contents
1. [System Architecture Overview](#system-architecture-overview)
2. [Module Interaction Flow](#module-interaction-flow)
3. [RAG Pipeline Flow](#rag-pipeline-flow)
4. [LLM Inference Flow](#llm-inference-flow)
5. [End-to-End Processing Flow](#end-to-end-processing-flow)

---

## System Architecture Overview

### High-Level Component Architecture

```mermaid
graph TB
    subgraph "User Interface Layer"
        UI[Streamlit UI M11]
        CLI[CLI Interface]
        JUPYTER[Jupyter Notebook]
    end
    
    subgraph "Orchestration Layer"
        ORCH[Orchestrator M10]
        CONFIG[Config Loader]
        LOGGER[Log Manager M4]
    end
    
    subgraph "Data Layer"
        READER[Excel Reader M1]
        WRITER[Excel Writer M3]
    end
    
    subgraph "AI Layer"
        ASSISTANT[Review Assistant M2]
        TRACKER[Correction Tracker M8]
        PUBLISHER[Publication Agent M9]
    end
    
    subgraph "Knowledge Layer"
        INDEXER[SOP Indexer M6]
        TAXONOMY[Taxonomy Manager M5]
        VECTORSTORE[(Vector Store)]
    end
    
    subgraph "LLM Layer"
        LMSTUDIO[LM Studio Server]
        LOCALLLM[Local LLM Model]
    end
    
    subgraph "Storage Layer"
        EXCEL[(Excel Files)]
        LOGS[(JSONL Logs)]
        DOCS[(SOP Documents)]
        OUTPUT[(CSV Outputs)]
    end
    
    UI --> ORCH
    CLI --> ORCH
    JUPYTER --> ORCH
    
    ORCH --> CONFIG
    ORCH --> READER
    ORCH --> ASSISTANT
    ORCH --> WRITER
    ORCH --> LOGGER
    
    READER --> EXCEL
    WRITER --> OUTPUT
    
    ASSISTANT --> INDEXER
    ASSISTANT --> LMSTUDIO
    
    INDEXER --> VECTORSTORE
    INDEXER --> DOCS
    
    TAXONOMY --> INDEXER
    
    TRACKER --> LOGGER
    PUBLISHER --> LOGGER
    
    LMSTUDIO --> LOCALLLM
    
    LOGGER --> LOGS
    
    style ORCH fill:#4A90E2,color:#fff
    style ASSISTANT fill:#F39C12,color:#fff
    style INDEXER fill:#E74C3C,color:#fff
    style LMSTUDIO fill:#9B59B6,color:#fff
```

### Module Dependencies

```mermaid
graph LR
    M1[M1: Excel Reader] --> M2[M2: Review Assistant]
    M6[M6: SOP Indexer] --> M2
    M2 --> M3[M3: Excel Writer]
    M2 --> M4[M4: Log Manager]
    M5[M5: Taxonomy Manager] --> M6
    M2 --> M8[M8: Correction Tracker]
    M8 --> M9[M9: Publication Agent]
    M1 --> M10[M10: Orchestrator]
    M2 --> M10
    M3 --> M10
    M4 --> M10
    M10 --> M11[M11: Streamlit UI]
    
    style M10 fill:#4A90E2,color:#fff
    style M2 fill:#F39C12,color:#fff
    style M6 fill:#E74C3C,color:#fff
```

---

## Module Interaction Flow

### Complete Processing Pipeline

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Orchestrator
    participant ExcelReader
    participant SOPIndexer
    participant ReviewAssistant
    participant LMStudio
    participant ExcelWriter
    participant Logger
    
    User->>+Orchestrator: run_demo(sample_size=10)
    
    rect rgb(200, 220, 240)
        Note over Orchestrator,ExcelReader: STEP 1-2: Data Ingestion
        Orchestrator->>+ExcelReader: read_review_sheet(config)
        ExcelReader->>ExcelReader: detect_header()
        ExcelReader->>ExcelReader: clean_df()
        ExcelReader->>ExcelReader: profile_data()
        ExcelReader-->>-Orchestrator: df, profile
        Orchestrator->>Orchestrator: sample_rows(n=10)
    end
    
    rect rgb(240, 220, 200)
        Note over Orchestrator,SOPIndexer: STEP 3: RAG Setup
        Orchestrator->>+SOPIndexer: initialize(embeddings_dir)
        SOPIndexer->>SOPIndexer: load_vector_store()
        SOPIndexer-->>-Orchestrator: indexer_ready
    end
    
    loop For each row in sample
        rect rgb(220, 240, 220)
            Note over Orchestrator,LMStudio: STEP 4: AI Inference
            Orchestrator->>+ReviewAssistant: infer_reason(row)
            
            ReviewAssistant->>ReviewAssistant: extract_comment(row)
            
            ReviewAssistant->>+SOPIndexer: search(comment, top_k=4)
            SOPIndexer->>SOPIndexer: encode_query()
            SOPIndexer->>SOPIndexer: similarity_search()
            SOPIndexer-->>-ReviewAssistant: context_chunks[4]
            
            ReviewAssistant->>ReviewAssistant: generate_prompt(comment, context)
            
            ReviewAssistant->>+LMStudio: POST /chat/completions
            LMStudio->>LMStudio: inference()
            LMStudio-->>-ReviewAssistant: json_response
            
            ReviewAssistant->>ReviewAssistant: parse_json()
            ReviewAssistant->>ReviewAssistant: validate_response()
            
            ReviewAssistant->>+Logger: log_inference(row, context, response)
            Logger->>Logger: write_jsonl()
            Logger-->>-ReviewAssistant: logged
            
            ReviewAssistant-->>-Orchestrator: ai_result
        end
    end
    
    rect rgb(240, 240, 200)
        Note over Orchestrator,ExcelWriter: STEP 5-6: Output Generation
        Orchestrator->>Orchestrator: aggregate_results()
        Orchestrator->>Orchestrator: merge_ai_columns()
        
        Orchestrator->>+ExcelWriter: write_csv(df_with_ai)
        ExcelWriter->>ExcelWriter: validate_ai_columns()
        ExcelWriter-->>-Orchestrator: output_path
    end
    
    rect rgb(220, 220, 240)
        Note over Orchestrator,Logger: STEP 7: Logging & Summary
        Orchestrator->>+Logger: log_pipeline_run()
        Logger-->>-Orchestrator: log_path
        
        Orchestrator->>Orchestrator: calculate_summary()
    end
    
    Orchestrator-->>-User: summary_stats
```

---

## RAG Pipeline Flow

### Document Indexing Process

```mermaid
flowchart TD
    START([Start: SOP Indexing]) --> LOAD[Load SOP Documents]
    LOAD --> CHECK{File Type?}
    
    CHECK -->|PDF| EXTRACT_PDF[Extract Text from PDF]
    CHECK -->|DOCX| EXTRACT_DOCX[Extract Text from DOCX]
    CHECK -->|Other| SKIP[Skip File]
    
    EXTRACT_PDF --> COMPUTE_HASH[Compute SHA-256 Checksum]
    EXTRACT_DOCX --> COMPUTE_HASH
    
    COMPUTE_HASH --> DETECT_SECTIONS[Detect Section Headers]
    DETECT_SECTIONS --> CHUNK{Section Found?}
    
    CHUNK -->|Yes| SPLIT_SECTIONS[Split by Sections]
    CHUNK -->|No| SPLIT_DOC[Treat as Single Document]
    
    SPLIT_SECTIONS --> CHECK_LENGTH{Length > 1600?}
    SPLIT_DOC --> CHECK_LENGTH
    
    CHECK_LENGTH -->|Yes| SUBSPLIT[Split into Sub-Chunks]
    CHECK_LENGTH -->|No| PRESERVE[Preserve as Single Chunk]
    
    SUBSPLIT --> METADATA[Add Metadata]
    PRESERVE --> METADATA
    
    METADATA --> DEDUP{Already Indexed?}
    
    DEDUP -->|Yes| SKIP_CHUNK[Skip Duplicate]
    DEDUP -->|No| EMBED[Generate Embeddings]
    
    EMBED --> NORMALIZE[Normalize Vectors]
    NORMALIZE --> STORE[Store in Vector DB]
    
    STORE --> MORE{More Chunks?}
    MORE -->|Yes| DETECT_SECTIONS
    MORE -->|No| STATS[Generate Statistics]
    
    SKIP_CHUNK --> MORE
    SKIP --> LOAD
    
    STATS --> END([End: Index Ready])
    
    style START fill:#4A90E2,color:#fff
    style END fill:#27AE60,color:#fff
    style EMBED fill:#F39C12,color:#fff
    style STORE fill:#E74C3C,color:#fff
```

### RAG Retrieval Process

```mermaid
flowchart TD
    START([Query: Comment Text]) --> PREPROCESS[Preprocess Query]
    
    PREPROCESS --> ENCODE[Encode Query with<br/>Sentence Transformer]
    
    ENCODE --> NORMALIZE[Normalize Query Vector]
    
    NORMALIZE --> SEARCH[Vector Similarity Search<br/>in ChromaDB/FAISS]
    
    SEARCH --> TOPK[Retrieve Top-K Results<br/>Default: K=4]
    
    TOPK --> SCORE[Calculate Similarity Scores]
    
    SCORE --> METADATA[Attach Metadata<br/>section, doc_id, title, page]
    
    METADATA --> FORMAT[Format Context Chunks]
    
    FORMAT --> CONCAT[Concatenate Context<br/>with Separators]
    
    CONCAT --> RETURN[Return Context String]
    
    RETURN --> END([Context for LLM Prompt])
    
    style START fill:#4A90E2,color:#fff
    style END fill:#27AE60,color:#fff
    style SEARCH fill:#E74C3C,color:#fff
    style CONCAT fill:#F39C12,color:#fff
```

---

## LLM Inference Flow

### Review Assistant Processing

```mermaid
flowchart TD
    START([Input: DataFrame Row]) --> EXTRACT[Extract Comment Field]
    
    EXTRACT --> VALIDATE{Comment Empty?}
    
    VALIDATE -->|Yes| ERROR_EMPTY[Return Error Result<br/>confidence=0.0]
    VALIDATE -->|No| RAG_SEARCH[RAG: Search SOP Context]
    
    RAG_SEARCH --> CONTEXT_OK{Context Found?}
    
    CONTEXT_OK -->|No| WARN[Warning: No Context<br/>Proceed with Empty Context]
    CONTEXT_OK -->|Yes| BUILD_PROMPT[Build Structured Prompt]
    
    WARN --> BUILD_PROMPT
    
    BUILD_PROMPT --> TEMPLATE[Load Prompt Template]
    TEMPLATE --> SUBSTITUTE[Substitute Variables<br/>comment, context]
    
    SUBSTITUTE --> API_CALL[Call LM Studio API<br/>POST /chat/completions]
    
    API_CALL --> API_OK{API Success?}
    
    API_OK -->|No| ERROR_API[Return Error Result<br/>LLM Connection Failed]
    API_OK -->|Yes| PARSE[Parse LLM Response]
    
    PARSE --> PARSE_OK{JSON Valid?}
    
    PARSE_OK -->|No| TRY_EXTRACT[Try Extract JSON<br/>from Markdown/Text]
    PARSE_OK -->|Yes| VALIDATE_STRUCT[Validate JSON Structure]
    
    TRY_EXTRACT --> EXTRACT_OK{Extraction Success?}
    
    EXTRACT_OK -->|No| ERROR_PARSE[Return Error Result<br/>Failed to Parse JSON]
    EXTRACT_OK -->|Yes| VALIDATE_STRUCT
    
    VALIDATE_STRUCT --> STRUCT_OK{All Fields Present?}
    
    STRUCT_OK -->|No| ERROR_STRUCT[Return Error Result<br/>Invalid Response Structure]
    STRUCT_OK -->|Yes| VALIDATE_CONF[Validate Confidence<br/>0.0 ≤ conf ≤ 1.0]
    
    VALIDATE_CONF --> CONF_OK{Valid Range?}
    
    CONF_OK -->|No| ERROR_CONF[Return Error Result<br/>Invalid Confidence Value]
    CONF_OK -->|Yes| LOG[Log Inference to JSONL]
    
    LOG --> FORMAT_OUTPUT[Format Output Dictionary<br/>AI_reason, AI_confidence, etc.]
    
    FORMAT_OUTPUT --> RETURN[Return AI Result]
    
    ERROR_EMPTY --> RETURN
    ERROR_API --> RETURN
    ERROR_PARSE --> RETURN
    ERROR_STRUCT --> RETURN
    ERROR_CONF --> RETURN
    
    RETURN --> END([Output: AI Columns Dict])
    
    style START fill:#4A90E2,color:#fff
    style END fill:#27AE60,color:#fff
    style API_CALL fill:#9B59B6,color:#fff
    style PARSE fill:#F39C12,color:#fff
    style LOG fill:#3498DB,color:#fff
    style ERROR_EMPTY fill:#E74C3C,color:#fff
    style ERROR_API fill:#E74C3C,color:#fff
    style ERROR_PARSE fill:#E74C3C,color:#fff
    style ERROR_STRUCT fill:#E74C3C,color:#fff
    style ERROR_CONF fill:#E74C3C,color:#fff
```

### LLM JSON Response Parsing

```mermaid
flowchart TD
    START([LLM Response String]) --> STRIP[Strip Whitespace]
    
    STRIP --> TRY_DIRECT[Try Direct JSON Parse]
    
    TRY_DIRECT --> DIRECT_OK{Success?}
    
    DIRECT_OK -->|Yes| RETURN_JSON[Return Parsed JSON]
    DIRECT_OK -->|No| CHECK_MARKDOWN{Contains ```json?}
    
    CHECK_MARKDOWN -->|Yes| EXTRACT_MD[Extract from Markdown<br/>Code Block]
    CHECK_MARKDOWN -->|No| FIND_BRACES[Find { and } positions]
    
    EXTRACT_MD --> PARSE_MD[Parse Extracted String]
    
    PARSE_MD --> MD_OK{Success?}
    
    MD_OK -->|Yes| RETURN_JSON
    MD_OK -->|No| FIND_BRACES
    
    FIND_BRACES --> BRACES_FOUND{Braces Found?}
    
    BRACES_FOUND -->|Yes| EXTRACT_JSON[Extract String<br/>Between Braces]
    BRACES_FOUND -->|No| CLEAN_RESPONSE[Clean Response<br/>Remove Escapes]
    
    EXTRACT_JSON --> PARSE_EXTRACTED[Parse Extracted String]
    
    PARSE_EXTRACTED --> EXTRACT_OK{Success?}
    
    EXTRACT_OK -->|Yes| RETURN_JSON
    EXTRACT_OK -->|No| CLEAN_RESPONSE
    
    CLEAN_RESPONSE --> PARSE_CLEAN[Parse Cleaned String]
    
    PARSE_CLEAN --> CLEAN_OK{Success?}
    
    CLEAN_OK -->|Yes| RETURN_JSON
    CLEAN_OK -->|No| LOG_ERROR[Log Parsing Error]
    
    LOG_ERROR --> RETURN_NULL[Return None]
    
    RETURN_JSON --> END([Parsed JSON Dict])
    RETURN_NULL --> END
    
    style START fill:#4A90E2,color:#fff
    style END fill:#27AE60,color:#fff
    style RETURN_NULL fill:#E74C3C,color:#fff
    style TRY_DIRECT fill:#F39C12,color:#fff
```

---

## End-to-End Processing Flow

### Complete Agentic Pipeline

```mermaid
graph TB
    subgraph "Input Phase"
        EXCEL[Excel Workbook<br/>Sample_Review_Workbook.xlsx]
        CONFIG[Configuration<br/>config.json]
        SOPS[SOP Documents<br/>PDFs/DOCX]
    end
    
    subgraph "Ingestion Phase"
        M1[M1: Excel Reader]
        M1_DETECT[Auto Header Detection]
        M1_CLEAN[Data Cleaning]
        M1_PROFILE[Data Profiling]
        
        M1 --> M1_DETECT --> M1_CLEAN --> M1_PROFILE
    end
    
    subgraph "Knowledge Phase"
        M6[M6: SOP Indexer]
        M6_LOAD[Load Documents]
        M6_CHUNK[Semantic Chunking]
        M6_EMBED[Generate Embeddings]
        M6_INDEX[Store in Vector DB]
        
        M6 --> M6_LOAD --> M6_CHUNK --> M6_EMBED --> M6_INDEX
    end
    
    subgraph "Processing Phase - Per Row"
        M2[M2: Review Assistant]
        M2_EXTRACT[Extract Comment]
        M2_RAG[RAG Context Retrieval]
        M2_PROMPT[Generate Prompt]
        M2_LLM[LLM Inference]
        M2_PARSE[Parse & Validate]
        M2_LOG[Log to JSONL]
        
        M2 --> M2_EXTRACT --> M2_RAG --> M2_PROMPT
        M2_PROMPT --> M2_LLM --> M2_PARSE --> M2_LOG
    end
    
    subgraph "Output Phase"
        M3[M3: Excel Writer]
        M3_MERGE[Merge AI Columns]
        M3_VALIDATE[Validate Output]
        M3_EXPORT[Export CSV]
        
        M3 --> M3_MERGE --> M3_VALIDATE --> M3_EXPORT
    end
    
    subgraph "Monitoring Phase"
        M4[M4: Log Manager]
        M8[M8: Correction Tracker]
        M9[M9: Publication Agent]
        
        M4 --> M8 --> M9
    end
    
    subgraph "Orchestration"
        M10[M10: Orchestrator]
        M10_STEP1[Step 1: Load Config]
        M10_STEP2[Step 2: Read Excel]
        M10_STEP3[Step 3: Init RAG]
        M10_STEP4[Step 4: Process Rows]
        M10_STEP5[Step 5: Write Output]
        M10_STEP6[Step 6: Log & Summary]
        
        M10 --> M10_STEP1 --> M10_STEP2 --> M10_STEP3
        M10_STEP3 --> M10_STEP4 --> M10_STEP5 --> M10_STEP6
    end
    
    subgraph "Output Artifacts"
        CSV[CSV with AI Columns<br/>excel_review_demo.csv]
        LOGS[JSONL Audit Logs<br/>review_assistant.jsonl]
        SUMMARY[Summary Statistics<br/>confidence, counts]
    end
    
    EXCEL --> M1
    CONFIG --> M10
    SOPS --> M6
    
    M1_PROFILE --> M10_STEP2
    M6_INDEX --> M2_RAG
    
    M10_STEP2 --> M2
    M2_LOG --> M4
    M2_PARSE --> M10_STEP4
    M10_STEP4 --> M3
    M3_EXPORT --> CSV
    M4 --> LOGS
    M10_STEP6 --> SUMMARY
    
    style M10 fill:#4A90E2,color:#fff
    style M2 fill:#F39C12,color:#fff
    style M6 fill:#E74C3C,color:#fff
    style CSV fill:#27AE60,color:#fff
    style LOGS fill:#3498DB,color:#fff
```

### Data Transformation Flow

```mermaid
graph LR
    subgraph "Stage 1: Raw Excel"
        RAW[Raw Excel Data<br/>Unknown header position<br/>Mixed data types<br/>Empty rows/columns]
    end
    
    subgraph "Stage 2: Cleaned DataFrame"
        CLEAN[Cleaned DataFrame<br/>Header detected at row 0<br/>Normalized columns<br/>Empty rows removed<br/>Profile generated]
    end
    
    subgraph "Stage 3: Sampled Data"
        SAMPLE[Sample N Rows<br/>For processing<br/>With comments<br/>Quality Review column]
    end
    
    subgraph "Stage 4: Context-Enriched"
        ENRICHED[Rows + SOP Context<br/>Each row has 4 context chunks<br/>From vector similarity search<br/>Relevant SOP sections]
    end
    
    subgraph "Stage 5: AI-Analyzed"
        ANALYZED[Rows + AI Columns<br/>AI_reason<br/>AI_confidence<br/>AI_comment_standardized<br/>AI_rationale_short<br/>AI_model_version]
    end
    
    subgraph "Stage 6: Final Output"
        OUTPUT[CSV with All Columns<br/>Original + AI columns<br/>Audit trail in JSONL<br/>Summary statistics]
    end
    
    RAW -->|Excel Reader M1| CLEAN
    CLEAN -->|Orchestrator M10| SAMPLE
    SAMPLE -->|RAG M6| ENRICHED
    ENRICHED -->|Review Assistant M2| ANALYZED
    ANALYZED -->|Excel Writer M3| OUTPUT
    
    style RAW fill:#E8E8E8
    style CLEAN fill:#D5E8F7
    style SAMPLE fill:#C7E2F4
    style ENRICHED fill:#B9DCF1
    style ANALYZED fill:#ABD6EE
    style OUTPUT fill:#27AE60,color:#fff
```

---

## Agentic Decision Points

### Intelligence in the Pipeline

```mermaid
flowchart TD
    START([Excel File]) --> AGENT1{Agent 1: Header Detection}
    
    AGENT1 -->|Heuristic: Max Non-Empty Count| DEC1[Decision: Header at Row X]
    
    DEC1 --> AGENT2{Agent 2: Data Cleaning}
    
    AGENT2 -->|Rule: Drop Empty Cols/Rows| DEC2[Decision: Keep N Columns]
    
    DEC2 --> AGENT3{Agent 3: Context Retrieval}
    
    AGENT3 -->|Semantic: Vector Similarity| DEC3[Decision: Top-4 SOP Sections]
    
    DEC3 --> AGENT4{Agent 4: LLM Reasoning}
    
    AGENT4 -->|Reasoning: RAG + Prompt| DEC4[Decision: Standardized Reason]
    
    DEC4 --> AGENT5{Agent 5: Confidence Scoring}
    
    AGENT5 -->|Self-Assessment| DEC5[Decision: Confidence 0.0-1.0]
    
    DEC5 --> AGENT6{Agent 6: Error Handling}
    
    AGENT6 -->|Exception Handling| DEC6[Decision: Continue or Fail Gracefully]
    
    DEC6 --> END([Output with AI Suggestions])
    
    style AGENT1 fill:#4A90E2,color:#fff
    style AGENT2 fill:#4A90E2,color:#fff
    style AGENT3 fill:#F39C12,color:#fff
    style AGENT4 fill:#9B59B6,color:#fff
    style AGENT5 fill:#E74C3C,color:#fff
    style AGENT6 fill:#27AE60,color:#fff
```

---

## Performance & Monitoring

### Pipeline Metrics Flow

```mermaid
graph TD
    subgraph "Input Metrics"
        IN_ROWS[Total Rows]
        IN_COLS[Total Columns]
        IN_SIZE[File Size]
    end
    
    subgraph "Processing Metrics"
        PROC_TIME[Processing Time per Row]
        PROC_TOKENS[Tokens Used]
        PROC_CONTEXT[Context Chunks Retrieved]
    end
    
    subgraph "Quality Metrics"
        QUAL_CONF[Average Confidence]
        QUAL_ERRORS[Error Rate]
        QUAL_PARSEFAIL[Parse Failure Rate]
    end
    
    subgraph "Output Metrics"
        OUT_SUCCESS[Successful Inferences]
        OUT_FAILED[Failed Inferences]
        OUT_LOWCONF[Low Confidence Count]
    end
    
    subgraph "Monitoring Dashboard"
        DASHBOARD[M11: Streamlit UI<br/>KPI Dashboard]
    end
    
    IN_ROWS --> PROC_TIME
    IN_COLS --> PROC_TIME
    PROC_TIME --> QUAL_CONF
    PROC_TOKENS --> QUAL_ERRORS
    PROC_CONTEXT --> QUAL_PARSEFAIL
    
    QUAL_CONF --> OUT_SUCCESS
    QUAL_ERRORS --> OUT_FAILED
    QUAL_PARSEFAIL --> OUT_LOWCONF
    
    OUT_SUCCESS --> DASHBOARD
    OUT_FAILED --> DASHBOARD
    OUT_LOWCONF --> DASHBOARD
    
    style DASHBOARD fill:#4A90E2,color:#fff
```

---

## Conclusion

These diagrams illustrate the **comprehensive agentic architecture** where:

1. **Multiple specialized agents** work together (Excel Reader, SOP Indexer, Review Assistant, Orchestrator)
2. **Intelligent decision-making** happens at each stage (header detection, semantic search, LLM reasoning)
3. **Robust error handling** ensures graceful degradation
4. **Complete traceability** through logging and monitoring
5. **Modular design** allows independent testing and replacement of components

The system demonstrates true **agentic behavior** through autonomous operation, structured reasoning, and self-awareness (confidence scoring).

---

**See Also**: 
- [AGENTIC_AI_ARCHITECTURE.md](./AGENTIC_AI_ARCHITECTURE.md) - Detailed architecture documentation
- [Project_Structure.md](./Project_Structure.md) - Module organization
- [LM_STUDIO_SETUP.md](./LM_STUDIO_SETUP.md) - LLM setup guide
