# Pipeline ETL OpenWeather 🌤️

A complete, production-ready ETL (Extract, Transform, Load) pipeline for weather data, built with Apache Airflow, Docker, Spark, and Google Cloud Platform (GCP). This project demonstrates modern data engineering practices including orchestration, containerization, and cloud data warehousing.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Key Components](#key-components)
- [Deployment](#deployment)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## 🎯 Overview

This project automates the collection, processing, and storage of weather forecasts for Casablanca using:

- **Extract**: Fetches weather data from OpenWeatherMap API
- **Transform**: Processes data with Apache Spark for analysis
- **Load**: Stores transformed data in Google BigQuery

The pipeline runs on a daily schedule and is fully orchestrated with Apache Airflow.

### Key Features

✅ **Automated Daily Pipeline** - Runs on a configurable schedule  
✅ **Docker Containerization** - Extract and transform tasks run in isolated containers  
✅ **Google Cloud Integration** - Uses GCS for data lake and BigQuery for data warehouse  
✅ **Infrastructure as Code** - Terraform configuration for reproducible cloud setup  
✅ **Error Handling** - Robust exception handling and validation  
✅ **Easy Local Development** - Docker Compose for one-command setup  

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     Apache Airflow                            │
│  (Orchestration & Scheduling)                                │
└──────────────────┬───────────────────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
┌───────▼──────────┐  ┌──────▼──────────┐
│   Extract Task   │  │ Transform Task  │
│  (Docker)        │  │ (Docker)        │
│                  │  │                 │
│ • OpenWeather    │  │ • Apache Spark  │
│   API            │  │ • Data Cleaning │
│ • Python         │  │ • Transformations
│                  │  │                 │
└───────┬──────────┘  └──────┬──────────┘
        │                    │
        ▼                    ▼
┌──────────────────┐  ┌──────────────────────┐
│   Google Cloud   │  │ Google BigQuery      │
│   Storage (GCS)  │  │ (Data Warehouse)     │
│   (Data Lake)    │  │                      │
└──────────────────┘  └──────────────────────┘
```

## 📦 Prerequisites

### Required
- **Docker** & **Docker Compose** (v20.10+)
- **Git**
- OpenWeatherMap API Key ([Get one here](https://openweathermap.org/api))

### Optional (for direct installation)
- Python 3.9+
- Terraform (for GCP deployment)
- Google Cloud Project with service account credentials

## 🚀 Installation

### Option 1: Docker Compose (Recommended)

1. **Clone and navigate to the project:**
```bash
git clone https://github.com/BriceGyb/Pipeline_ETL_OpenWeather.git
cd Pipeline_ETL_OpenWeather
```

2. **Start the services:**
```bash
docker-compose up -d
```

3. **Initialize Airflow:**
```bash
docker-compose exec airflow-webserver airflow db init
```

4. **Access Airflow UI:**
   - URL: `http://localhost:8080`
   - Default credentials: `admin` / `admin`

### Option 2: Local Installation

1. **Create and activate a virtual environment:**
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

2. **Install dependencies:**
```bash
pip install -r requirements.txt
pip install -r requirements-airflow.txt
```

3. **Initialize Airflow:**
```bash
airflow db init
airflow users create --username admin --password admin --firstname Admin --lastname User --role Admin --email admin@example.com
```

4. **Start services in separate terminals:**
```bash
# Terminal 1: Start Scheduler
airflow scheduler

# Terminal 2: Start Webserver
airflow webserver --port 8080
```

## ⚙️ Configuration

### Environment Variables

Set these variables in Airflow or in your system:

1. **Via Docker Compose:**
   Edit `docker-compose.yml` or use Airflow UI:

2. **Via Airflow UI:**
   - Go to Admin → Variables
   - Add the following variables:

| Variable | Description | Example |
|----------|-------------|---------|
| `openweather_api_key` | Your OpenWeatherMap API key | `your_api_key_here` |
| `google_cloud_project` | Your GCP project ID | `my-project-2025` |

### GCP Setup

If using Google Cloud (recommended):

1. **Create a GCP Project:**
```bash
gcloud projects create pipeline-weather-2025
gcloud config set project pipeline-weather-2025
```

2. **Enable Required APIs:**
```bash
gcloud services enable storage-api.googleapis.com
gcloud services enable bigquery.googleapis.com
gcloud services enable cloudresourcemanager.googleapis.com
```

3. **Create a Service Account:**
```bash
gcloud iam service-accounts create airflow-sa
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member serviceAccount:airflow-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com \
  --role roles/editor
```

4. **Deploy Infrastructure with Terraform:**
```bash
cd terraform
terraform init
terraform plan
terraform apply
```

5. **Create and mount credentials:**
   - Download service account key (JSON format)
   - Place in `./gcp/credentials.json`
   - Update path in `docker-compose.yml` if needed

## 📖 Usage

### Running the Pipeline Manually

1. **Trigger the DAG:**
   - Via Airflow UI: Click the "Trigger DAG" button on the pipeline
   - Via CLI:
     ```bash
     airflow dags trigger weather_elt_pipeline_docker_cloud
     ```

2. **Monitor Execution:**
   - Watch logs in Airflow UI
   - Check task status and logs in detail

### Checking Results

```bash
# View DAG details
airflow dags list

# Check task logs
airflow logs weather_elt_pipeline_docker_cloud extract_from_api_to_gcs

# View BigQuery results
bq query --use_legacy_sql=false '
  SELECT * FROM `project-id.weather_data.casablanca_forecasts` 
  LIMIT 10
'
```

### Schedule Adjustment

Edit the schedule in [dags/weather_pipeline_dag.py](dags/weather_pipeline_dag.py):

```python
schedule="@daily",  # Daily execution
# Or use cron:
schedule="0 2 * * *",  # 2 AM UTC daily
```

## 📁 Project Structure

```
Pipeline_ETL_OpenWeather/
├── dags/
│   └── weather_pipeline_dag.py    # Airflow DAG definition
├── src/
│   ├── extract.py                 # Extract data from API
│   └── transform.py               # Transform with Spark & load to BQ
├── terraform/
│   └── main.tf                    # GCP infrastructure setup
├── gcp/
│   └── credentials.json           # GCP service account (add this)
├── logs/                          # Airflow logs (auto-created)
├── docker-compose.yml             # Services orchestration
├── Dockerfile                     # Base Python image
├── Dockerfile.extract             # Extract container
├── Dockerfile.transform           # Transform container
├── requirements.txt               # Python dependencies
├── requirements-airflow.txt       # Airflow-specific dependencies
├── webserver_config.py            # Airflow configuration
└── README.md                      # This file
```

## 🔧 Key Components

### Extract Task (`src/extract.py`)

Fetches weather forecast data from OpenWeatherMap API:

- **Input**: OpenWeatherMap API endpoint
- **Processing**: 
  - Validates API key from environment
  - Fetches 5-day forecast for Casablanca
  - Generates timestamp for data versioning
- **Output**: JSON file stored in GCS data lake
- **Error Handling**: Graceful exception handling for API failures

**Environment Variables Used:**
- `OPENWEATHER_API_KEY`: API authentication key
- `GOOGLE_CLOUD_PROJECT`: GCP project for authentication

### Transform Task (`src/transform.py`)

Processes raw data with Spark and loads to BigQuery:

- **Input**: Latest JSON file from GCS
- **Processing**:
  - Initializes Spark session for big data processing
  - Flattens nested JSON structure
  - Extracts weather metrics (temperature, humidity, pressure, etc.)
  - Converts Unix timestamps to readable format
  - Data quality validation
- **Output**: Cleaned data in BigQuery table
- **Error Handling**: Validates schema and data integrity

**Environment Variables Used:**
- `GOOGLE_CLOUD_PROJECT`: GCP project ID
- GCP credentials via service account

### Airflow DAG (`dags/weather_pipeline_dag.py`)

Orchestrates the pipeline:

```
extract_from_api_to_gcs >> transform_in_spark_and_load_to_bq
```

- **Frequency**: Daily (configurable)
- **Executor**: LocalExecutor (can be changed to distributed)
- **Docker Integration**: Both tasks run in isolated containers
- **Error Handling**: Automatic retries and alerting

### Infrastructure (`terraform/main.tf`)

Provisions GCP resources:

- **Storage Bucket** (Data Lake): Stores raw weather data
- **BigQuery Dataset**: Data warehouse for analysis
- **Temp Bucket**: Temporary storage for Spark operations
- **Auto-cleanup**: Automatic deletion of temp files after 1 day

## 🌐 Deployment

### Deploy to Production

1. **Use Terraform for Infrastructure:**
```bash
cd terraform
terraform apply
```

2. **Push to Docker Registry (optional):**
```bash
docker build -f Dockerfile.extract -t your-registry/weather-extract:latest .
docker push your-registry/weather-extract:latest
```

3. **Deploy Airflow on Kubernetes/VM:**
   - Update `docker-compose.yml` with production settings
   - Configure proper database (PostgreSQL recommended)
   - Set up monitoring and alerts

### Production Checklist

- [ ] Configure production-grade PostgreSQL instead of default
- [ ] Set up proper secret management (use Google Secret Manager)
- [ ] Configure monitoring and logging
- [ ] Set up alerting for failed DAG runs
- [ ] Configure automated backups
- [ ] Document SLAs and runbooks
- [ ] Implement data quality checks

## 🐛 Troubleshooting

### Docker Compose Issues

**Services not starting:**
```bash
# Check container status
docker-compose ps

# View logs
docker-compose logs -f airflow-webserver

# Restart services
docker-compose restart
```

**Permission issues:**
```bash
# Fix ownership
sudo chown -R $USER:$USER ./logs ./plugins
```

### Airflow Issues

**Can't access webserver:**
- Check if port 8080 is available
- Verify containers are running: `docker-compose ps`
- Check logs: `docker-compose logs airflow-webserver`

**DAG not appearing:**
```bash
# Refresh DAGs
docker-compose exec airflow-webserver airflow dags list

# Check DAG syntax
docker-compose exec airflow-webserver airflow dags validate
```

**Task failures:**
1. Check Airflow UI for detailed logs
2. Verify environment variables are set
3. Confirm GCP credentials are valid
4. Check API key is active and has quota

### GCP Issues

**BigQuery insert failures:**
```bash
# Verify dataset exists
bq ls --dataset_id=weather_data

# Check table schema
bq show weather_data.casablanca_forecasts
```

**GCS access denied:**
```bash
# Verify service account permissions
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:serviceAccount:*"
```

## 📊 Monitoring

### Check Pipeline Health

1. **Airflow UI Dashboard:**
   - Monitor DAG runs and task status
   - Set up email alerts for failures

2. **View Data in BigQuery:**
```bash
bq query --use_legacy_sql=false '
  SELECT 
    date(dt) as forecast_date,
    COUNT(*) as record_count,
    ROUND(AVG(main.temp), 2) as avg_temp
  FROM `project-id.weather_data.casablanca_forecasts`
  GROUP BY forecast_date
  ORDER BY forecast_date DESC
'
```

3. **GCS Monitoring:**
```bash
gsutil ls -r gs://tr-weather-pipeline-datalake-2025/
gsutil du -s gs://tr-weather-pipeline-datalake-2025/
```

## 🤝 Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 📞 Support

For issues and questions:
- Check the [Troubleshooting](#troubleshooting) section
- Review [Airflow Documentation](https://airflow.apache.org/)
- Check [Google Cloud Documentation](https://cloud.google.com/docs)
- Open an issue on GitHub

## 🔗 Useful Links

- [OpenWeatherMap API](https://openweathermap.org/api)
- [Apache Airflow Documentation](https://airflow.apache.org/)
- [Google Cloud Storage](https://cloud.google.com/storage)
- [BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- [Apache Spark Documentation](https://spark.apache.org/docs/)
- [Terraform Google Cloud Provider](https://registry.terraform.io/providers/hashicorp/google/latest)

---

**Last Updated:** January 2025  
**Version:** 1.0.0  
**Maintainer:** [Your Name/Organization]
