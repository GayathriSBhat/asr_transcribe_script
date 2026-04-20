

# How to use?

1. Use load_dataset.ipynb to pull audio data from Hugging Face.
2. Provide data:
   Name sub-folder as the required language inside audio_files and update for required languages in transcribe.ipynb
   Update MCF_Languages Excel sheet inside the transcription_data folder and add the Original_transcription in the sheet for the required language.
3. Run transcription pipeline (transcribe.inpynb))
   pipeline: transcribe -> Translate -> Scoring -> save to excel sheet

> Note:
>
> HF token guildelines:
>
> Use Read Token (fine grained not required)
>
> When you're downlaoding transcripts directly from HF use link in below format:
>
> https://huggingface.co/datasets/sarulab-speech/commonvoice22_sidon/resolve/7c06e40565468fda8c80a57c0ce4a7d9af97c095/ab/validation-00000.tar.gz
