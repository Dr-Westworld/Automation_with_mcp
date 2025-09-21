
`code of this project is on my private repo as this project was for my college`

---

<<<<<<< HEAD
# Automation Project

This project automates document generation from Notion data and sends emails with the generated documents.

## Overview

The automation consists of two main modules:
1. **Orchestration Pipeline** - Fetches data from Notion, generates documents, uploads to Google Drive, convert to pdf by exporting it from drive and then reuploading it  
2. **PDF Email Sender** - Sends emails with the generated PDF documents to faculty members

## Quick Start

1. Configure the `automation_config.json` file with your settings
2. Set the EMAIL_PASSWORD environment variable:
   ```
   # Windows
   set EMAIL_PASSWORD=your_secure_password
   
   # Linux/Mac
   export EMAIL_PASSWORD=your_secure_password
   ```
3. Run the full automation:
   ```
   python coordinator.py
   ```

## Configuration

The `automation_config.json` file controls the automation behavior:

```json
{
    "production_mode": false,
    "keep_ray_alive": true,
    "stop_on_failure": true,
    "modules": {
        "orchestration": {
            "enabled": true,
            "script_path": "e:\\main.py",
            "params": {
                "template_path": "template1.docx",
                "output_folder": "output_files"
            }
        },
        "email_sender": {
            "enabled": true,
            "script_path": "e:\\PDFEmailSender.py",
            "params": {
                "email_sender": "your_email@gmail.com",
                "email_password": ""
            }
        }
    },
    "execution_flow": ["orchestration", "email_sender"],
    "schedule": {
        "run_daily": true,
        "time": "09:00",
        "days": ["Monday", "Wednesday", "Friday"]
    }
}
```

### Configuration Options

| Option | Description |
|--------|-------------|
| `production_mode` | Set to `true` to use Ray's distributed mode for production |
| `keep_ray_alive` | Keep Ray running between modules for faster execution |
| `stop_on_failure` | Stop the automation if a module fails |
| `modules` | Configuration for each module |
| `execution_flow` | Order of module execution |
| `schedule` | Scheduling options (when using a scheduler) |

## Command Line Options

Run specific parts of the automation:

```
# Run with a different config file
python coordinator.py --config my_config.json

# Run only the orchestration module
python coordinator.py --module orchestration

# Run only the email sender module
python coordinator.py --module email_sender

# Run in production mode (uses distributed Ray)
python coordinator.py --production
```

## Performance Optimization

The coordinator automatically optimizes performance:

- Uses Ray local mode for faster development/testing
- Intelligent resource allocation based on available CPU/GPU
- Caching and parallel execution where possible
- Option to keep Ray alive between modules to reduce startup overhead

## Troubleshooting

### Common Issues

1. **Email Sending Fails**
   - Check that EMAIL_PASSWORD is set correctly
   - Verify your email provider allows app access
   - For Gmail, you may need to create an App Password

2. **Ray Initialization Takes Too Long**
   - Use `keep_ray_alive: true` in config
   - Set `production_mode: false` during development
   - Run with the `--module` option to execute only what's needed

3. **Files Not Found**
   - Ensure all paths in config file are correct and absolute
   - Check the output folder exists and is writable

### Logs

Check the logs in the current directory (`automation_log_YYYYMMDD.log`) for detailed error information.

## Security Note

Never store email passwords or sensitive credentials in the config file.
Use environment variables instead:

```
set EMAIL_PASSWORD=your_secure_password
```

## Advanced Usage

### Scheduling with Windows Task Scheduler

1. Create a batch file (run_automation.bat):
   ```
   @echo off
   cd /d path\to\your\project
   set EMAIL_PASSWORD=your_secure_password
   python coordinator.py
   ```

2. Create a task in Windows Task Scheduler to run this batch file on your schedule.

### Scheduling with Cron (Linux/Mac)

Add to crontab:
```
0 9 * * 1,3,5 cd /path/to/project && export EMAIL_PASSWORD=your_secure_password && python coordinator.py
```

## Workflow

