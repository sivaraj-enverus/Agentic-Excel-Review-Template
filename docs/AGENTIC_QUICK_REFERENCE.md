# Agentic AI Quick Reference Guide

## What is Agentic AI for Excel Preprocessing?

This system uses **Agentic AI** - autonomous AI agents that perceive, reason, act, and learn - to intelligently preprocess and analyze Excel review data.

---

## 🎯 Quick Start: Understanding the System

### **3-Minute Overview**

The system has 4 main agentic components:

1. **Excel Reader Agent (M1)** - Intelligently reads Excel files
   - Auto-detects headers
   - Cleans and normalizes data
   - Creates data profile

2. **SOP Indexer Agent (M6)** - Builds knowledge base
   - Indexes SOP documents into vector database
   - Enables semantic search
   - Retrieves relevant context

3. **Review Assistant Agent (M2)** - Analyzes comments
   - Uses RAG (Retrieval-Augmented Generation)
   - Calls local LLM for reasoning
   - Generates standardized suggestions

4. **Orchestrator Agent (M10)** - Coordinates everything
   - Manages workflow
   - Handles errors
   - Logs all steps

---

## 📊 How Does It Work? (Example)

### **Input: Excel Row**
```
Row 1: "Equipment not calibrated according to SOP-123456. Missing calibration certificate."
```

### **Step-by-Step Agentic Processing**

#### **Step 1: Excel Reader Agent (M1)**
```python
# Automatically detects header and reads data
df, profile = read_review_sheet(config)
# Output: Clean DataFrame with detected header at row 0
```

#### **Step 2: SOP Indexer Agent (M6) - RAG Context**
```python
# Semantic search for relevant SOP sections
context_chunks = sop_indexer.search("Equipment not calibrated", top_k=4)

# Retrieved Context:
# 1. "4.1.3 Equipment Calibration Requirements - All measuring equipment..."
# 2. "5.2 Calibration Certificate Documentation - Each calibration must..."
# 3. "6.1 Annual Calibration Schedule - Equipment must be calibrated..."
# 4. "7.3 Non-Compliance Procedures - Missing certificates require..."
```

#### **Step 3: Review Assistant Agent (M2) - LLM Reasoning**
```python
# Generate structured prompt with comment + context
prompt = f"""
System: You are an expert in analyzing Excel review comments...

Comment: {comment}
Context: {context_chunks}

Return JSON: {{"reason": "...", "confidence": 0.85, ...}}
"""

# Call local LLM
response = llm.infer(prompt)

# LLM Output:
{
  "reason": "Missing Calibration Certificate",
  "confidence": 0.92,
  "comment_standardized": "Equipment calibration incomplete - temperature sensor missing certificate per SOP-123456 §4.1.3",
  "rationale_short": "Comment clearly identifies missing calibration documentation for specific equipment",
  "model_version": "ExcelReview-v0.1"
}
```

#### **Step 4: Orchestrator Agent (M10) - Aggregation**
```python
# Add AI columns to original DataFrame
df['AI_reason'] = "Missing Calibration Certificate"
df['AI_confidence'] = 0.92
df['AI_comment_standardized'] = "Equipment calibration incomplete..."
df['AI_rationale_short'] = "Comment clearly identifies..."
df['AI_model_version'] = "ExcelReview-v0.1"

# Export to CSV
df.to_csv('out/excel_review_demo.csv')

# Log to JSONL for audit trail
log_inference(row, context, response)
```

### **Output: Enhanced Excel Data**
```
Original + AI Columns:
- AI_reason: "Missing Calibration Certificate"
- AI_confidence: 0.92
- AI_comment_standardized: "Equipment calibration incomplete..."
- AI_rationale_short: "Comment clearly identifies..."
- AI_model_version: "ExcelReview-v0.1"
```

---

## 🧠 Key Agentic AI Concepts

### **1. Autonomy**
- **Header Detection**: Automatically finds header row without manual configuration
- **Semantic Chunking**: Intelligently splits SOP documents by sections
- **Context Retrieval**: Automatically finds relevant SOP sections

