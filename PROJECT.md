# PROJECT.md

# Skin Lesion Identification App

## Overview
This project is a mobile application for identifying and classifying skin lesions from user-submitted images. Users will be able to create an account, capture or upload a photo of a skin lesion, receive a machine learning prediction, and store their scan history.

This project is also a structured learning project. The goal is not only to build a working app, but to fully understand the mobile, backend, machine learning, and reinforcement learning systems involved.

## What Is a Skin Lesion
A skin lesion is an abnormal area of skin tissue. This can include:
- spots
- moles
- bumps
- patches
- sores
- growths
- scaly areas
- discolored areas

A lesion is not automatically dangerous or cancerous. Some lesions are harmless, some are inflammatory, and some may be suspicious or malignant.

## Main Goal
Build a mobile app that:
- allows users to sign up and log in
- accepts skin lesion images from the camera or photo library
- sends images to a backend for prediction
- returns a lesion classification and confidence score
- stores scan history for each user
- creates a foundation for future reinforcement learning features

## Clinical Scope
This project focuses on identifying a broad range of skin lesion categories from images.

Active categories (HAM10000, Phases 1–2):
- melanoma
- nevus
- basal cell carcinoma
- actinic keratosis
- benign keratosis
- dermatofibroma
- vascular lesion

Deferred to Phase 3 (require additional datasets beyond HAM10000):
- squamous cell carcinoma
- seborrheic keratosis

See `ml/data/README.md` for the canonical label set and dataset translation tables.

## Product Vision
The app should feel like a real mobile tool for lesion screening and organization. A user should be able to:
1. create an account
2. capture or upload a lesion image
3. receive a prediction from the model
4. view prediction confidence
5. save and review previous scans

The app is not intended to replace medical professionals or provide medical advice. It is a technical and educational project focused on classification and triage-style support.

## Tech Stack

### Mobile App
- React Native
- Expo
- TypeScript

### Backend
- Python
- FastAPI

### Machine Learning
- PyTorch

### Database / Auth / Storage
- Supabase

## Why This Stack
This stack keeps the project modern while still being manageable.

- React Native + Expo allows mobile-first development and easy phone testing through QR code scanning
- TypeScript improves code quality and maintainability
- FastAPI works well with Python-based machine learning services
- PyTorch is used for image classification and later reinforcement learning experiments
- Supabase provides authentication, database support, and image storage in one system

## Learning Goals
This project is meant to teach:
- React Native and Expo fundamentals
- TypeScript for mobile app development
- FastAPI backend development
- API design and file upload handling
- Supabase authentication, storage, and database usage
- PyTorch model training and inference
- reinforcement learning in a realistic product setting

## Initial Project Scope
Version 1 will focus on the core pipeline:
1. user signs up or logs in
2. user uploads or captures a lesion image
3. backend receives the image
4. model returns a prediction
5. result is saved to the database
6. user can review previous scans

## Modeling Direction

### Phase 1: Supervised Learning Baseline
The first model will use supervised learning for image classification.

Possible starting tasks:
- binary classification: suspicious vs non-suspicious lesion
- multiclass classification: lesion category prediction

The likely best path is to begin with a simpler baseline and then expand to broader multiclass classification once the training and inference pipeline is stable.

### Phase 2: Broader Lesion Classification
After the baseline works, the model can expand to classify a broader range of lesion types supported by the dataset.

### Phase 3: Confidence and Decision Support
Later versions may include:
- confidence thresholds
- uncertainty warnings
- better result interpretation
- image quality checks

## Reinforcement Learning Plan
Reinforcement learning will not be used for the initial image classifier itself.

Instead, RL may later be used for decision-making tasks such as:
- deciding whether another image is needed
- deciding whether confidence is high enough to finalize a result
- selecting the next best follow-up question
- improving the scan flow over time

The first version of the project will prioritize a strong supervised learning classifier before adding RL.

## Dataset Direction
The project will likely rely on large public skin lesion datasets.

Important dataset qualities:
- large number of labeled images
- reliable lesion labels
- class balance or manageable imbalance
- metadata if available
- image types that are relevant to mobile use cases

The exact dataset will be chosen later, but the project aims to support a broad lesion classification task rather than only a narrow binary screen.

## Core Features
- user authentication
- image upload or camera capture
- skin lesion prediction
- confidence score display
- scan history
- secure per-user data access
- backend API for inference
- storage of uploaded images and prediction results

## Backend Responsibilities
The backend will:
- receive uploaded images
- validate request data
- preprocess images for inference
- run model prediction
- return prediction results
- communicate with Supabase when saving scan data
- expose clean API routes for the mobile app

## Database Design

### profiles
- id
- email
- created_at

### scans
- id
- user_id
- image_url
- prediction
- confidence
- created_at

Optional future fields:
- lesion_type
- model_version
- notes
- follow_up_required

## Development Order

### Phase 1: Planning and Structure
- define project scope
- finalize tech stack
- create repo structure
- document system architecture

### Phase 2: Backend Foundation
- create FastAPI app
- connect Supabase
- set up authentication support
- create image upload pipeline
- build placeholder prediction endpoint

### Phase 3: Machine Learning
- choose dataset
- clean and inspect labels
- build PyTorch training pipeline
- train baseline model
- save and load model weights
- connect inference to backend

### Phase 4: Mobile App
- create Expo app
- add authentication screens
- add camera / upload flow
- connect to backend
- build result screen
- build history screen

### Phase 5: Improvement
- improve UI and user flow
- add better prediction handling
- add confidence warnings
- add broader lesion support
- add RL-based decision support

## MVP Definition
A successful MVP is:
- a mobile app where users can sign in
- upload or take a lesion photo
- receive a prediction from a backend model
- save and view previous scan results

## Future Ideas
- multiclass lesion classification improvements
- lesion change tracking over time
- follow-up scan reminders
- result explanations
- Grad-CAM or heatmap visualizations
- RL-based follow-up decision policy
- clinician-facing dashboard
- image quality assessment before prediction

## Constraints
- this is an educational and technical project
- this is not a medical device
- predictions should be treated as experimental
- the app should avoid presenting results as medical certainty

## Guiding Principles
- keep the system understandable
- build in phases
- avoid over-engineering early
- prioritize learning and clarity
- make each part of the stack serve a real purpose