The project consists of two main modules:

1. **Orchestration Pipeline** (`main.py`):
   - Fetches data from Notion using the `NotionDataFetcher` and `NotionCacheHandler` classes.
   - Generates personalized documents (DOCX) using a template and faculty data.
   - Uploads the generated DOCX files to Google Drive using the `GoogleDriveUploader` class.
   - Converts the DOCX files to PDF format and reuploads them to a specified Google Drive folder.
   - Updates the Notion database with links to the uploaded PDFs.

2. **PDF Email Sender** (`PDFEmailSender.py`):
   - Reads faculty data from a cache file.
   - Sends personalized emails with PDF attachments to faculty members.
   - Utilizes Ray for parallel email sending and includes network retry logic for robust delivery.

### Workflow Steps

1. **Data Fetching and Caching**:
   - The `NotionCacheHandler` fetches data from Notion and caches it locally.
   - It compares the fetched data with the cached data to identify changes.

2. **Document Generation**:
   - For each changed row, the `orchestrate_pipeline` function in `main.py` generates a personalized DOCX document using a template.
   - The generated DOCX is uploaded to Google Drive.

3. **PDF Conversion and Upload**:
   - The uploaded DOCX is converted to PDF format.
   - The PDF is reuploaded to a specified Google Drive folder.

4. **Notion Update**:
   - The Notion database is updated with links to the uploaded PDFs.

5. **Email Sending**:
   - The `PDFEmailSender` class reads the faculty data and sends personalized emails with PDF attachments.

### Performance Optimization

- The orchestration pipeline uses Ray for parallel processing, optimizing resource utilization based on available CPU and GPU cores.
- The `process_small_batch` function processes documents in parallel with controlled concurrency, ensuring efficient handling of large datasets.

### Error Handling

- The workflow includes robust error handling for network issues, file operations, and API interactions.
- Logs are generated for tracking progress and debugging issues.

### Dependencies

- **Orchestration Pipeline**:
  - `prefect`
  - `ray`
  - `googleapiclient`
  - `notion_client`
  - `docxtpl`

- **PDF Email Sender**:
  - `ray`
  - `pandas`
  - `smtplib`
  - `ssl`
  - `email`
  - `os`
  - `json`
  - `logging`

(Note: This list might not be exhaustive, please refer to `requirements.txt` for the complete list of dependencies.)

# PDF Email Sender

This script automates the process of sending personalized emails with PDF attachments to faculty members.

## Features

- Sends personalized emails based on faculty data.
- Attaches generated PDF files to emails.
- Utilizes Ray for parallel email sending.
- Includes network retry logic for robust email delivery.

## Setup

1.  **Clone the repository:**

    ```bash
    git clone <repository_url>
    cd <repository_directory>
    ```

2.  **Install dependencies:**

    ```bash
    pip install -r requirements.txt
    ```

