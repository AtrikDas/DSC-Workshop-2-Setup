# DSC-Workshop-2-Setup

# Introduction
Have you ever wondered if memes are funnier in French rather than English? In this workshop we will be learning how to translate memes from different languages into English. We will use the Vision API, Translation API, and the Text-to-speech API of GCP to achieve our objectives:
1. Pass text recognized by the Cloud Vision API to the Cloud Translation API.
2. Create and use Cloud Translation glossaries to personalize Cloud Translation API translations.
3. Create an audio representation of translated text using the Text-to-Speech API.

# Setup
1. Install Anaconda from [here](https://www.anaconda.com/products/individual) which provides a virtual environment to run your python scripts and libraries. **Make sure to add anaconda to your path.**
2. After downloading it, create an environment called 'visionAI' by running these commands:

```bash
conda create --name visionAI
```
```bash
activate visionAI
```
3. Once you're in the env, install the python libraries necessary for GCP to work:
```bash
pip install --upgrade google-cloud-storage
```
4. Follow the installation instructions [here](https://cloud.google.com/sdk/docs/install) to install the Cloud SDK.
5. **Make sure to add the SDK bin file to your path (environment variables).** It should be of this file path pattern assuming you installed it at the default location:  
C:\Users\\{your_name}\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin
6. Create a free account on GCP [here](https://cloud.google.com/) and navigate to your console.
