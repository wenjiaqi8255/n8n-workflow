<img width="1222" alt="image" src="https://github.com/user-attachments/assets/d5c7ecb3-300f-4d26-86a2-90fd721e91bc" />


## Automated Job Hunting Assistant (LinkedIn + Indeed): Get Best Job Matches via Email + Generate Cover Letters + Save to Notion

**Author**: Wen
**Email address**: wenjiaqi8255@outlook.com

**Description**: 
This n8n workflow is designed to automate the process of finding and pushing the best job recommendations. It retrieves your resume, scrapes job listings from LinkedIn and Indeed, uses AI (Google Gemini) to match jobs with your resume, and then sends the best job recommendations to you via email. The workflow runs automatically according to schedule.

**Core Features**:
- Automatic resume and job matching
- Personalized cover letter generation
- Scheduled email delivery of high-match jobs
- Notion integrated job management
- Multi-platform job data aggregation (LinkedIn, Indeed)

**Credit**: Tianyi tianyi.datascience@gmail.com

### Schedule Trigger
This node triggers the workflow to run automatically at the specified time. Default setting is to trigger at 8:00 AM daily, you can adjust the frequency according to your needs.
Note: Timezone can be set in n8n Settings

### Input
**Key Configuration Node**
This sets important variables used throughout the workflow:
- **ApifyAPIKey**: API key for accessing Apify services
- **Preference**: Your job preferences, affecting AI matching results
- **EmailAddressToReceiveJobRecommendations**: Email to receive job recommendations
- **recommendedJobCount**: Number of job recommendations per time
- **first_timeout**: First API request timeout (seconds)
- **request_interval**: Retry request interval time (seconds)

### DownloadResume
Downloads your resume PDF file from Google Drive. Make sure to upload your resume to Google Drive and get the file ID.

### Extract Information from Resume PDF
Extracts text content from the resume PDF, preparing for subsequent AI matching.

### SetResumeField
Sets the extracted resume text as a variable for use in later steps of the workflow.

### start_apify_run_linkedin
Starts the Apify crawler to scrape job information from LinkedIn.
Make sure to set the correct search URL and parameters in jsonBody.

### Check Run Status / Get Dataset Items
These nodes check the crawler's running status and get the result data.
Includes error handling and retry mechanisms to ensure reliable job information acquisition.

### start_apify_run_indeed
Starts the Apify crawler to scrape job information from Indeed.
Ensure to set the correct search URL and parameters in jsonBody.

### normalize_job_data
Standardizes job data obtained from different platforms, ensuring a unified data structure.
Handles field mapping and default value settings.

### Merge
Merges job data from LinkedIn and Indeed, creating a unified job list.

### Supabase / Supabase1
Stores the crawled job data in the Supabase database for persistent storage and deduplication.
Ensures the same job information is not processed repeatedly.

### filter_by_status / get_unique_jobs
Filters out new and unprocessed job information, ensuring job ID uniqueness.

### stringify_json / Aggregate
Prepares job data in a format that AI can process and aggregates all relevant information.

### AI Agent: Find Best-matched jobs
**Core Matching Node**

**Input**:
- **scraped_job_data**: All job information scraped from LinkedIn and Indeed
- **user_cv**: User resume text (extracted from PDF)
- **user_preferences**: User job preference settings (such as location, industry, salary expectations, etc.)
- **recommendedJobCount**: Number of jobs to match

**Output**:
Structured list containing detailed information of recommended jobs:
- **database_id**: Job record ID
- **Company Name**: Recruiting company name
- **Job Title**: Position name
- **Industry**: Industry field
- **Reason for Match**: Explanation of match reason (1-2 sentences explaining why this job matches the user)
- **Application URL**: Application link
- **Location**: Work location
- **Flexibility**: Work arrangement (remote, hybrid, on-site, etc.)
- **Salary Range**: Salary range (if available)

### update_job_status
Updates the status of processed jobs to avoid recommending the same job repeatedly.

### extract_matched_jobs / extract_update_operation
Extracts the final recommended jobs from AI matching results and database records that need to be updated.

