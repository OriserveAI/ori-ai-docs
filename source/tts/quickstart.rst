Quickstart
==========

This guide will help you get started with the Ori TTS API.

Authentication
--------------

.. code-block:: text

   Authorization: Bearer <your-api-token>

.. note::

   The API has two base URLs depending on the language:

   - ``https://ori-tts-test.oriserve.com`` — for ``hi`` (Hindi) and ``en-IN`` (English, India)
   - ``https://ori-tts-multi-test.oriserve.com`` — for ``bho``, ``ml``, ``mag``, ``mai``, ``mr``, ``gu``, ``hne``, ``ta``, ``te``, ``kn``, ``bn``

   See :doc:`api/index` for the full language table.

Basic HTTP Example
------------------

The simplest way to generate speech is using the HTTP endpoint:

.. code-block:: bash

   curl -X POST "https://ori-tts-test.oriserve.com/v1/audio/speech" \
     -H "Authorization: Bearer your-api-token" \
     -H "Content-Type: application/json" \
     -d '{
       "input": "Hello, welcome to Ori TTS!",
       "voice": "0fQopeE3S42LxYNQSllH",
       "language": "en-IN",
       "response_format": "mp3_44100_128"
     }' \
     --output speech.mp3

.. code-block:: bash

    from openai import AsyncOpenAI
    import soundfile as sf
    import numpy as np
    from IPython.display import Audio
    import io

    client = AsyncOpenAI(base_url="https://ori-tts-test.oriserve.com/v1", api_key="<TOKEN>")

    def audio_header_creater(audio, channels=1, sample_rate=24000, bits_per_sample=16):
        audio_duration = len(audio)
        riff = b"RIFF" # 32 bytes
        chunk = np.array([audio_duration+36], dtype=np.int32).tobytes()
        wavfmt = b"WAVEfmt "
        bits16 = b"\x10\x00\x00\x00"
        audio_format = b"\x01\x00"
        channel_bytes = np.array([channels], dtype=np.int16).tobytes()
        sample_rate_bytes= np.array([sample_rate], dtype=np.int32).tobytes()
        byte_rate = np.array([sample_rate*channels*bits_per_sample / 8], dtype=np.int32).tobytes()
        bytes_in_frame = np.array([channels*bits_per_sample/8], dtype=np.int16).tobytes()
        bits_per_sample_bytes = np.array([bits_per_sample], dtype=np.int16).tobytes()
        data_bytes = b"data"
        file_size = np.array([audio_duration], dtype=np.int32).tobytes()

        header = riff+chunk+wavfmt+bits16+audio_format+channel_bytes+sample_rate_bytes+byte_rate+bytes_in_frame+bits_per_sample_bytes+data_bytes+file_size

        return header

    input_text = "एक खारा पानी के मुहाना जेहर पूरा नइ घेराए हवय अउ एक कोती ले पानी जाथे अउ एमा मछरी पालन करे जाथे"

    async def generate_audio():
        audio_chunks = []
        async with client.audio.speech.with_streaming_response.create(
            model="ori-tts-v2.1",
            voice="hlsNMLVWSHSwSAUpp9af", #voice id for Kavya
            input=input_text,
            response_format: "pcm_24000" # Default
            extra_body={
                "language":"hi",
                "user_id": "some-user-id",
                "speechReqId": "some-request-id",
            }
        ) as response:
            async for chunk in response.iter_bytes(chunk_size=1024):
                audio_chunks.append(chunk)

        audio_data = b''.join(audio_chunks)

        header = audio_header_creater(audio_data, sample_rate=24000)
        audio_io = io.BytesIO(header + audio_data)

        aud, sr = sf.read(audio_io)
        display(Audio(data=aud, rate=sr, autoplay=False))

WebSocket Example
-----------------

For real-time streaming, use the WebSocket endpoint:

