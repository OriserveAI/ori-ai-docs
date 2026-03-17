Python Client
=============

This page shows how to call the OriSTT REST API from Python using the standard
``requests`` library — no WebSocket dependencies required.

Installation
------------

.. code-block:: bash

   pip install requests

Non-Streaming Transcription
---------------------------

Send an audio file and receive the complete transcription in a single response:

.. code-block:: python

   import requests

   API_URL = "https://{server-url}/openai/v1/audio/transcriptions"
   API_KEY = "your_api_key_here"  # Contact Oriserve AI Team

   def transcribe(audio_path: str, language: str = "hi") -> str:
       """
       Transcribe an audio file via the OriSTT REST API.

       Args:
           audio_path: Path to the audio file (WAV, MP3, etc.)
           language:   Language code — "hi" (Hindi) or "en" (Hinglish)

       Returns:
           Transcribed text as a string.
       """
       with open(audio_path, "rb") as f:
           response = requests.post(
               API_URL,
               headers={"Authorization": f"Bearer {API_KEY}"},
               data={
                   "language": language,
                   "stream": "false",
                   "model": "ori-prime-v2.3",
                   "temperature": "0.0",
               },
               files={"file": f},
           )

       response.raise_for_status()
       return response.json()["text"]


   if __name__ == "__main__":
       text = transcribe("audio.wav", language="hi")
       print(f"Transcription: {text}")

Streaming Transcription (SSE)
------------------------------

Stream the transcription token-by-token using Server-Sent Events:

.. code-block:: python

   import requests
   import json

   API_URL = "https://{server-url}/openai/v1/audio/transcriptions"
   API_KEY = "your_api_key_here"  # Contact Oriserve AI Team

   def transcribe_stream(audio_path: str, language: str = "hi") -> str:
       """
       Transcribe an audio file and print chunks as they arrive via SSE.

       Args:
           audio_path: Path to the audio file (WAV, MP3, etc.)
           language:   Language code — "hi" (Hindi) or "en" (Hinglish)

       Returns:
           Complete transcribed text assembled from all chunks.
       """
       full_text = ""

       with open(audio_path, "rb") as f:
           response = requests.post(
               API_URL,
               headers={"Authorization": f"Bearer {API_KEY}"},
               data={
                   "language": language,
                   "stream": "true",
                   "model": "ori-prime-v2.3",
                   "temperature": "0.0",
               },
               files={"file": f},
               stream=True,
           )
           response.raise_for_status()

           for raw_line in response.iter_lines():
               if not raw_line:
                   continue

               line = raw_line.decode("utf-8")

               # SSE lines start with "data: "
               if not line.startswith("data:"):
                   continue

               payload = line[len("data:"):].strip()

               # End-of-stream sentinel
               if payload == "[DONE]":
                   break

               chunk = json.loads(payload)
               delta = chunk["choices"][0]["delta"].get("content", "")
               print(delta, end="", flush=True)
               full_text += delta

       print()  # newline after streaming
       return full_text


   if __name__ == "__main__":
       text = transcribe_stream("audio.wav", language="hi")
       print(f"\nFull transcription: {text}")

Configuration Reference
-----------------------

.. list-table::
   :header-rows: 1
   :widths: 25 20 55

   * - Variable
     - Example value
     - Notes
   * - ``API_URL``
     - ``https://{server-url}/openai/v1/audio/transcriptions``
     - Provided by the Oriserve AI Team
   * - ``API_KEY``
     - ``Ss28dCUhAd5YTwmOMpA38XFaDaldRn3W``
     - Provided by the Oriserve AI Team
   * - ``language``
     - ``"hi"`` or ``"en"``
     - Hindi or Hinglish
   * - ``model``
     - ``"ori-prime-v2.3"``
     - Currently the only supported model
   * - ``stream``
     - ``"true"`` or ``"false"``
     - SSE streaming or single response
   * - ``temperature``
     - ``"0.0"``
     - 0.0 for deterministic output

Error Handling
--------------

.. code-block:: python

   import requests

   try:
       text = transcribe("audio.wav", language="hi")
       print(f"Transcription: {text}")

   except requests.exceptions.HTTPError as e:
       print(f"API error {e.response.status_code}: {e.response.text}")

   except requests.exceptions.ConnectionError:
       print("Could not reach the API server. Check your network and server URL.")

   except Exception as e:
       print(f"Unexpected error: {e}")

Dependencies
------------

.. code-block:: text

   requests>=2.28.0
