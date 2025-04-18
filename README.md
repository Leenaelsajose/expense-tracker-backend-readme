# Expense Tracker Backend

- I don't have a pipeline created for this project so the code would need to be deploy using the below gloud scripts to GCP.
  - Enable Necessary APIs
  - Go to "APIs & Services" > "Library" in the Cloud Console.
  - Search for and enable the following APIs:
  - Cloud Functions API
  - Cloud Firestore API
  - Cloud Storage
  - BigQuery API
  - Firebase Management API
 
  - Create a new bucket to store user-uploaded receipts name it expense-tracker-receipts.
  - Go to [Firebase console](https://console.firebase.google.com/) and create project and then select Leena-Final-Project.
  - From under build select Firestore Database and create the default database and create a collection Expenses under it.
 
  - Create new BigQuery dataset named expense_tracker_data and create a table expenses_analyzed under it.
  - Assign below roles to the service account running the cloud run functions
    - artifactregistry.writer
    - bigquery.dataEditor
    - datastore.importExportAdmin
    - datastore.owner
    - datastore.user
    - editor
    - eventarc.eventReceiver
    - iam.serviceAccountUser
    - logging.logWriter
    - run.admin
   - Example `gcloud projects add-iam-policy-binding expense-tracker     --member="serviceAccount:273413129270-compute@developer.gserviceaccount.com"     --role="roles/datastore.user`
   

   - Visit the [Google Cloud SDK download page](https://cloud.google.com/sdk/docs/install) and download the Windows installer and install it.
   - Authenticate with Google Cloud `gcloud auth login`
   - Set Google Cloud project: leena-final-project `gcloud config set project leena-final-project`
   - Clone the project and set the directory to the expensetracker folder and build the code `dotnet build`
   - The code has 4 GCP Cloud Run functions,
     - expensetrackerbackend which adds the expenses to Firestore database
     - expensetrackertobigquery which gets triggered every time a document is added in the Firestore Expenses collection and adds it to the BigQuery table 
       expenses_analyzed in the dataset expense_tracker_data
     - get-expenses while pulls all the expenses for a user from Firestore.
     - upload-receipt which allows user to upload receipts corresponding to an expense to a storage bucket and updates Firestore with the signed url to that receipt 
      in the bucket.
   - Deploy each function with the below commands
     - Deploy expensetrackerbackend `gcloud functions deploy expensetrackerbackend --project leena-final-project --region us-central1 --runtime dotnet8 --source . --entry-point ExpenseTracker.ExpenseTrackerFunction --trigger-http --allow-unauthenticated![image](https://github.com/user-attachments/assets/c69cabd8-82ae-444c-8ec0-f20fbda6ee0d)`
     - Deploy expensetrackertobigquery `gcloud functions deploy ExpenseToBigQuery  --project leena-final-project --region us-central1 --runtime dotnet8 --source . --entry-point ExpenseTracker.ExpenseToBigQueryFunction  --trigger-http --allow-unauthenticated![image](https://github.com/user-attachments/assets/aa43598e-fc7f-4659-b0e1-5082cfdd771c)`
     - Add trigger to expensetrackertobigquery `gcloud eventarc triggers create expensetobigquery --location=us-central1 --service-account=273413129270-compute@developer.gserviceaccount.com --destination-run-service=expensetobigquery --destination-run-region=us-central1 --destination-run-path="/" --event-filters="database=(default)" --event-filters-path-pattern="document=Expenses/**" --event-filters="type=google.cloud.firestore.document.v1.written" --event-data-content-type=application/protobuf![image](https://github.com/user-attachments/assets/1c566491-fda1-49dc-a1b5-62d1c62818a2)`
     - Deploy get-expenses `gcloud functions deploy get-expenses --runtime dotnet8 --trigger-http --entry-point ExpenseTracker.GetExpensesFunction --region us-central1 --project leena-final-project![image](https://github.com/user-attachments/assets/591a1468-34c2-414b-9c5c-60d64f8c4b31)`
     - Deploy upload-receipt `gcloud functions deploy upload-receipt --project leena-final-project --region us-central1 --runtime dotnet8 --source . --entry-point ExpenseTracker.UploadReceiptFunction --trigger-http --allow-unauthenticated![image](https://github.com/user-attachments/assets/44386c73-7699-4522-b79c-a0bd102d3529)`
