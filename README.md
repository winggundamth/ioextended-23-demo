# ioextended-23-demo

Google Cloud Workstations Demo

## Run the following commands on Cloud Workstations

### Preparation

* Create Cloud Workstations

```bash
gcloud auth login

gcloud config set project zcloud-cicd
export PROJECT_ID=$(gcloud config get project)
export REGION=asia-southeast1
gcloud config set deploy/region ${REGION}

gcloud artifacts repositories create containers --repository-format=docker \
  --location=${REGION} --description="Docker repository"

gcloud projects add-iam-policy-binding ${PROJECT_ID} \
  --member=serviceAccount:$(gcloud projects describe ${PROJECT_ID} \
  --format="value(projectNumber)")-compute@developer.gserviceaccount.com \
  --role="roles/clouddeploy.jobRunner"
gcloud iam service-accounts add-iam-policy-binding $(gcloud projects describe ${PROJECT_ID} \
  --format="value(projectNumber)")-compute@developer.gserviceaccount.com \
  --member=serviceAccount:$(gcloud projects describe ${PROJECT_ID} \
  --format="value(projectNumber)")-compute@developer.gserviceaccount.com \
  --role="roles/iam.serviceAccountUser" \
  --project=${PROJECT_ID}
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
  --member=serviceAccount:$(gcloud projects describe ${PROJECT_ID} \
  --format="value(projectNumber)")-compute@developer.gserviceaccount.com \
  --role="roles/run.developer"
```

### Run on demo

```bash
gcloud config set project zcloud-cicd
export PROJECT_ID=$(gcloud config get project)
export REGION=asia-southeast1
gcloud config set deploy/region ${REGION}

git clone https://github.com/googlecloudplatform/software-delivery-shield-demo-java.git
cd software-delivery-shield-demo-java/backend
sed -i "s/PROJECT_ID/${PROJECT_ID}/g" cloudrun.clouddeploy.yaml
sed -i "s/us-central1/${REGION}/g" cloudbuild.yaml cloudrun.clouddeploy.yaml

gcloud builds submit --config=cloudbuild.yaml --region=${REGION}
gcloud deploy apply --file cloudrun.clouddeploy.yaml
gcloud deploy releases create test-release-$(date +"%Y%m%d%H%M") \
  --delivery-pipeline=cloudrun-guestbook-backend-delivery \
  --skaffold-file=cloudrun.skaffold.yaml \
  --images=java-guestbook-backend=${REGION}-docker.pkg.dev/${PROJECT_ID}/containers/java-guestbook-backend:quickstart
```

* Change `software-delivery-shield-demo-java/backend/pom.xml` library `gson` from `2.8.8` to `2.8.9`
* Change to Java Image in `software-delivery-shield-demo-java/backend/Dockerfile` to `17.0.8_7-jre`
* Rebuild the whole thing again

## Clean everything

```bash
gcloud run services delete guestbook-backend-dev --region=${REGION} --project=${PROJECT_ID} --quiet
gcloud run services delete guestbook-backend-prod --region=${REGION} --project=${PROJECT_ID} --quiet
gcloud deploy delivery-pipelines delete cloudrun-guestbook-backend-delivery \
  --force --region=${REGION} --project=${PROJECT_ID} --quiet
gcloud artifacts repositories delete containers --location=${REGION} --quiet
```

* Delete Cloud Workstations
