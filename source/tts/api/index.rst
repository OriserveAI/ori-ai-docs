API Reference
=============

This section provides detailed documentation for all Ori TTS API endpoints.

Endpoints Overview
------------------

.. list-table::
   :header-rows: 1
   :widths: 15 10 75

   * - Endpoint
     - Method
     - Description
   * - ``/v1/audio/speech``
     - POST
     - Generate speech from text (OpenAI compatible)
   * - ``/ori_tts_socket``
     - WebSocket
     - Real-time TTS streaming via WebSocket
   * - ``/v1/audio/voices``
     - GET
     - List all available voices
   * - ``/v1/voice_clone``
     - POST
     - Clone a voice from audio file

Authentication
--------------


Include your API token in the ``Authorization`` header:

.. code-block:: text

   Authorization: Bearer <your-api-token>

Unauthorized requests will receive a ``401 Unauthorized`` response:

.. code-block:: json

   {"error": "Unauthorized"}

Base URLs
---------

The API uses two base URLs depending on the language of your request:

**Hindi and English (India):**

.. code-block:: text

   https://ori-tts-test.oriserve.com

**Other Indian languages:**

.. code-block:: text

   https://ori-tts-multi-test.oriserve.com

See the `Supported Languages`_ table below for which URL to use for each language.

Supported Languages
-------------------

.. list-table::
   :header-rows: 1
   :widths: 20 30 50

   * - Language Code
     - Language
     - Base URL
   * - ``hi``
     - Hindi
     - ``https://ori-tts-test.oriserve.com``
   * - ``en-IN``
     - English (India)
     - ``https://ori-tts-test.oriserve.com``
   * - ``bho``
     - Bhojpuri
     - ``https://ori-tts-multi-test.oriserve.com``
   * - ``ml``
     - Malayalam
     - ``https://ori-tts-multi-test.oriserve.com``
   * - ``mag``
     - Magahi
     - ``https://ori-tts-multi-test.oriserve.com``
   * - ``mai``
     - Maithili
     - ``https://ori-tts-multi-test.oriserve.com``
   * - ``mr``
     - Marathi
     - ``https://ori-tts-multi-test.oriserve.com``
   * - ``gu``
     - Gujarati
     - ``https://ori-tts-multi-test.oriserve.com``
   * - ``hne``
     - Chhattisgarhi
     - ``https://ori-tts-multi-test.oriserve.com``
   * - ``ta``
     - Tamil
     - ``https://ori-tts-multi-test.oriserve.com``
   * - ``te``
     - Telugu
     - ``https://ori-tts-multi-test.oriserve.com``
   * - ``kn``
     - Kannada
     - ``https://ori-tts-multi-test.oriserve.com``
   * - ``bn``
     - Bengali
     - ``https://ori-tts-multi-test.oriserve.com``

Supported Audio Formats
-----------------------

**MP3 Formats:**

.. list-table::
   :header-rows: 1
   :widths: 30 30 40

   * - Format
     - Sample Rate
     - Bitrate
   * - ``mp3_22050_32``
     - 22050 Hz
     - 32 kbps
   * - ``mp3_24000_48``
     - 24000 Hz
     - 48 kbps
   * - ``mp3_44100_32``
     - 44100 Hz
     - 32 kbps
   * - ``mp3_44100_64``
     - 44100 Hz
     - 64 kbps
   * - ``mp3_44100_96``
     - 44100 Hz
     - 96 kbps
   * - ``mp3_44100_128``
     - 44100 Hz
     - 128 kbps
   * - ``mp3_44100_192``
     - 44100 Hz
     - 192 kbps

**PCM Formats (16-bit signed, little-endian):**

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Format
     - Sample Rate
   * - ``pcm_8000``
     - 8000 Hz
   * - ``pcm_16000``
     - 16000 Hz
   * - ``pcm_22050``
     - 22050 Hz
   * - ``pcm_24000``
     - 24000 Hz
   * - ``pcm_32000``
     - 32000 Hz
   * - ``pcm_44100``
     - 44100 Hz
   * - ``pcm_48000``
     - 48000 Hz

**μ-law Format:**

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Format
     - Sample Rate
   * - ``ulaw_8000``
     - 8000 Hz (telephony standard)

Detailed Documentation
----------------------

.. toctree::
   :maxdepth: 1

   tts
   voice
