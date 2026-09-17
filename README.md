# AI-Enabled Text Summarization System

An end-to-end machine learning pipeline for automated abstractive text summarization using transformer models, deployed on AWS with automated CI/CD.

## Overview

This project implements a production-grade NLP system that automatically generates concise, coherent summaries from long-form text. The system fine-tunes the BART transformer model on 25,000 articles to learn abstractive summarization, achieves a ROUGE-1 validation score of 0.41, and serves predictions via a Flask API with ~3-4 second CPU inference latency.

**Key Features:**

- Fine-tuned BART transformer model for abstractive text summarization
- Robust data processing pipeline (extraction, cleaning, validation)
- REST API for real-time inference
- Containerized deployment with Docker
- Automated CI/CD pipeline using GitHub Actions
- Cloud-hosted on AWS EC2 with ECR image registry
- Optimized for AWS Free Tier constraints (1.5 GB Docker image)

## Architecture

```
Raw Dataset (25,000 articles)
    ↓
Data Ingestion & Cleansing (12% anomalies filtered)
    ↓
Model Training (BART transformer, fine-tuned)
    ↓
Model Evaluation (ROUGE-1: 0.41)
    ↓
Flask API Service (3-4 sec inference)
    ↓
Docker Containerization (1.5 GB optimized)
    ↓
GitHub Actions CI/CD (< 5 min deployment)
    ↓
AWS EC2 + ECR Deployment
```

## Tech Stack

**Backend:** Python, Flask, Node.js  
**ML/NLP:** Transformers (BART), Pandas, NumPy  
**Data Storage:** PostgreSQL, MinIO (S3)  
**DevOps:** Docker, AWS (EC2, ECR), GitHub Actions  
**Metrics:** ROUGE Score, Validation Framework

## Project Structure

```
.
├── config/
│   ├── config.yaml              # Configuration parameters
│   └── params.yaml              # Model and training parameters
├── src/
│   ├── config/                  # Configuration managers
│   ├── components/              # Pipeline components
│   │   ├── data_ingestion.py
│   │   ├── data_transformation.py
│   │   ├── model_trainer.py
│   │   └── model_evaluation.py
│   └── pipeline/                # ML pipeline orchestration
│       └── training_pipeline.py
├── notebooks/                   # Exploratory analysis
├── app.py                       # Flask web application
├── main.py                      # Pipeline entry point
├── Dockerfile                   # Container specification
├── requirements.txt             # Python dependencies
└── README.md                    # This file
```

## Getting Started

### Prerequisites

- Python 3.8+
- Git
- Conda (optional but recommended)
- AWS Account (for deployment)
- Docker (for containerization)

### Installation

**Step 1: Clone the Repository**

```bash
git clone https://github.com/based-afk/Text-Summarization
cd Text-Summarization
```

**Step 2: Create a Virtual Environment**

Using conda (recommended):

```bash
conda create -n text-summarizer python=3.8 -y
conda activate text-summarizer
```

Or using venv:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

**Step 3: Install Dependencies**

```bash
pip install -r requirements.txt
```

### Running Locally

**Option 1: Start the Flask Web App**

```bash
python app.py
```

The application will be available at `http://localhost:5000`

Open your browser and paste an article to get a summary.

**Option 2: Run the Training Pipeline**

```bash
python main.py
```

This executes the complete pipeline:

1. Data ingestion and validation
2. Text preprocessing and tokenization
3. Model fine-tuning
4. Evaluation and metrics reporting

**Option 3: Docker Deployment (Local)**

Build the Docker image:

```bash
docker build -t text-summarizer:latest .
```

Run the container:

```bash
docker run -p 5000:5000 text-summarizer:latest
```

Access the app at `http://localhost:5000`

## Usage

### Web Interface

1. Navigate to `http://localhost:5000`
2. Paste or type an article in the text area
3. Click "Generate Summary"
4. View the generated summary and key metrics

### API Endpoint

```bash
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{"article_text": "Your long article text here..."}'
```

**Response:**

```json
{
  "original_text": "Your long article text here...",
  "summary": "Concise generated summary...",
  "inference_time": 3.2
}
```

## Configuration

Edit `config/config.yaml` to customize:

- Dataset paths
- Model names and versions
- Training parameters (batch size, learning rate, epochs)
- Validation thresholds

Edit `config/params.yaml` for:

- Model hyperparameters
- Preprocessing settings
- Evaluation metrics

## Model Details

**Model:** BART (facebook/bart-base)  
**Training Data:** 25,000 articles (cleaned and validated)  
**Validation Metric:** ROUGE-1 Score = 0.41  
**Inference Latency:** 3-4 seconds (CPU-based)  
**Inference Hardware:** AWS t2.micro (Free Tier eligible)

## Evaluation Metrics

- **ROUGE-1:** Overlap of individual words between generated and reference summaries
- **ROUGE-L:** Longest common subsequence (captures sentence structure)
- **Data Quality:** 12% of anomalous records filtered during preprocessing

See `/reports` for detailed evaluation reports.