### **2. Reasoning**
- **RAG**: Retrieves relevant knowledge before making decisions
- **Structured Prompts**: Clear instructions for consistent LLM responses
- **Validation**: Checks output structure and confidence ranges

### **3. Self-Awareness**
- **Confidence Scoring**: AI expresses certainty (0.0 to 1.0)
- **Rationale**: Explains why it made each decision
- **Error Handling**: Gracefully handles failures with low confidence

### **4. Learning & Adaptation**
- **Logging**: Every inference logged to JSONL for review
- **Correction Tracking**: Compares AI suggestions to human decisions
- **Metrics**: Tracks performance (accuracy, confidence, speed)

---

## 🔧 Architecture at a Glance

```
┌─────────────┐
│ Excel File  │
└──────┬──────┘
       │
       ▼
┌─────────────┐      ┌─────────────┐
│ M1: Reader  │      │ M6: SOP     │
│ (Auto-      │      │ Indexer     │
│  detect)    │      │ (RAG)       │
└──────┬──────┘      └──────┬──────┘
       │                     │
       │    ┌────────────────┘
       │    │
       ▼    ▼
    ┌───────────┐
    │ M2: Review│
    │ Assistant │
    │ (LLM)     │
    └─────┬─────┘
          │
          ▼
    ┌───────────┐
    │ M10:      │
    │ Orchestr. │
    └─────┬─────┘
          │
          ▼
    ┌───────────┐
    │ CSV Output│
    │ + Logs    │
    └───────────┘
```

---

## 📈 What Makes This "Agentic"?

| Traditional Script | Agentic AI |
|-------------------|------------|
| Fixed header row | Auto-detects header |
| Keyword matching | Semantic understanding |
| Rule-based | LLM reasoning |
| Brittle on changes | Adapts to variations |
| No explanation | Provides rationale |
| No confidence | Self-aware scoring |
| Basic logging | Full audit trail |

---

## 🎓 Technical Implementation

### **RAG (Retrieval-Augmented Generation)**
```python
# 1. Index SOP documents
sop_indexer.load_docs(["SOP-123456.pdf", "SOP-789012.docx"])
sop_indexer.chunk_and_clean(docs)  # Semantic chunking
sop_indexer.embed_and_index(chunks)  # Vector embeddings

# 2. Retrieve context at inference time
context = sop_indexer.search("calibration certificate", top_k=4)
# Returns 4 most semantically similar SOP sections

# 3. Augment LLM prompt with retrieved context
prompt = f"Comment: {comment}\nContext: {context}\nAnalyze..."
```

### **LLM Integration (LM Studio)**
```python
# Local LLM via OpenAI-compatible API
response = requests.post(
    "http://127.0.0.1:1234/v1/chat/completions",
    json={
        "model": "local-model",
        "messages": [{"role": "user", "content": prompt}],
        "temperature": 0.1,  # Low for consistency
        "max_tokens": 500
    }
)
```

### **Orchestration Pattern**
```python
# Sequential execution with error handling
for row in df.iterrows():
    try:
        ai_result = review_assistant.infer_reason(row)
        ai_results.append(ai_result)
        log_inference(row, ai_result)
    except Exception as e:
        ai_results.append(error_result(e))
        log_error(row, e)
```

---

## 📋 Complete Data Flow

```
1. Excel File
   ↓ (M1: Excel Reader - Auto-detect header)
2. Clean DataFrame
   ↓ (M10: Orchestrator - Sample rows)
3. Sample Rows
   ↓ (M6: SOP Indexer - Semantic search)
4. Rows + SOP Context
   ↓ (M2: Review Assistant - LLM inference)
5. Rows + AI Columns
   ↓ (M3: Excel Writer - Export)
6. CSV Output + JSONL Logs
```

---

## 🎯 Key Files to Explore

### **Documentation (Start Here)**
- `docs/AGENTIC_AI_ARCHITECTURE.md` - **Complete architecture explanation**
- `docs/AGENTIC_FLOW_DIAGRAMS.md` - **Visual flow diagrams**
- `README.md` - Quick start guide

### **Core Implementation**
- `src/ai/orchestrator.py` - Master agent coordinator
- `src/ai/review_assistant.py` - LLM reasoning agent
- `src/utils/sop_indexer.py` - RAG implementation
- `src/excel/excel_reader.py` - Excel ingestion agent

