Python Client
=============

Full-featured Python client for streaming audio to OriSTT.

Installation
------------

Install required dependencies:

.. code-block:: bash

   pip install websockets numpy soundfile librosa samplerate python-dotenv audioop

Client Code
-----------

Complete Python client implementation:

.. code-block:: python

    import asyncio
    import websockets
    import time
    import json
    import logging
    import numpy as np
    import base64
    import librosa
    import audioop
    import os
    import soundfile as sf
    import samplerate
    from dotenv import load_dotenv

    load_dotenv()


    async def process_and_stream_audio(uri, audio_file_path, encoding="linear16"):
        """
        Process and stream audio file to OriSTT WebSocket endpoint.

        Args:
            uri (str): WebSocket connection URI with query parameters
            audio_file_path (str): Path to audio file to transcribe
            encoding (str): Audio encoding format ("linear16" or "mulaw")

        Returns:
            str: Complete transcribed text
        """
        logger = logging.getLogger("websockets")
        ori_asr_key = os.getenv("ASR_API_KEY")

        print("Processing audio...")
        sending_sample_rate = 16000
        CHUNK_SIZE = sending_sample_rate // 1000 * 20  # 20 ms chunks
        chunk_size = CHUNK_SIZE

        try:
            # Load and resample audio
            audio, original_sr = sf.read(audio_file_path)
            audio_resampled = audio

            if original_sr != sending_sample_rate:
                ratio = float(sending_sample_rate) / original_sr
                audio_resampled = samplerate.resample(
                    audio, ratio, converter_type='sinc_best'
                )

            # Convert to 16-bit PCM
            audio_int16 = (audio_resampled * 32767).astype(np.int16)

            # Encode audio
            if encoding.lower() == "linear16":
                encoded_audio_bytes = audio_int16.tobytes()
            elif encoding.lower() == "mulaw":
                encoded_audio_bytes = audioop.lin2ulaw(audio_int16.tobytes(), 2)

            # Create base64 chunks
            bytes_per_sample = 2 if encoding.lower() == "linear16" else 1
            chunk_bytes = chunk_size * bytes_per_sample
            num_chunks = (len(encoded_audio_bytes) + chunk_bytes - 1) // chunk_bytes

            base64_chunks = [
                base64.b64encode(
                    encoded_audio_bytes[i * chunk_bytes : (i + 1) * chunk_bytes]
                ).decode()
                for i in range(num_chunks)
            ]
            print("Starts audio sending stream...")
            # Connect to WebSocket
            async with websockets.connect(
                uri,
                logger=logger,
                ping_interval=30,
                ping_timeout=None,
                close_timeout=None,
                additional_headers={"Authorization": f"Bearer {ori_asr_key}"}
            ) as websocket:

                full_message = ""

                async def send():
                    """Send audio chunks to the server."""
                    for i, data in enumerate(base64_chunks):
                        payload = {
                            "audio": data,
                            "time": time.perf_counter(),
                        }
                        await websocket.send(json.dumps(payload))

                    # Send silence frames at the end
                    samples_per_frame = int(sending_sample_rate * (20 / 1000.0))
                    silence_pcm = np.zeros(samples_per_frame, dtype=np.int16)
                    silence_bytes = silence_pcm.tobytes()
                    silence_chunk = base64.b64encode(silence_bytes).decode("utf-8")

                    for _ in range(samples_per_frame):
                        await websocket.send(
                            json.dumps({
                                "audio": silence_chunk,
                                "time": time.perf_counter(),
                            })
                        )

                async def recv():
                    """Receive transcription results from the server."""
                    nonlocal full_message

                    while True:
                        try:
                            TIMEOUT = 5
                            response = await asyncio.wait_for(
                                websocket.recv(), timeout=TIMEOUT
                            )

                            # Parse response
                            data = json.loads(response)
                            status = data.get("status")
                            data_msg = data.get("data")

                            if status == "recognized" and data_msg not in ['nan']:
                                print(f"- {data_msg} - ")
                                full_message += data_msg + " "

                        except asyncio.TimeoutError:
                            break
                        except websockets.exceptions.ConnectionClosed:
                            print("\nWebSocket connection closed")
                            break
                        except Exception as e:
                            print(f"\nError receiving message: {e} \n Response received: {response}")
                            break

                # Run send and receive concurrently
                await asyncio.gather(recv(), send())

                print(f"\nComplete Transcription: {full_message}")

            return full_message

        except Exception as e:
            print(f"Error processing audio: {e}")
            return None

