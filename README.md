# DSC-Workshop-2-Setup

# Introduction
Have you ever wondered if memes are funnier in French rather than English? In this workshop we will be learning how to translate memes from different languages into English. We will use the Vision API, Translation API, and the Text-to-speech API of GCP to achieve our objectives:
1. Pass text recognized by the Cloud Vision API to the Cloud Translation API.
2. Create and use Cloud Translation glossaries to personalize Cloud Translation API translations.
3. Create an audio representation of translated text using the Text-to-Speech API.

# Setup - DO THIS BEFORE WORKSHOP!!
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

# Code Snippets 

We will be using these code snippets during the workshop. You don't have to worry what each line means right now! Everything will be explained during the workshop. 

#### Code snippet #1

```python
import html
import io
import os


from google.api_core.exceptions import AlreadyExists
from google.cloud import texttospeech
from google.cloud import translate_v3beta1 as translate
from google.cloud import vision
```

#### Code snippet #2

```python

os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = r'{insert your own json key}'
PROJECT_ID = os.environ["GOOGLE_CLOUD_PROJECT"]
```

#### Code snippet #3

```python

def pic_to_text(infile):
    
   
    client = vision.ImageAnnotatorClient()

    
    with io.open(infile, "rb") as image_file:
        content = image_file.read()

    image = vision.Image(content=content)

    
    response = client.document_text_detection(image=image)
    text = response.full_text_annotation.text
    print("Detected text: {}".format(text))

    return text
```

#### Code snippet #4

```python

def create_glossary(languages, project_id, glossary_name, glossary_uri):
    
    
    client = translate.TranslationServiceClient()

    
    location = "us-central1"

    
    name = client.glossary_path(project_id, location, glossary_name)

    
    language_codes_set = translate.types.Glossary.LanguageCodesSet(
        language_codes=languages
    )

    gcs_source = translate.types.GcsSource(input_uri=glossary_uri)

    input_config = translate.types.GlossaryInputConfig(gcs_source=gcs_source)

    
    glossary = translate.types.Glossary(
        name=name, language_codes_set=language_codes_set, input_config=input_config
    )

    parent = f"projects/{project_id}/locations/{location}"

    
    try:
        operation = client.create_glossary(parent=parent, glossary=glossary)
        operation.result(timeout=90)
        print("Created glossary " + glossary_name + ".")
    except AlreadyExists:
        print(
            "The glossary "
            + glossary_name
            + " already exists. No new glossary was created."
        )    
```

#### Code snippet #5

```python

def translate_text(
    text, source_language_code, target_language_code, project_id, glossary_name
):
    

    
    client = translate.TranslationServiceClient()

    
    location = "us-central1"

    glossary = client.glossary_path(project_id, location, glossary_name)

    glossary_config = translate.types.TranslateTextGlossaryConfig(glossary=glossary)

    parent = f"projects/{project_id}/locations/{location}"

    result = client.translate_text(
        request={
            "parent": parent,
            "contents": [text],
            "mime_type": "text/plain",  # mime types: text/plain, text/html
            "source_language_code": source_language_code,
            "target_language_code": target_language_code,
            "glossary_config": glossary_config,
        }
    )

   
    return result.glossary_translations[0].translated_text   
```

#### Code snippet #6

```python

def text_to_speech(text, outfile):
    

    
    escaped_lines = html.escape(text)

    
    ssml = "<speak>{}</speak>".format(
        escaped_lines.replace("\n", '\n<break time="2s"/>')
    )

    
    client = texttospeech.TextToSpeechClient()

    
    synthesis_input = texttospeech.SynthesisInput(ssml=ssml)

    
    voice = texttospeech.VoiceSelectionParams(
        language_code="en-US", ssml_gender=texttospeech.SsmlVoiceGender.MALE
    )

    
    audio_config = texttospeech.AudioConfig(
        audio_encoding=texttospeech.AudioEncoding.MP3
    )

   

    request = texttospeech.SynthesizeSpeechRequest(
        input=synthesis_input, voice=voice, audio_config=audio_config
    )

    response = client.synthesize_speech(request=request)

    
    with open(outfile, "wb") as out:
        out.write(response.audio_content)
        print("Audio content written to file " + outfile) 
```

#### Code snippet #7

```python
def main():

    
    infile = "resources/example.png"
    
    outfile = "resources/example.mp3"

    
    glossary_langs = ["fr", "en"]
    
    glossary_name = "bistro-glossary"
    
    glossary_uri = "gs://cloud-samples-data/translation/bistro_glossary.csv"

    create_glossary(glossary_langs, PROJECT_ID, glossary_name, glossary_uri)

    
    text_to_translate = pic_to_text(infile)
    
    text_to_speak = translate_text(
        text_to_translate, "fr", "en", PROJECT_ID, glossary_name
    )
    
    text_to_speech(text_to_speak, outfile)

if __name__ == "__main__":
    main()
```
