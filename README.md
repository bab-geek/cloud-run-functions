# Cloud Run Functions: Command Line

## Project Description
This project demonstrates how to create, deploy, and test a Cloud Run function using the Google Cloud Shell command line. Cloud Run functions are event-driven serverless functions that execute code in response to events such as HTTP requests, Pub/Sub messages, or Cloud Storage changes. This lab guides you through building a simple "Hello World" function triggered by Pub/Sub events.

## Lab Objectives
- Create a Cloud Run function that responds to Pub/Sub events
- Deploy the function using Google Cloud CLI commands
- Test the function by publishing messages to a Pub/Sub topic
- View and analyze function execution logs
- Understand the serverless function lifecycle on Google Cloud Platform

## Solution/Approach
The solution involves creating a Node.js function that processes Pub/Sub messages. The approach follows these steps:

1. **Function Creation**: Develop a simple Node.js function using the Functions Framework that decodes and logs Pub/Sub messages
2. **Deployment Configuration**: Use `gcloud` commands to deploy the function with Pub/Sub trigger configuration
3. **Testing**: Publish test messages to the Pub/Sub topic to trigger function execution
4. **Monitoring**: Use Cloud Logging to view function execution logs and verify proper operation

## Key Configurations

### Function Code (`index.js`)
```javascript
const functions = require('@google-cloud/functions-framework');

// Register a CloudEvent callback with the Functions Framework
functions.cloudEvent('helloPubSub', cloudEvent => {
  const base64name = cloudEvent.data.message.data;
  const name = base64name
    ? Buffer.from(base64name, 'base64').toString()
    : 'World';
  console.log(`Hello, ${name}!`);
});
```

### Package Configuration (`package.json`)
```json
{
  "name": "gcf_hello_world",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/functions-framework": "^3.0.0"
  }
}
```

### Deployment Command
```bash
gcloud functions deploy nodejs-pubsub-function \
  --gen2 \
  --runtime=nodejs20 \
  --region=REGION \
  --source=. \
  --entry-point=helloPubSub \
  --trigger-topic cf-demo \
  --stage-bucket PROJECT_ID-bucket \
  --service-account cloudfunctionsa@PROJECT_ID.iam.gserviceaccount.com \
  --allow-unauthenticated
```

### Testing Command
```bash
gcloud pubsub topics publish cf-demo --message="Cloud Function Gen2"
```

## Architecture Overview
```
┌─────────────────┐     Pub/Sub Message    ┌────────────────────┐
│   Publisher     │ ──────────────────────> │  Cloud Run Function│
│ (gcloud command)│                        │  (helloPubSub)     │
└─────────────────┘                        └────────────────────┘
                                                │
                                                ▼ Writes to logs
                                        ┌────────────────────┐
                                        │   Cloud Logging    │
                                        └────────────────────┘
```

## Prerequisites & Setup
1. **Google Cloud Account**: Access to Google Cloud Platform with billing enabled
2. **Google Cloud SDK**: Installed and configured (`gcloud` command line tool)
3. **Cloud Shell**: Recommended environment for execution
4. **Project Permissions**: Appropriate IAM roles (Cloud Functions Admin, Pub/Sub Publisher, etc.)

## Step-by-Step Implementation

### 1. Environment Setup
```bash
# Set the default region
gcloud config set run/region us-central1

# Create project directory
mkdir gcf_hello_world && cd $_
```

### 2. Function Development
- Create `index.js` with the Cloud Run function code
- Create `package.json` with dependencies
- Install dependencies: `npm install`

### 3. Function Deployment
- Deploy using `gcloud functions deploy` with Gen2 architecture
- Configure Pub/Sub trigger topic (`cf-demo`)
- Set appropriate service account and permissions

### 4. Testing & Validation
- Publish test message to Pub/Sub topic
- Verify function execution via logs
- Check function status and URL endpoint

## Expected Output
When the function executes successfully, you should see log entries similar to:
```
LEVEL: I
NAME: nodejs-pubsub-function
EXECUTION_ID: h4v6akxf4sxt
TIME_UTC: 2024-08-05 15:15:25.723
LOG: Hello, Cloud Function Gen2!
```
<img width="1452" height="663" alt="Screenshot 2026-01-15 173534" src="https://github.com/user-attachments/assets/d645b772-9d9a-4beb-ad69-35eabcb02f1a" />

<img width="1465" height="652" alt="Screenshot 2026-01-15 173453" src="https://github.com/user-attachments/assets/fb97be5c-007b-481e-bd7a-cd886a10a8ad" />
<img width="1464" height="685" alt="Screenshot 2026-01-15 172653" src="https://github.com/user-attachments/assets/27f1b10c-9446-45d5-906d-f0056501ecb4" />
<img width="1409" height="657" alt="Screenshot 2026-01-15 172429" src="https://github.com/user-attachments/assets/288b8642-cf52-400c-93e9-83638d6395d0" />
<img width="1427" height="649" alt="Screenshot 2026-01-15 172414" src="https://github.com/user-attachments/assets/10083a90-d1da-4c84-a525-3f17b619a14c" />

<img width="1081" height="688" alt="Screenshot 2026-01-15 172215" src="https://github.com/user-attachments/assets/fcaaaa41-2822-4cce-88f3-4e5957513e40" />


## Best Practices Demonstrated
1. **Separation of Concerns**: Function code separate from deployment configuration
2. **Environment Variables**: Using project-specific configurations
3. **Logging**: Structured logging for monitoring and debugging
4. **Security**: Service account configuration and authentication settings
5. **Resource Management**: Clean project structure and naming conventions

## Troubleshooting Tips
1. **Deployment Failures**: Check IAM permissions and service account configuration
2. **Function Not Triggering**: Verify Pub/Sub topic exists and messages are being published
3. **Logs Not Appearing**: Wait 1-2 minutes for log propagation or check Logs Explorer
4. **Permission Errors**: Ensure `--allow-unauthenticated` flag for public functions or configure appropriate IAM

## Relevant Documentation & Resources
- [Cloud Functions Documentation](https://cloud.google.com/functions/docs)
- [Cloud Run Functions Overview](https://cloud.google.com/run/docs/triggering/cloud-functions)
- [Pub/Sub Triggers for Cloud Functions](https://cloud.google.com/functions/docs/calling/pubsub)
- [gcloud functions deploy Reference](https://cloud.google.com/sdk/gcloud/reference/functions/deploy)
- [Functions Framework for Node.js](https://github.com/GoogleCloudPlatform/functions-framework-nodejs)
- [Cloud Logging Documentation](https://cloud.google.com/logging/docs)

## Lab Completion Checklist
- [ ] Created function directory and code files
- [ ] Successfully deployed function with `gcloud functions deploy`
- [ ] Verified function is ACTIVE using `gcloud functions describe`
- [ ] Published test message to Pub/Sub topic
- [ ] Viewed function execution logs in Cloud Logging
- [ ] Confirmed function outputs appear correctly in logs

## Next Steps & Advanced Topics
- Implement error handling and retry logic
- Add unit tests for function code
- Configure monitoring alerts and dashboards
- Implement CI/CD pipeline for automated deployments
- Explore other trigger types (HTTP, Cloud Storage, Firestore)
- Implement function versioning and traffic splitting

---
