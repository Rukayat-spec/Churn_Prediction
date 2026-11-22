# Telecom Customer Churn Prediction | End-to-End Machine Learning & MLOps

## Project Overview

This project demonstrates the development and deployment of an end-to-end machine learning pipeline designed to identify telecommunications customers who are at risk of discontinuing their services.

The solution combines predictive modelling, experiment tracking, API development, containerisation, and cloud deployment to demonstrate how a machine learning model can move beyond experimentation into a production-style environment.

An XGBoost classification model is used to estimate customer churn risk, while FastAPI and Gradio provide accessible interfaces for generating predictions. The application is containerised using Docker and deployed on AWS ECS Fargate, with GitHub Actions supporting automated delivery.

## Business Problem

Customer attrition presents a significant challenge for telecommunications providers, affecting recurring revenue, customer lifetime value, and acquisition costs.

Identifying customers who are likely to leave enables businesses to move from reactive retention efforts towards more targeted, proactive interventions.

This project addresses that challenge by developing a predictive solution that can:

- **Identify at-risk customers:** Estimate the likelihood of customer churn using historical customer information.
- **Support retention strategies:** Provide predictions that can help customer engagement teams prioritise retention activities.
- **Improve model accessibility:** Make predictions available through a REST API and an interactive web interface.
- **Enable reproducible experimentation:** Track model configurations, evaluation metrics, and generated artifacts using MLflow.
- **Support reliable deployment:** Use containerisation and automated workflows to maintain consistency between development and cloud environments.

## Technical Architecture

The project brings together several technologies across the machine learning lifecycle.

| Component | Technology | Function |
|---|---|---|
| Predictive modelling | XGBoost | Binary classification of customer churn |
| Experiment management | MLflow | Tracking experiments, metrics, and model artifacts |
| API development | FastAPI | Serving predictions through HTTP endpoints |
| Interactive application | Gradio | Browser-based interface for testing predictions |
| Containerisation | Docker | Packaging the application and dependencies |
| CI/CD | GitHub Actions | Automating image builds and delivery |
| Image registry | Docker Hub | Storing container images |
| Cloud infrastructure | AWS ECS Fargate | Running the containerised application |
| Traffic management | AWS Application Load Balancer | Routing incoming requests to the application |
| Monitoring | Amazon CloudWatch | Capturing application logs and deployment events |

## Implementation

### 1. Data Preparation and Model Development

The machine learning workflow begins with preparing customer data for predictive modelling.

This involves transforming input features into a suitable format, developing an XGBoost classification model, and evaluating its ability to distinguish between customers who churn and those who remain.

MLflow is integrated into the training process to maintain a record of experiments, model performance, and saved artifacts. This provides a consistent way to compare model runs and retrieve trained models.

### 2. Model Serving and User Interaction

To make the trained model accessible outside the development environment, the project uses FastAPI to expose prediction functionality through a REST API.

The application provides the following routes:

- `POST /predict` — Accepts customer information and returns a model prediction.
- `GET /` — Provides a health-check endpoint for monitoring application availability.
- `/ui` — Hosts a Gradio interface for submitting customer details and viewing predictions interactively.

This design allows predictions to be generated programmatically through API requests or manually through a browser.

### 3. Application Containerisation

Docker is used to package the inference application, its dependencies, and the required model artifacts into a deployable container.

The application runs through Uvicorn on port `8000`, providing a consistent execution environment across local development and AWS infrastructure.

Containerisation reduces environment-specific issues and simplifies application deployment.

### 4. Continuous Integration and Deployment

GitHub Actions manages the container build and delivery workflow.

When changes are pushed to the main branch, the workflow builds a new Docker image and publishes it to Docker Hub.

The updated image can then be deployed to AWS ECS Fargate by initiating a new service deployment, either manually or through an automated workflow step.

This approach supports repeatable releases and reduces the manual effort involved in delivering application updates.