3.  **Environment Variables:**

    Ensure you have the following environment variables set or replace placeholders in the script:
    -   `EMAIL_SENDER`: Your email address (e.g., `your_email@gmail.com`)
    -   `EMAIL_PASSWORD`: Your email password or app-specific password.

    **Note:** If using Gmail, you might need to generate an [App password](https://support.google.com/accounts/answer/185833?hl=en) if 2-Step Verification is enabled.

4.  **Configuration Files:**

    -   `constants.py`: Define `CACHE_FILE` (path to your JSON data file) and `OUTPUT_FOLDER` (directory where PDFs are stored).

## Usage

1.  **Prepare your data:**

    Ensure your `CACHE_FILE` (a JSON file) contains a list of dictionaries, where each dictionary represents a faculty member with at least the following keys:
    -   `Faculty`: Name of the faculty member (used for PDF filename).
    -   `Post`: Faculty's post/role.
    -   `Department`: Faculty's department.
    -   `Email`: Faculty's email address.

    Example `cache.json`:
    ```json
    [
        {
            "Faculty": "Dr. John Doe",
            "Post": "Professor",
            "Department": "Computer Science",
            "Email": "john.doe@example.com"
        },
        {
            "Faculty": "Dr. Jane Smith",
            "Post": "Associate Professor",
            "Department": "Physics",
            "Email": "jane.smith@example.com"
        }
    ]
    ```

2.  **Place PDF files:**

    Ensure that the personalized PDF files (named as `<Faculty_Name>.pdf`, e.g., `Dr. John Doe.pdf`) are present in the `OUTPUT_FOLDER`.

3.  **Run the script:**

    ```bash
    python PDFEmailSender.py
    ```

    The script will read the faculty data, prepare personalized emails, attach the corresponding PDFs, and send them out. Progress and errors will be logged to the console.

## Modules and Functions

-   `PDFEmailSender.py`:
    -   `send_email_with_retry(msg, email_sender, email_password)`: Sends email with network retry logic.
    -   `PDFEmailSender` class:
        -   `__init__(self, pdf_folder)`: Initializes the sender with the PDF folder.
        -   `prepare_email_body(data)`: Static method to prepare the email body.
        -   `send_email.remote(data, email_sender, email_password)`: Ray remote function to send personalized emails with PDF attachments.

-   `constants.py`: Defines `CACHE_FILE` and `OUTPUT_FOLDER` paths.

-   `faculty_data_utils.py`: Contains `extract_faculty_data` for processing faculty information.

-   `network_utils.py`: Provides `get_network_handler` and `NetworkError` for network operations and error handling.

## Error Handling

-   The script logs errors for missing PDF files and network issues.
-   Ray is initialized and shut down gracefully.

## Dependencies

-   `ray`
-   `pandas`
-   `smtplib`
-   `ssl`
-   `email`
-   `os`
-   `json`
-   `logging`

(Note: This list might not be exhaustive, please refer to `requirements.txt` for the complete list of dependencies.)

## Ray and Prefect Usage

### Parallelism and Pipelining

- **Ray**: The project leverages Ray for parallel processing, allowing tasks to be executed concurrently. This is particularly useful for handling large datasets and optimizing resource utilization.
  - **Parallel Execution**: Functions like `extract_checked_events`, `generate_docx`, and `process_document_pipeline_ray` are decorated with Ray to enable parallel execution.
  - **Resource Optimization**: The `process_small_batch` function dynamically adjusts the number of concurrent tasks based on available CPU and GPU resources, ensuring efficient processing.

- **Prefect**: Prefect is used for workflow orchestration, providing a robust framework for managing task dependencies and retries.
  - **Task Management**: Tasks are defined with retry logic and caching to handle transient failures and improve performance.
  - **Flow Orchestration**: The `orchestrate_pipeline` function uses Prefect's flow decorator to manage the overall workflow, ensuring tasks are executed in the correct order.

### Network Handling

- **Network Resilience**: The project includes a `network_handler` that provides context for network operations, ensuring robust handling of network issues.
  - **Retry Logic**: Network operations are wrapped in a context that includes retry logic, allowing the system to recover from transient network failures.
  - **Error Logging**: Detailed logging is implemented to track network errors and facilitate debugging.

### Performance Considerations

- **Concurrency Control**: The number of concurrent tasks is controlled to prevent memory issues and ensure stable performance, especially for large batches of documents.
- **GPU Acceleration**: When available, GPU resources are utilized to accelerate event processing, enhancing overall performance.

### Dependencies

- **Ray**: Used for parallel task execution.
- **Prefect**: Used for workflow orchestration and task management.

(Note: This list might not be exhaustive, please refer to `requirements.txt` for the complete list of dependencies.) 

[🎬 Watch the demo](./2025-08-12 23-12-36.mkv)
`It is just a half part of the project it is not the entire project.
This project is at is limit as of september 2025 a feature which was suppose to give the user the simplicity in use is now made a paid feature which at the time of the project was a free.
Rest majority of the code is in working as of september.`
---

---
`this is now a software which works without that one feature which use to give the users the insane simplicity because of the the automation webhook which which was free since 2021 till the august of 2025 after the increase in ai this feature was made paid.`

---