### generate_html_template
Converts matching results into an attractive HTML email template, including job details and direct application links.

### Email the top job recommendations
Sends the best matched jobs to you via email, including job title, company, location, salary, and application links.

## Workflow Setup Guide
### Please follow these steps to set up the workflow:

### 1. Configure the input node:
Open the "Input" node in the workflow.
Fill in the required fields:
- **ApifyAPIKey**: Enter your Apify API key (Login to Apify: https://www.apify.com?fpr=oljm1 -> Settings -> API & Integration)
- **Preference**: Specify your job preferences (such as "no German requirement, high pay")
- **EmailAddressToReceiveJobRecommendations**: Provide the email address where you want to receive job recommendations
- **recommendedJobCount**: Set the number of job recommendations per time (default is 10)

### 2. Configure LinkedIn job search URL:

- Open LinkedIn Jobs search page in your browser's privacy/incognito window: https://www.linkedin.com/jobs/search/
- Use LinkedIn's search interface to set your needed filters (keywords, location, experience level, etc.). **Be sure to add the "Past 24 hours" filter**
- Copy the complete URL from the browser's address bar
- Open the "start_apify_run_linkedin" node in the workflow
- Find the "urls" array in the "jsonBody" parameter:
  ```json
  "urls": [
      "https://www.linkedin.com/jobs/search/..."
  ]

Replace the existing URL ("https://www.linkedin.com/jobs/search/...") with the complete URL you copied from the privacy window. Make sure the URL is enclosed in double quotes

3. Configure Indeed job search (optional):

Similarly, you can configure Indeed search parameters in the "start_apify_run_indeed" node
Customize search keywords, location, and other parameters to match your job search needs

4. Upload your resume:

Upload your resume (PDF format) to Google Drive
Update the file ID in the "DownloadResume" node to point to your resume

5. Configure Notion integration (optional):

If you want to use Notion to track application progress, make sure to set up the correct Notion Database ID
Note: It's recommended to create two databases:

Job Tracker database
Properties: Company Name(title), Job Title, Industry, Application URL, Location, Flexibility, Salary Range, Status

Cover Letter database
Properties: Title(title), Company Name

6. Database configuration:

Ensure that a table named job_listings is created in Supabase or your chosen database system
The table structure should include the following fields (can be created using the following SQL):

```sql
CREATE TABLE job_listings (
  id TEXT PRIMARY KEY,
  company_name TEXT NOT NULL,
  job_title TEXT NOT NULL,
  application_url TEXT,
  location TEXT,
  description TEXT,
  industry TEXT,
  flexibility TEXT,
  salary_range TEXT,
  status TEXT,
  source TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  processed_at TIMESTAMP WITH TIME ZONE,
  reason_for_match TEXT
);
```

### Webhook
Receives cover letter generation requests, processes cover letter generation for specific job IDs.

### get_job_id / supabase_get
Retrieves the job ID from the request and finds the corresponding job description in the database.

### create_new_job_listing
Creates a record in the job tracker database.

### DownloadResume
Downloads your resume PDF file from Google Drive. Make sure to upload your resume to Google Drive and get the file ID.

### Extract Information from Resume PDF
Extracts text content from the resume PDF, preparing for subsequent AI cover letter generation.

### AI Agent (Cover Letter Generation)
**Personalized Cover Letter Generator**
Uses AI to generate tailored professional cover letters based on your resume and specific job descriptions.
Cover letter style features:
- Company-needs focused, highlighting how you solve specific problems
- Concise and direct expression
- Authentic personalized content, avoiding clichés
- Provides strong evidence for key job requirements
- Quality over quantity, providing powerful examples rather than vague achievements

### Gmail / Markdown
Formats cover letter content and sends it to you via email.

### create_new_cover_letter / append_blocks
Stores the generated cover letter in a Notion database for easy viewing and editing.

### Input
**Key Configuration Node**
Sets the important variable used in the workflow: **EmailAddressToReceiveCoverLetter**: Email to receive the cover letter

### Note on costs
It is recommended to pin the data after running to save costs when you're debugging.