## Deployment

### AWS Deployment with GitHub Actions

#### Prerequisites

- AWS Account with appropriate permissions
- GitHub repository with this code

#### Step 1: Create IAM User for Deployment

1. Login to AWS Console
2. Navigate to IAM → Users → Create User
3. Attach policies:
   - `AmazonEC2FullAccess` (for EC2 instance management)
   - `AmazonEC2ContainerRegistryFullAccess` (for ECR access)
4. Create access keys and save them securely

#### Step 2: Create ECR Repository

```bash
# In AWS Console or via CLI:
aws ecr create-repository --repository-name text-summarizer --region us-east-1
```

Note the URI (example: `566373416292.dkr.ecr.us-east-1.amazonaws.com/text-summarizer`)

#### Step 3: Launch EC2 Instance

1. Go to EC2 Dashboard
2. Click "Launch Instance"
3. Select Ubuntu 20.04 LTS
4. Instance type: `t2.micro` (Free Tier eligible)
5. Configure security group to allow:
   - Port 22 (SSH)
   - Port 80 (HTTP)
   - Port 443 (HTTPS)
6. Launch and download key pair (.pem file)

#### Step 4: Configure EC2

SSH into your instance:

```bash
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

Install Docker:

```bash
sudo apt-get update -y
sudo apt-get upgrade -y

curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

sudo usermod -aG docker ubuntu
newgrp docker
```

#### Step 5: Configure GitHub Secrets

Go to your GitHub repository → Settings → Secrets and variables → Actions

Add the following secrets:

```
AWS_ACCESS_KEY_ID          = <your-access-key>
AWS_SECRET_ACCESS_KEY      = <your-secret-key>
AWS_REGION                 = us-east-1
AWS_ECR_LOGIN_URI          = 566373416292.dkr.ecr.us-east-1.amazonaws.com
ECR_REPOSITORY_NAME        = text-summarizer
EC2_HOST                   = <your-ec2-public-ip>
EC2_USERNAME               = ubuntu
EC2_SSH_PRIVATE_KEY        = <contents-of-your-pem-file>
```

#### Step 6: Set Up GitHub Actions Runner

1. Go to Settings → Actions → Runners → New self-hosted runner
2. Choose **Linux** OS
3. Download and follow the setup commands provided
4. Register the runner with your repository

#### Step 7: Deploy

Push to main branch to trigger automatic deployment:

```bash
git add .
git commit -m "Deploy update"
git push origin main
```

GitHub Actions will:

1. Build Docker image
2. Push to ECR
3. Deploy to EC2
4. Start the service

Your app will be live at `http://<EC2_PUBLIC_IP>` within 5 minutes.

## Performance Metrics

- **Training Time:** ~6-8 hours (single GPU)
- **Data Processing:** 500MB+ CSV handled in constant memory (45MB heap)
- **Model Inference:** 3-4 seconds per article (CPU)
- **Deployment Time:** < 5 minutes via CI/CD pipeline
- **Container Size:** 1.5 GB (optimized for Free Tier)

## Development Workflow

The project follows a modular pipeline approach:

1. **Update `config.yaml`** → Set paths and global configuration
2. **Update `params.yaml`** → Modify model hyperparameters
3. **Update `entity`** → Define data schemas
4. **Update `config/` manager** → Load and validate configs
5. **Update `components/`** → Implement pipeline steps
6. **Update `pipeline/`** → Orchestrate workflow
7. **Update `main.py`** → Set entry point logic
8. **Update `app.py`** → Expose via API/web interface

## Troubleshooting

**Issue:** Port 5000 already in use

```bash
# Find and kill process using port 5000
lsof -i :5000
kill -9 <PID>
```

**Issue:** Docker build fails due to size

```bash
# Build with specific optimizations
docker build --no-cache -t text-summarizer:latest .
```

**Issue:** AWS credentials not working

```bash
# Verify credentials are set correctly
aws sts get-caller-identity
```

**Issue:** GPU memory errors during training

```bash
# Reduce batch size in params.yaml
batch_size: 8  # Instead of 16
```

## Future Enhancements

- [ ] Multi-language summarization support
- [ ] Fine-grained abstractive vs. extractive summary options
- [ ] Batch processing API for bulk summarization
- [ ] Real-time performance monitoring dashboard
- [ ] Model versioning and A/B testing framework
- [ ] Advanced caching for frequently summarized content

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add feature'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Contact

**Author:** Subhasis Bhuyan  
**Email:** subhasisbhuyan2004@gmail.com  
**GitHub:** [github.com/based-afk](https://github.com/based-afk)  
**LinkedIn:** [linkedin.com/in/subhasis-bhuyan](https://linkedin.com/in/subhasis-bhuyan)

## Acknowledgments

- BART model from [Hugging Face Transformers](https://huggingface.co/)
- Dataset inspiration from [CNN/DailyMail](https://github.com/abisee/cnn-dailymail)
- AWS Free Tier for cost-effective cloud deployment

---

**Last Updated:** September 2026  
**Status:** Production Ready
