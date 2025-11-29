# 🚀 Quick Start Guide - Uber Data Pipeline

Get your data pipeline running in 15 minutes!

## 📋 What You'll Need

- Google Cloud Platform account (with billing enabled)
- Python 3.9+
- Basic knowledge of SQL and Python

## ⚡ 3-Step Setup

### Step 1: GCP Setup (5 minutes)

```bash
# Set your project ID
export PROJECT_ID="your-uber-pipeline-project"

# Enable required APIs
gcloud services enable compute.googleapis.com
gcloud services enable bigquery.googleapis.com
gcloud services enable storage.googleapis.com

# Create storage bucket
gsutil mb gs://uber_data_analysis_${PROJECT_ID}

# Upload data
gsutil cp uber_data.csv gs://uber_data_analysis_${PROJECT_ID}/

# Create BigQuery dataset
bq mk --dataset --location=US ${PROJECT_ID}:uber_data_analytics_project
```

### Step 2: Deploy Mage AI (5 minutes)

```bash
# Create VM for Mage
gcloud compute instances create mage-pipeline \
    --machine-type=e2-standard-2 \
    --zone=us-central1-a \
    --image-family=ubuntu-2004-lts \
    --image-project=ubuntu-os-cloud \
    --boot-disk-size=50GB

# SSH into VM
gcloud compute ssh mage-pipeline --zone=us-central1-a

# Install Mage
sudo apt-get update && sudo apt-get install -y python3-pip
pip3 install mage-ai pandas google-cloud-bigquery

# Start Mage
mage start uber_pipeline
```

Access Mage at: `http://[VM_EXTERNAL_IP]:6789`

### Step 3: Run Pipeline (5 minutes)

1. **In Mage UI**:
   - Create new pipeline: "uber_etl"
   - Add data loader block → Paste `extract_data.py`
   - Add transformer block → Paste `trasform_data.py`
   - Add data exporter block → Paste `load_data.py`

2. **Configure** `io_config.yaml`:
```yaml
bigquery:
  project: your-project-id
  dataset: uber_data_analytics_project
  keyfile: /path/to/service-account.json
```

3. **Run Pipeline**: Click "Execute pipeline" button

## ✅ Verify Setup

```sql
-- Check tables in BigQuery
bq ls uber_data_analytics_project

-- Query sample data
bq query --use_legacy_sql=false '
  SELECT COUNT(*) as total_records 
  FROM `uber_data_analytics_project.fact_table`
'
```

Expected output: `100000 records`

## 📊 View Results in Looker Studio

1. Go to [Looker Studio](https://lookerstudio.google.com)
2. Create new report
3. Add BigQuery data source: `uber_data_analytics_project.table_analytics`
4. Create visualizations!

## 🐛 Quick Troubleshooting

**Can't access Mage UI?**
```bash
# Open firewall
gcloud compute firewall-rules create allow-mage \
    --allow=tcp:6789 --source-ranges=0.0.0.0/0
```

**BigQuery permission error?**
```bash
# Grant permissions to service account
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:YOUR-SA@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/bigquery.dataEditor"
```

## 🎯 Next Steps

- Read [README_COMPREHENSIVE.md](./README_COMPREHENSIVE.md) for details
- Explore sample queries in `sql_commands`
- Check Jupyter notebook: `Uber_DataPipeline.ipynb`

## 💡 Pro Tips

1. **Cost Control**: Stop Compute Engine when not in use
2. **Monitoring**: Enable Cloud Logging for pipeline runs
3. **Automation**: Schedule pipeline in Mage for daily runs

---

**Total Setup Time**: ~15 minutes  
**Cost**: ~$0.10/hour for Compute Engine + BigQuery query costs

Happy data engineering! 🎉
