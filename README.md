# Automation

# Project Overview

## Introduction

This project automates the process of generating personalized documents from Notion data and sending emails with these documents to faculty members. It leverages modern technologies to ensure efficiency, scalability, modularity and robustness.

## Technologies Used

- **Ray**: For parallel processing and resource optimization.
- **Prefect**: For workflow orchestration and task management.
- **Google Drive API**: For document upload and management.
- **Notion API**: For data fetching and database updates.
- **Slack**: `Slack-bot` For running the script 
- **SMTP**: For sending emails with attachments.
- **Machine Learning**: 
  - **SpaCy**: For natural language processing and entity recognition
  - **DateParser**: For intelligent date and time parsing
  - **Regex**: For pattern matching and time range detection

## Performance Optimization

- **Parallelism**: Utilizes Ray for parallel task execution, optimizing resource utilization based on available CPU and GPU cores.
- **Concurrency Control**: Adjusts the number of concurrent tasks to prevent memory issues and ensure stable performance.
- **GPU Acceleration**: When available, GPU resources are utilized to accelerate event processing.

## Key Features

- **Data Fetching and Caching**: Utilizes Notion API to fetch faculty data and caches it locally for quick access and change detection.
- **Document Generation**: Generates personalized DOCX documents using templates and faculty data.
- **Google Drive Integration**: Uploads generated documents to Google Drive, converts them to PDF, and reuploads them to a specified folder in the Google drive.
- **Notion Database Update**: Updates the Notion database with links to the uploaded PDFs(Google drive link with  only access to the authenticate user).
- **Email Automation**: Sends personalized emails with PDF attachments to faculty members using SMTP.

## Workflow

1. **Data Fetching and Caching**:
   - The `NotionCacheHandler` fetches data from Notion and caches it locally.
   - It compares the fetched data with the cached data to identify changes.
   - The cahnged data is then send to the `orchestrate_pipeline.`

2. **Document Generation**:
   - For each changed row, the `orchestrate_pipeline` function generates a personalized DOCX document using a template.
   - The `DateTimeExtractor` class uses SpaCy's NLP model to intelligently parse dates and times from event text.
   - Advanced entity recognition identifies temporal expressions and standardizes them into consistent formats.
   - The generated DOCX is uploaded to Google Drive.

3. **PDF Conversion and Upload**:
   - The uploaded DOCX is converted to PDF format.
   - The PDF is reuploaded to a specified Google Drive folder.

4. **Notion Update**:
   - The Notion database is updated with links to the uploaded PDFs.

5. **Email Sending**:
   - The `PDFEmailSender` class reads the faculty data and sends personalized emails with PDF attachments.

## Machine Learning Components

- **Natural Language Processing (NLP)**:
  - Utilizes SpaCy's pre-trained language model (`en_core_web_sm`) for intelligent date and time extraction from text.
  - Implements advanced entity recognition to identify dates and temporal expressions.
  - Combines NLP with regex-based parsing for robust time range detection.
  - Handles various date and time formats, including:
    - Standard time formats (e.g., "6:30am", "18:00")
    - Time ranges (e.g., "6:30am-8pm", "6:30am to 8pm")
    - Natural language date expressions
  - Standardizes extracted dates and times into consistent formats for document generation.

## Error Handling and Logging

- **Robust Error Handling**: Includes comprehensive error handling for network issues, file operations, and API interactions.
- **Detailed Logging**: Logs are generated for tracking progress and debugging issues.

This project demonstrates the integration of multiple APIs and technologies to automate document generation and email sending, which showcase skills in data handling, API integration, and workflow automation. 
