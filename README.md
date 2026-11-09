
# DwellTime-Documentation

## Retail Dwell Time Analytics Application

### 1. Overview

Dwell Time Analytics is an AI-powered retail video analytics application designed to monitor customer movement, measure time spent in different store zones, and generate actionable business insights.

The application integrates Computer Vision, Neural Networks, Vision Language Models (VLM), Snowflake Cortex AI, and Streamlit to provide intelligent retail analytics.

### 2. Key Features

- Customer detection and movement tracking
- Customer dwell time calculation
- Zone-wise customer activity analysis
- Customer entry and exit monitoring
- Frame-level video analysis and summarization
- Vision Language Model-based Visual Question Answering (VQA)
- AI-generated retail insights using Snowflake Cortex
- Interactive analytics dashboard using Streamlit

### 3. Technology Stack

| Component | Technology |
|-----------|------------|
| Programming Language | Python |
| Application Framework | Streamlit |
| Computer Vision | OpenCV |
| AI/ML | Deep Learning, Neural Networks |
| Video Intelligence | Detection and Tracking Models |
| Visual Question Answering | Vision Language Model (VLM) |
| Cloud Data Platform | Snowflake |
| Generative AI | Snowflake Cortex AI |
| Data Processing | Python, Pandas, Snowpark |
| Database | Snowflake SQL |

### 4. Application Architecture

Video Input
    |
    v
Frame Extraction & Preprocessing
    |
    v
Neural Network Inference
    |
    +-- Customer Detection
    +-- Customer Tracking
    +-- Zone Identification
    +-- Dwell Time Calculation
    |
    v
Vision Language Model (VQA)
    |
    +-- Frame Understanding
    +-- Visual Question Answering
    +-- Scene Summarization
    |
    v
Snowflake Database
    |
    +-- Detection Data
    +-- Tracking Data
    +-- Frame Summaries
    +-- Zone Analytics
    +-- Dwell Time Records
    |
    v
Snowflake Cortex AI
    |
    +-- Natural Language Insights
    +-- Analytics Summarization
    |
    v
Streamlit Analytics Dashboard

### 5. AI and Neural Network Components

**Customer Detection**

Neural networks identify customers within video frames and generate object-level detection information.

**Customer Tracking**

Tracking models associate detected customers across consecutive frames to analyze movement patterns.

**Dwell Time Analytics**

Calculates the duration customers spend within predefined retail zones using tracking information and timestamps.

**Vision Language Model (VLM)**

Enables intelligent interpretation of retail video frames through Visual Question Answering.

Example questions:
- How many customers are visible?
- Which store zone is most crowded?
- What activities are occurring in the scene?
- Summarize customer activity in this frame.

### 6. Snowflake Integration

Snowflake serves as the central platform for storing processed video analytics data and generating AI-powered insights.

**Database:** RETAIL_ANALYTICS

**Schema:** DWELL_TIME

#### Database Tables

| Table | Description |
|-------|-------------|
| VIDEO_METADATA | Video information, source, duration and processing status |
| FRAME_METADATA | Frame ID, timestamp and processing details |
| DETECTION_DATA | Customer detection results and bounding-box coordinates |
| TRACKING_DATA | Customer tracking IDs, positions and timestamps |
| ZONE_DWELL_TIME | Zone entry, exit and dwell duration |
| FRAME_SUMMARY | VLM-generated frame descriptions |
| VQA_RESULTS | Visual questions and AI-generated answers |
| CORTEX_INSIGHTS | AI-generated analytics summaries |

#### Snowflake Services

- Snowflake SQL for structured analytics storage
- Snowpark Python for data processing
- Snowflake Cortex AI for natural-language summaries and insights
- Snowflake Connector for Python for database connectivity

### 7. Streamlit Dashboard

The Streamlit application provides:

- Video upload and processing
- Customer detection visualization
- Tracking and movement analysis
- Zone-wise dwell time analytics
- Customer traffic statistics
- Frame-level AI summaries
- VQA interaction
- Snowflake-powered analytics
- AI-generated retail insights

### 8. Installation and Setup

#### Step 1: Clone Repository

```bash
git clone <repository-url>
cd DwellTime-Documentation
```

#### Step 2: Create Virtual Environment

```bash
python -m venv venv
```

#### Step 3: Activate Virtual Environment

Windows:

```bash
venv\Scripts\activate
```

Linux/macOS:

```bash
source venv/bin/activate
```

#### Step 4: Install Dependencies

```bash
pip install -r requirements.txt
```

#### Step 5: Configure Snowflake

Create a `.env` file with the following configuration:

```env
SNOWFLAKE_ACCOUNT=your_account
SNOWFLAKE_USER=your_username
SNOWFLAKE_PASSWORD=your_password
SNOWFLAKE_WAREHOUSE=your_warehouse
SNOWFLAKE_DATABASE=RETAIL_ANALYTICS
SNOWFLAKE_SCHEMA=DWELL_TIME
```

#### Step 6: Run Streamlit Application

```bash
streamlit run app.py
```

Open the application at:

http://localhost:8501

### 9. Application Workflow

1. Upload retail surveillance video.
2. Extract and preprocess video frames.
3. Perform customer detection and tracking.
4. Calculate customer dwell time across store zones.
5. Generate frame summaries and VQA responses.
6. Store processed analytics in Snowflake.
7. Generate business insights using Snowflake Cortex AI.
8. Visualize results through the Streamlit dashboard.

### 10. Retail Business Applications

- Customer engagement analysis
- Store layout optimization
- High-traffic zone identification
- Customer movement analysis
- Product placement evaluation
- Staff allocation planning
- Retail operational intelligence

### 11. Conclusion

Dwell Time Analytics combines Computer Vision, Neural Networks, Vision Language Models, Snowflake Cortex AI, and Streamlit to transform retail surveillance videos into structured analytics and actionable business intelligence.

The solution enables retailers to understand customer behavior, optimize store operations, and make data-driven decisions.

---

**Project:** Retail Dwell Time Analytics  
**Platform:** Python | AI/ML | Snowflake | Streamlit  
**Application Type:** AI-Powered Retail Video Analytics