Save this file as client.py

Usage Example
-------------

Basic usage with command-line arguments:

.. code-block:: python

   import argparse

   async def main():
       parser = argparse.ArgumentParser(
           description="Stream audio transcription via WebSocket"
       )
       parser.add_argument(
           "-f", "--filepath",
           required=True,
           help="Input audio file path"
       )

       args = parser.parse_args()

       # Configuration
       sending_sample_rate = 16000
       language = 'hi'
       model_name = "ori-prime-v2.3"
       call_id = "sample_test_asr"
       encoding = "linear16"
       temperature = 0.0

       URL = (
           'wss://ori-asr-test.oriserve.com/connect?'
           f'model={model_name}'
           f'&sample_rate={sending_sample_rate}'
           f'&language={language}'
           f'&temperature={temperature}'
           f'&call_id={call_id}'
       )

       print(f"Processing audio file: {args.filepath}")
       print(f"WebSocket URI: {URL}")
       print("-" * 50)

       start_time = time.perf_counter()

       try:
           result = await process_and_stream_audio(
               URL, args.filepath, encoding
           )
           if result:
               print(f"\nTranscription successful!")
           else:
               print(f"\nTranscription failed!")

       except Exception as e:
           print(f"Connection error: {e}")

       end_time = time.perf_counter()
       print(f"\nTotal execution time: {end_time - start_time:.2f} seconds")

   if __name__ == "__main__":
       asyncio.run(main())

Command Line Usage
------------------

Run the client from the command line:

.. code-block:: bash

   python client.py -f /path/to/audio.wav

Library Usage
-------------

Import and use as a library in your code:

.. code-block:: python

    from client import process_and_stream_audio
    import asyncio
    ws_url = (
        "wss://ori-asr-test.oriserve.com/connect"
        "?model=ori-prime-v2.3"
        "&sample_rate=16000"
        "&language=hi"
        "&temperature=0.0"
        "&call_id=my_test_call"
    )

    result = asyncio.run(process_and_stream_audio(ws_url, "chunk.wav"))
    print(f"Transcription: {result}")

Configuration Options
---------------------

The client supports the following configuration options:

Audio Settings
~~~~~~~~~~~~~~

.. code-block:: python

   sending_sample_rate = 16000  # or 16000
   encoding = "linear16"        # or "mulaw"

Model Settings
~~~~~~~~~~~~~~

.. code-block:: python

   model_name = "ori-prime-v2.3"
   language = 'hi'              # or 'en'
   temperature = 0.0            # 0.0 for deterministic output

Connection Settings
~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   call_id = "unique_call_id"
   prompt = "payment%20bank"    # URL-encoded, max 5 words

Features
--------

* **Automatic resampling**: Handles audio files at any sample rate
* **Format conversion**: Supports LINEAR16 and μ-law encoding
* **Concurrent streaming**: Sends and receives data simultaneously
* **Silence padding**: Automatically adds silence frames for clean endings
* **Error handling**: Robust timeout and connection error handling
* **Progress logging**: Real-time transcription output

Dependencies
------------

.. code-block:: text

   websockets>=10.0
   numpy>=1.20.0
   soundfile>=0.10.0
   librosa>=0.9.0
   samplerate>=0.1.0
   python-dotenv>=0.19.0
