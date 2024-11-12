flowchart TD

%% Initialization and Configuration
A[Start Process - background_processor.py] --> B[Load Gmail API Credentials]
B --> C[Load Existing Email Data - incremental_email_handler.py]
C --> D{Are Credentials Valid?}
D -- Yes --> E[Initialize Gmail API Service - parsegmail.py]
D -- No --> F[Obtain New Credentials]

F --> E
E --> G[Construct Query for Gmail API - parsegmail.py]
G --> H[Load Filter Senders List - gmailextract.py]
H --> I[Begin Fetching Emails in Batches - gmailextract.py]

%% Fetching and Processing Emails
I --> J{Messages Found?}
J -- No --> Z[End Process - No New Emails]
J -- Yes --> K[Process Each Message - gmailextract.py]

%% Message Processing Steps
K --> L[Parse Headers (Sender, Date, Subject) - gmailextract.py]
L --> M[Filter by Sender - filter_senders.json]
M --> N[Decode Email Content - parsegmail.py]
N --> O[Structure Email Body - gmailextract.py]

%% Transformation and Error Points
O --> P[Clean HTML Content - decode_and_extract_text()]
P --> Q[Structure Text Data - EmailCleaner in gmailextract.py]
Q --> R{Email Before Cutoff Date?}
R -- Yes --> S[Skip Email, Continue]
R -- No --> T[Add Processed Email to Batch]

%% Error Handling in Processing
K --> AE[Error in Parsing Headers?]
AE --> AF[Log Parsing Error - gmail_filter.log]
AF --> K
P --> AG[Error in Cleaning HTML?]
AG --> AH[Log HTML Cleaning Error - gmail_filter.log]
AH --> P

%% Batch Processing and MongoDB Storage
T --> U{Batch Size >= Threshold?}
U -- Yes --> V[Save Batch to JSON - incremental_email_handler.py]
V --> W[Backup Existing JSON - incremental_email_handler.py]
W --> X[Save Batch to MongoDB - mongo_loader.py]
X --> Y[Clear Processed Batch from Memory]

%% MongoDB Transformation and Error Points
X --> AI[Error in MongoDB Insertion?]
AI --> AJ[Log MongoDB Error - background_processor.log]
AJ --> I
Y --> AL[Update Progress Log - background_processor.py]
AL --> AM[Send Notification if Threshold Reached - background_processor.py]
AM --> I

%% Finalization
Z --> AN[Save Final Processing Logs - gmail_processing_logs.json]
AN --> AO[End Process - Save Complete]

%% Detailed Error Handling and Transformation Points
subgraph "Error and Transformation Points"
    AG[HTML Parsing Error] --> AG_DESC((Why: Invalid HTML structure or unexpected tags))
    AE[Parsing Error in Headers] --> AE_DESC((Why: Missing or malformed headers))
    AI[MongoDB Insertion Error] --> AI_DESC((Why: Connectivity or duplicate key issues))
    AP[JSON Save Error] --> AP_DESC((Why: File permissions or format issue))
    AQ[Transformation Errors in Email Body] --> AQ_DESC((Why: Non-UTF8 encoding or special characters))
end

%% Final Verification and Sync
X --> AR[Verify MongoDB Sync - verify_mongo_data.py]
AR --> AS{Is MongoDB and JSON Synced?}
AS -- Yes --> AT[Final Verification Success - Log Completion]
AS -- No --> AU[Send Discrepancy Notification - background_processor.py]

%% Possible Discrepancies and Issues
subgraph "Potential Issues and Discrepancies"
    AD_DESC((Possible Issues: \n- Batch size not reached often\n- JSON and MongoDB sync discrepancies))
    AQ_DESC((Transformation Errors in HTML or JSON structures))
    AP_DESC((JSON Save Error: Inconsistent file format, invalid characters))
    AR_DESC((Verification Issues: Missing data or partial saves))
end