### **Configuration**
- `config.json` - System configuration
- `ai/prompts/review_prompt.txt` - LLM prompt template

---

## 🚀 Running the System

### **Quick Demo**
```bash
# 1. Start LM Studio (load a model and start server)
# 2. Run demo
python Agentic_Excel_Review_Demo.py

# Or with custom sample size
python src/ai/orchestrator.py --n 20
```

### **Expected Output**
```
[STEP 1] Load config: Configuration loaded
[STEP 2] Load sample from Review Sheet: Loaded 500 rows
[STEP 3] Build context (RAG): SOP indexer initialized
[STEP 4] Call LLM for reasoning: Processing 10 rows
  Processed row 1/10
  Processed row 2/10
  ...
[STEP 5] Write AI_ columns to demo CSV: Saved to out/excel_review_demo.csv
[STEP 6] Log each inference: Logs written to logs/review_assistant.jsonl
[DONE] Summary:
  Rows processed: 10
  Average confidence: 0.883
  Output file: out/excel_review_demo.csv
```

---

## 🎓 Learning Path

### **Level 1: Understanding**
1. Read `docs/AGENTIC_AI_ARCHITECTURE.md` - Understand core concepts
2. View `docs/AGENTIC_FLOW_DIAGRAMS.md` - See visual flows
3. Run `Agentic_Excel_Review_Demo.py` - See it in action

### **Level 2: Exploring**
1. Examine `src/excel/excel_reader.py` - See auto-header detection
2. Study `src/utils/sop_indexer.py` - Understand RAG
3. Review `src/ai/review_assistant.py` - See LLM integration

### **Level 3: Customizing**
1. Modify `ai/prompts/review_prompt.txt` - Customize LLM behavior
2. Adjust `config.json` - Change file paths, sample sizes
3. Extend `src/ai/orchestrator.py` - Add new processing steps

---

## 💡 Key Insights

### **Why This is "Agentic"**
1. **Autonomous Operation**: No manual header specification or hardcoded rules
2. **Intelligent Reasoning**: LLM understands context via RAG
3. **Self-Awareness**: Confidence scoring and rationale generation
4. **Adaptive**: Handles format variations and unknown comments
5. **Coordinated**: Multiple specialized agents work together

### **Design Philosophy**
- **Safety First**: Read-only access, AI_ column prefix, human oversight
- **Transparency**: Full logging, explainable outputs
- **Modularity**: Each agent is independent and testable
- **Compliance**: Audit trail, model versioning, correction tracking

---

## 📚 Additional Resources

### **Detailed Documentation**
- [Complete Architecture Guide](docs/AGENTIC_AI_ARCHITECTURE.md)
- [Visual Flow Diagrams](docs/AGENTIC_FLOW_DIAGRAMS.md)
- [Project Structure](docs/Project_Structure.md)
- [LM Studio Setup](docs/LM_STUDIO_SETUP.md)

### **Code Examples**
- `Agentic_Excel_Review_Demo.py` - Complete demo script
- `src/run_excel_review_demo.py` - CLI interface
- `src/ui/excel_review_app.py` - Streamlit UI

---

## 🎉 Summary

This system demonstrates **production-grade Agentic AI** for Excel preprocessing:

✅ **Autonomous**: Auto-detects headers, chunks documents, retrieves context  
✅ **Intelligent**: RAG + LLM for semantic understanding  
✅ **Safe**: Read-only, AI_ prefixes, full audit trail  
✅ **Modular**: 11 independent modules working together  
✅ **Explainable**: Confidence scores and reasoning  

**Next Steps**: 
1. Read [AGENTIC_AI_ARCHITECTURE.md](docs/AGENTIC_AI_ARCHITECTURE.md) for deep dive
2. View [AGENTIC_FLOW_DIAGRAMS.md](docs/AGENTIC_FLOW_DIAGRAMS.md) for visual understanding
3. Run the demo to see it in action!

---

**Created by**: Navid Broumandfar  
**Version**: 1.0  
**License**: MIT
