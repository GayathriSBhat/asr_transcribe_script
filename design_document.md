# 🧾 Design Document: Gemini-Based ASR Evaluation & Enrichment Pipeline (Notebook Implementation)

---

## **1. Objective**

Build a **Jupyter Notebook-based pipeline** that:

- Reads a multi-sheet Excel file (`MCF_Languages.xlsx`)
- Processes audio files organized by language folders:
  - /audio_files/<language_name>/
- Uses **Gemini 3.1 Pro** for:
- Transcription (same language)
- Translation (to English)
- Match scoring (semantic + lexical + hallucination-aware) 
- Error analysis (2-line explanation)
- Writes results **in-place (if it already exists just continue filling other fields, need not refill)** into the same Excel file

---

## **2. Inputs**

### **2.1 Excel File**

- File: `MCF_Languages.xlsx`
- Contains:
- Multiple sheets (each sheet = language)
- Required columns:
  - `Audio_Samples`
  - `Original_Transcript`

---

### **2.2 Audio Files**

##### Directory Structure

/audio_files/

-> Hausa/

> -> sample1.wav

> ->sample2.mp3

Igbo/

> sample1.wav

### **Supported Formats**

- `.wav`
- `.mp3`
- `.m4a`
- `.flac`

---

## **3. Outputs**

For each sheet, populate:

| Column Name                         | Description                       |
| ----------------------------------- | --------------------------------- |
| `Gemini_3.1_Pro_Trancript`          | Transcription (original language) |
| `Gemini_3.1_Pro_Translation`        | English translation               |
| `Gemini_3.1_Pro_Match_Percentage`   | Final score (%)                   |
| `Gemini_3.1_Pro_WER`                | Word Error Rate                   |
| `Gemini_3.1_Analysis`               | 2-line explanation                |

---

## **4. System Architecture**

### **4.1 High-Level Flow**

>  /audio_files/<sheet_name>/



---



### **5.2 Audio Mapping**

- Use:

  > Audio_Samples → filename
  >
- Construct path:

  > /audio_files/`<language>`/<Audio_Samples>
  >

  ---

  ### **5.3 Row Processing Unit**

  Each row:


  ```json
  {
  "language": "Hausa",
  "audio_file": "Sample1.wav",
  "original_transcript": "..."
  }
  ```

  ## **6. Agent-Based Design**

  ---

  ### **6.1 Transcription Agent**

  **Input:**

  * Audio file
  * Language

  **Process:**

  * Send audio to Gemini
  * Prompt:

"Transcribe the audio exactly in its original language.
Do not translate. Do not summarize."

     **Output:**

  * Raw transcription
    ---

  ### **6.2 Translation Agent**

  **Input:**

  * Transcription

  **Prompt:**

" Translate the following text to English.
Preserve meaning exactly. Do not summarize."

  **Output:**

  * English translation

---

**(A) Semantic Similarity**

* Use embeddings
* Use The Token-Level Approach (e.g., BERTScore)
---

#### **(B) Word Error Rate (WER)**

Formula:

WER=(S+D+I)/N

Where:

* S = substitutions
* D = deletions
* I = insertions
* N = total words in original

#### apply the clipping function

clipped_WER=min(WER, 1.0)

** clip

clipped_WER= min(0.8,0.1)

---

#### **(C) Sequential Similarity **

Longest Common Subsequence (ROUGE-L) 


#### **(D) Hallucination Score (LLM-Based)**

Prompt:

"Compare the original transcript and the ASR output.

Score hallucination from 0 to 1:
0 = no hallucination
1 = severe hallucination

Return only a number."

---

#### **(E) Final Match Score**

match_score =
    (0.35 × semantic_similarity)   # e.g. sentence-transformers / BERTScore
+ (0.30 × (1 - clipped_WER))    # clamp WER to [0,1]
+ (0.20 × sequence_similarity)  # e.g. ROUGE-L or sequence matcher ratio
+ (0.15 × (1 - hallucination_score))

clipped_WER = min(WER, 1.0)       # prevents negative contributions

---

### **6.4 Analysis Agent**

**Input:**

* Original transcript
* Model output
* Scores

**Prompt:**

"Explain why the transcription quality is low or high.

Focus on:
- missing words
- incorrect meaning
- hallucinations

Answer in exactly 2 lines."

**Output:**

* Short explanation

---

## **7. Pipeline Workflow**

### **Step-by-Step Execution**

1. Load Excel file
2. Iterate through each sheet
3. For each row:

* Identify language
* Locate audio file
* Run:
  *  Transcription
  *  Translation
  *  Scoring
  *  Analysis

1. Update DataFrame
2. Save Excel (overwrite/ don't create new sheet, update in same sheet)

---

## **8. Notebook Structure**

### **Cell Layout**

1. Configuration
2. Imports
3. Gemini Initialization
4. Utility Functions
5. Agents
6. Core Pipeline
7. Excel Processing
8. Execution

---

## **9. Configuration**

AUDIO_ROOT = "./audio_files"
EXCEL_PATH = "./MCF_Languages.xlsx"

SUPPORTED_FORMATS = [".wav", ".mp3", ".m4a", ".flac"]

MAX_RETRIES = 1
PROCESS_MODE = "sequential"

---

## **10. Excel Handling**

### **Behavior**

* Preserve all sheets
* Don't create new sheet update in same sheet the columns that are not filled.
* Create missing columns automatically

---

## **11. Error Handling**

| Scenario            | Action           |
| ------------------- | ---------------- |
| Missing audio       | Skip row + log   |
| API failure         | Retry once       |
| Empty transcription | Flag in analysis |
| Invalid sheet       | Skip             |

---

## **12. Performance Constraints**

* Sequential processing (low API usage)
* No parallelism
* Minimal retries (1)

---

## **13. Extensibility**

Future models can be added easily:

TRANSCRIPTION_MODELS = ["Gemini-3.1-Pro-Preview", "Gemini-2.5-Pro", "OmniASR7B", "MMS"]

TRANSLATION_MODEL=  ["Gemini-2.5-Pro"]

MATCH_SCORING = 

Add later:

* Meta MMS

---

## **14. Logging**

Track:

* Missing files
* Failed API calls
* Low match scores (<50%)

---

## **15. Assumptions**

* Sheet names match folder names
* Audio filenames match Excel entries
* Gemini supports audio input
* Transcripts are reasonably aligned

---

## **16. Example Transformation**

### **Input**

| Audio       | Original   |
| ----------- | ---------- |
| sample1.wav | Hausa text |

---

### **Output**

| Column                          | Value                              |
| ------------------------------- | ---------------------------------- |
| Gemini_3.1_Pro_Trancript        | Hausa transcription                |
| Gemini_3.1_Pro_Translation      | English                            |
| Gemini_3.1_Pro_Match_Percentage | 87.3                               |
| Gemini_3.1_Pro_WER              | 0.11                               |
| Gemini_3.1_Analysis             | Minor omissions, meaning preserved |

---

## **17. Future Enhancements**

* Parallel execution
* Caching results
* UI dashboard
* Multi-model comparison
* Confidence scoring

---

## ✅ **Status**

| Feature                  | Status |
| ------------------------ | ------ |
| Multi-sheet Excel        | ✅     |
| Multi-language support   | ✅     |
| Gemini ASR               | ✅     |
| Translation              | ✅     |
| Match scoring            | ✅     |
| Analysis                 | ✅     |
| Notebook-friendly design | ✅     |