### 5. AWS Infrastructure and Monitoring

The containerised application is hosted on AWS ECS using Fargate, eliminating the need to manage the underlying servers directly.

An Application Load Balancer receives HTTP requests on port `80` and forwards them to the application running on port `8000`.

Security groups restrict communication between the public load balancer and the ECS tasks.

Amazon CloudWatch captures container logs and ECS service events, supporting application monitoring and troubleshooting.

## Deployment Workflow

The deployment process follows these stages:

1. **Source control:** Application changes are committed and pushed to GitHub.
2. **Automated build:** GitHub Actions builds the Docker image.
3. **Image publishing:** The resulting image is uploaded to Docker Hub.
4. **Service deployment:** AWS ECS launches an updated container task.
5. **Health validation:** The Application Load Balancer checks the application's root endpoint to confirm availability.
6. **Prediction access:** Once the task passes its health checks, users can interact with the application through the API or Gradio interface.

## Engineering Challenges and Troubleshooting

### Load Balancer Health Check Failures

**Issue:** ECS tasks were running, but the Application Load Balancer reported unhealthy targets.

**Root cause:** Inconsistencies between the configured target port and the application's health-check endpoint.

**Resolution:** Implemented a root health-check route and aligned the load balancer listener, target group configuration, and application port.

### Python Module Resolution Errors

**Issue:** The application failed to start inside Docker because Python could not locate a required module.

**Root cause:** The container's Python import path did not correctly reference the application's source directory.

**Resolution:** Updated the Docker environment configuration to include the source directory and corrected the Uvicorn application reference.

### Application Endpoint Connectivity Issues

**Issue:** Requests to the load balancer's public DNS address timed out.

**Root cause:** Security group rules prevented traffic from reaching the ECS application.

**Resolution:** Configured the load balancer to accept HTTP traffic on port `80` and restricted ECS task access on port `8000` to traffic originating from the load balancer's security group.

### Updated Docker Image Not Reflected in Deployment

**Issue:** Changes to the application were not visible after a new container image was published.

**Root cause:** The ECS service continued running tasks based on its existing deployment.

**Resolution:** Initiated a new ECS deployment to refresh the running tasks and incorporated an optional deployment trigger into the CI/CD workflow.

### MLflow Model Retrieval Failures

**Issue:** The inference application could not locate the expected model artifact in the configured MLflow experiment.

**Root cause:** Differences in experiment naming and model artifact references between training and inference.

**Resolution:** Standardised the experiment configuration and model logging process, ensuring the application could consistently retrieve the required model.

### Differences Between Local and Cloud Model Paths

**Issue:** Model artifact locations differed between the local development environment and the deployed Docker container.

**Root cause:** MLflow generated environment-dependent artifact paths.

**Resolution:** Used environment-specific model loading configurations, allowing local development to reference MLflow artifacts while the deployed application accessed the model packaged within the container.

## Key Learning Outcomes

This project provides practical experience across the complete machine learning delivery lifecycle, including:

- Developing and evaluating a customer churn classification model.
- Managing reproducible machine learning experiments using MLflow.
- Building prediction services with FastAPI and Gradio.
- Packaging machine learning applications with Docker.
- Implementing CI/CD workflows using GitHub Actions.
- Deploying containerised applications with AWS ECS Fargate.
- Configuring application networking, health checks, and cloud monitoring.
- Troubleshooting integration issues across model development, application serving, and cloud infrastructure.

## Future Improvements

Potential extensions to the project include:

- Evaluating additional classification algorithms against the XGBoost baseline.
- Introducing model explainability using SHAP to identify important churn drivers.
- Incorporating precision, recall, F1-score, ROC-AUC, and PR-AUC into model evaluation.
- Optimising decision thresholds based on customer retention priorities.
- Monitoring prediction quality and data drift over time.
- Developing a customer risk segmentation framework to support targeted retention campaigns.
- Automating model retraining and deployment when performance degradation is detected.

