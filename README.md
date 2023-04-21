
## Create a CloudSql instance 
```bash
 gcloud sql instances create cloudrun-db-instance \
--database-version=MYSQL_8_0 \
--cpu=1 \
--memory=4GB \
--region=us-central1 \
--authorized-networks <public_p>/32 \
--root-password=sql@2012
```


## Create a database 
```bash
gcloud sql databases create compare_db --instance=cloudrun-db-instance
```

## Run the gcloud sql users create command to create the user.
```bash
gcloud sql users create cloud-user \
--instance=cloudrun-db-instance \
--password=sql@2012
```

## (Optional) Configure a Cloud Run service account
```bash
gcloud iam service-accounts list

gcloud projects add-iam-policy-binding ams369 \
  --member="serviceAccount:SERVICE_ACCOUNT_EMAIL" \
  --role="roles/cloudsql.client"
```

## Configure a Cloud SQL sample app
```bash
mvn clean package com.google.cloud.tools:jib-maven-plugin:2.8.0:build \
 -Dimage=gcr.io/ams369/cloudsqlrun:v1 -DskipTests
```


## CLoud run Deploy
```bash
gcloud run deploy run-sql --image gcr.io/ams369/cloudsqlrun:v1 \
  --region us-central1 \
  --allow-unauthenticated \
  --add-cloudsql-instances testingiamproject-369302:us-central1:cloudrun-db-instance \
  --set-env-vars INSTANCE_UNIX_SOCKET="/cloudsql/testingiamproject-369302:us-central1:cloudrun-db-instance" \
  --set-env-vars INSTANCE_CONNECTION_NAME="ams369:us-central1:cloudrun-db-instance" \
  --set-env-vars DB_NAME="compare_db" \
  --set-env-vars DB_USER="cloud-user" \
  --set-env-vars DB_PASS="sql@2012"
```

## Note
Make sure to create VPC connector in the same n/w where ur cloud sql instance is created 
