1. Install Homebrew (if not already installed): `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
2. Install Python: `brew install python@3.10`
   1. As of this date (2023-09-07) Whisper works with Python up to 3.10
3. If `running python --version` triggers a `zsh` error, create an alias into `.zshrc` file for `python` and `pip`
   1. `nano ~/.zshrc`

   ```
   alias python="your/python3/location/bin/python3"
   alias pip="your/python3/location/bin/pip3"
   ```
   2. If you don't know your python installation location, use `which python` in terminal
   3. Save the chages with CTRL + O then Enter then CTRL + X
4. Install PyTorch, a machine learning library
   1. Go to https://pytorch.org/get-started/locally/
   2. Scroll down to Start Locally section
   3. Proceed with making selections such as:
      1. PyTorch Build: Stable (2.0.0)
      2. Your OS
      3. Package: Pip
      4. Language: Python
      5. Compute Platform: CUDA (for computers with NVIDIA cards), CPU (for non-NVIDIA computers) also called Default
      6. Copy the generated command below these options
      7. Paste it in terminal and run it to install this library
    4. `brew install ffmpeg` for Mac
       1. Install Chocolatey for Windows and install `ffmpeg` using that
    5. `pip install -U openai-whisper`
5. Install the rest of the remaining packages that are needed:
   ```
   pip install openai
   pip install python-docx
   ```
6. create aliases to run python scripts
   1. `nano ~/.zshrc`
   ```
   alias mmft="python /Users/username/.../openai-meeting-minutes-generator/text_file_to_meeting_minutes_generator.py"

   alias mmrg="python /Users/username/.../openai-meeting-minutes-generator/remote-whisper-to-gpt-4_meeting_minutes_generator.py"

   alias mml2rg="python /Users/username/.../openai-meeting-minutes-generator/local-whisper-to-gpt-4_meeting_minutes_generator.py"
   ```
   2. Save the chages with `CTRL + O` then `Enter` then `CTRL + X`
   3. Execute the scripts by providing the expected arguments:
   ```
   mmft <path-to-text-file> <path-to-new-docx-file> [model type]
   mml2rg <path-to-audo-file> <path-to-new-docx-file> [model type]
   mmrg <path-to-audo-file> <path-to-new-docx-file>
   ```
   ### Issues Encountered with mml2rg (local whisper with remote gpt-4). See file: local-whisper-to-gpt-4_meeting_minutes_generator.py

      ### With Higher Level Access To The Model

      A warning will be shown that the model switches to fp32 due to machine limitations

      ### With Lower Level Access To The Model

      ```jsx
      RuntimeError: "slow_conv2d_cpu" not implemented for 'Half'

      // solution. use the fp16=False
      // The whisper.DecodingOptions(fp16=False) explicitly sets the decoding to use single precision (float32) instead of half precision (float16)
      // Given the experienced error this makes sense. Also, when running "whisper filename.wav" in terminal, this calling directly the local service
      // a warning is displayed: "warnings.warn("FP16 is not supported on CPU; using FP32 instead")" but in this case whisper is able to fallback to fp32
      // automatically. In case of python script, this has to be mentioned specifically
      # decode the audio
      options = whisper.DecodingOptions(fp16=False)
      ```

   ### Issues Encountered with mmrg (local whisper with remote gpt-4). See file: remote-whisper-to-gpt-4_meeting_minutes_generator.py

   Once the code is run, there is a limit for the audio files provided. An error is provided in return:

   ```jsx
   Maximum content size limit (26214400) exceeded (26439132 bytes read)
   ```

   This means that the OpenAI's Whisper model (and other OpenAI APIs) have a content size limit, which at the time of this error message seems to be 26,214,400 bytes (or around 26.2 MB). Use Audacity to reduce the audio file size

   For an audio file of 3 min long (2m 56s), the process duration is of 1m 22s with a cost of around 0.02$ using Whisper-1 ASR model and GPT-4 for summarizing, extracting key points and notes about the meeting atmosphere

7. (Optional) Use local installation of whisper without python scripts:
   
   ```jsx
   // using the default model (the smallest one)
   // transcribe single audio file
   whisper "filename.wav"
   // or
   whisper filename.wav
   // by default this command will trigger the transcribe task

   // transcribe multiple files
   whisper filename1.wav filename2.wav

   // use a different model, medium for example
   // if this model was not used before, first it will download it
   // the processing of the audio file takes a lot longer this time
   // some noticeable things it's that it will include some additional punctuation
   whisper filename.wav --model medium

   // specify a specific language instead of using auto detection
   whisper german.wav --language German

   //specify separate tasks, for instance to translate one text from one language to another
   whisper german.wav --language German --task translate

   // specify an output directory to write the transcription into files
   whisper filename.wav --output_dir /location/dir

   //use help to find out all the arguments to be used with whisper
   whisper --help
   ```
   ## Issues Encountered

   ### SSL Error

      - when running: `whisper “file.wav”`

      Solution:

      ```jsx
      ln -s /etc/ssl/* /Library/Frameworks/Python.framework/Versions/3.xx/etc/openssl
      ```

## Info
- by default (as of August 2023) Whisper AI uses the small model out of the rest available. See the README.md file for all available models: https://github.com/openai/whisper
    - in general:
        - the larger the model, the better the quality you’ll get but also a GPU capable of running that is required
        - also the larger the model, it will also take a longer time to process

### Supported Languages

On the Github page documentation, you can find out all the supported languages: https://github.com/openai/whisper
In general, the lower the number specified for each language, the higher the quality of the transcription

### Official Documentation

To create the transcription and create summaries, extract meeting minutes and so on using the remote Whisper + GPT-4
Follow this tutorial: https://platform.openai.com/docs/tutorials/meeting-minutes