.. code-block:: python

    def audio_header_creater(audio, channels=1, sample_rate=8000, bits_per_sample=16):
        audio_duration = len(audio)
        riff = b"RIFF" # 32 bytes
        chunk = np.array([audio_duration+36], dtype=np.int32).tobytes()
        wavfmt = b"WAVEfmt "
        bits16 = b"\x10\x00\x00\x00"
        audio_format = b"\x01\x00"
        channel_bytes = np.array([channels], dtype=np.int16).tobytes()
        sample_rate_bytes= np.array([sample_rate], dtype=np.int32).tobytes()
        byte_rate = np.array([sample_rate*channels*bits_per_sample / 8], dtype=np.int32).tobytes()
        # 8 represents mono stereo ## needs more investigation if crash
        bytes_in_frame = np.array([channels*bits_per_sample/8], dtype=np.int16).tobytes()
        bits_per_sample_bytes = np.array([bits_per_sample], dtype=np.int16).tobytes()
        data_bytes = b"data"
        file_size = np.array([audio_duration], dtype=np.int32).tobytes()

        header = riff+chunk+wavfmt+bits16+audio_format+channel_bytes+sample_rate_bytes+byte_rate+bytes_in_frame+bits_per_sample_bytes+data_bytes+file_size

        return header

    async def test_pcm_ws(encoding='pcm_24000'):
        full_audio = []
        sr = int(encoding.split('_')[-1])
        import uuid
        input_text = "यूनियन बजट 2026 की तारीख जैसे-जैसे नजदीक आ रही है कुछ सालों से भारी टैक्स और सख्त नियमों की मार झेल रहे इस सेक्टर को अब उम्मीद है कि वित्त मंत्री निर्मला सीतारमण इस बार कोई बड़ा ऐलान कर सकती हैं। लोगों का सवाल यही है कि क्या बजट 2026 में क्रिप्टो इन्वेस्टर्स को 30% टैक्स से राहत मिलेगी या सरकार का पूरा फोकस डिजिटल रुपयायानी CBDC को आगे बढ़ाने पर ही रहेगा हालांकि सरकार और RBI का नजरिया अब भी सतर्क है। ajaj.ali@gamil.com नीति-निर्माताओं की चिंता प्राइवेट क्रिप्टोकरेंसी की वोलैटिलिटी, मनी लॉन्ड्रिंग और ऑफशोर  प्लेटफॉर्म्स पर ट्रैकिंग की दिक्कतों को लेकर है। यही वजह है कि सरकार डिजिटल रुपया यानी CBDC को तेजी से आगे बढ़ा रही है। RBI समर्थित डिजिटल रुपया को सुरक्षित, रेगुलेटेड और भरोसेमंद ऑप्शन के तौर पर पेश किया जा रहा है। माना जा रहा है कि बजट 2026 में CBDC से जुड़े पायलट प्रोजेक्ट्स, टेक्नोलॉजी और यूज-केस के लिए एक्स्ट्रा  फंडिंग और इंसेंटिव्स मिल सकते हैं। कुल मिलाकर, बजट 2026 क्रिप्टो निवेशकों के लिए उम्मीद और अनिश्चितता का मिश्रण है। वित्त मंत्रीनिर्मला सीतारमण की 1 फरवरी की घोषणा"

        async with websockets.connect(
                                "wss://ori-tts-test.oriserve.com/ori_tts_socket",
                                additional_headers={
                                    "Authorization": "Bearer Ss28dCUhAd5YTwmOMpA38XFaDaldRn3W",
                                    },
                            ) as ws:
            data = {
                'text': input_text,
                'language': 'hi',
                'voice_id': 'gmRe3gT8SxqKZOCV211U',
                "encoding":encoding,
                }

            await ws.send(json.dumps(data))
            first = True
            while True:
                response_msg = await ws.recv()
                if first:
                    first=False
                response_json = json.loads(response_msg)
                end_flags = bool(response_json.get("audio_streaming_complete", "False"))
                audio_chunks = response_json.get("audio_chunks", [])
                for _, chunk in enumerate(audio_chunks):
                    full_audio.append(base64.b64decode(chunk))
                if end_flags:
                    break

        header = audio_header_creater(b"".join(full_audio), sample_rate=sr)
        f_path = "test_ws.wav"
        with open(f_path, "wb") as f:
            f.write(header + b"".join(full_audio))

Voice Cloning Example
---------------------

To clone a voice from an audio sample:

.. code-block:: bash

   curl -X POST "https://ori-tts-test.oriserve.com/v1/voice_clone" \
     -H "Authorization: Bearer your-api-token" \
     -F "speaker_name=my_custom_voice" \
     -F "user_id=user123" \
     -F "audio=@sample.wav"

.. note::

   Audio files must be between 5 seconds and 5 minutes in duration.

The response will include a ``voice_id`` that you can use in subsequent TTS requests.

Listing Available Voices
------------------------

To get a list of all available voices:

.. code-block:: bash

   curl -X GET "https://ori-tts-test.oriserve.com/v1/audio/voices" \
     -H "Authorization: Bearer your-api-token"

Next Steps
----------

* See :doc:`api/tts` for detailed TTS endpoint documentation
* See :doc:`api/voice` for voice management endpoints
