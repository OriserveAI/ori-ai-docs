Voice Management Endpoints
==========================

These endpoints allow you to list available voices and create custom voice clones.

GET /v1/audio/voices
---------------------

Retrieve a list of all available voices including predefined and cloned voices.

**Request:**

.. code-block:: bash

   curl -X GET "https://ori-tts-test.oriserve.com/v1/audio/voices" \
     -H "Authorization: Bearer your-api-token"

**Response:**

.. code-block:: json

   {
     "0fQopeE3S42LxYNQSllH": {
       "voice_name": "default_voice",
       "primary_languages": ["hi", "en-IN"],
       "custom": false
     },
     "abc123xyz": {
       "voice_name": "my_custom_voice",
       "primary_languages": ["hi"],
       "custom": true
     }
   }

The response is a dict keyed by ``speaker_id``. Each value is a voice object with the following fields:

**Response Fields:**

.. list-table::
   :header-rows: 1
   :widths: 20 15 65

   * - Field
     - Type
     - Description
   * - ``voice_name``
     - string
     - Human-readable name of the voice
   * - ``primary_languages``
     - array
     - List of language codes supported by this voice
   * - ``custom``
     - boolean
     - ``true`` for cloned voices, ``false`` for predefined voices

**Status Codes:**

- ``200 OK``: Success
- ``401 Unauthorized``: Invalid or missing authentication
- ``500 Internal Server Error``: Error fetching voice list

POST /v1/voice_clone
--------------------

Clone a voice from an audio sample. The audio must be between 5 seconds and 5 minutes in duration.

**Request:**

.. code-block:: bash

   curl -X POST "https://ori-tts-test.oriserve.com/v1/voice_clone" \
     -H "Authorization: Bearer your-api-token" \
     -F "speaker_name=my_voice" \
     -F "user_id=user123" \
     -F "audio=@sample.wav"

**Request Parameters (multipart/form-data):**

.. list-table::
   :header-rows: 1
   :widths: 20 15 10 55

   * - Parameter
     - Type
     - Required
     - Description
   * - ``speaker_name``
     - string
     - Yes
     - Name for the cloned voice
   * - ``user_id``
     - string
     - Yes
     - User identifier
   * - ``audio``
     - file
     - Yes
     - Audio file (WAV, MP3, etc.)

**Audio Requirements:**

- **Minimum duration**: 5 seconds
- **Maximum duration**: 5 minutes (300 seconds)
- **Format**: WAV recommended, other formats supported via librosa

**Response (Success):**

.. code-block:: json

   {
     "message": "Voice Clone ID: abc123xyz\n (PLEASE SAVE THIS FOR LATER USAGE,YOU WON'T BE ABLE TO SEE THIS MESSAGE AGAIN)",
     "user_id": "user123",
     "speaker_name": "my_voice",
     "voice_id": "abc123xyz"
   }

**Response (Error - Duration too short):**

.. code-block:: json

   {
     "message": "Audio duration is too small < 5 seconds could not clone audio"
   }

**Response (Error - Duration too long):**

.. code-block:: json

   {
     "message": "Audio duration is too long > 5 minutes could not clone audio"
   }

**Status Codes:**

- ``200 OK``: Voice cloned successfully
- ``400 Bad Request``: Error during cloning process
- ``401 Unauthorized``: Invalid or missing authentication
- ``422 Unprocessable Entity``: Audio duration out of range

**Python Example:**

.. code-block:: python

   import requests

   with open("sample.wav", "rb") as audio_file:
       response = requests.post(
           "https://ori-tts-test.oriserve.com/v1/voice_clone",
           headers={"Authorization": "Bearer your-api-token"},
           files={"audio": ("sample.wav", audio_file, "audio/wav")},
           data={
               "speaker_name": "my_custom_voice",
               "user_id": "user123"
           }
       )

   result = response.json()
   voice_id = result.get("voice_id")
   print(f"Created voice with ID: {voice_id}")

**Using the Cloned Voice:**

Once cloned, use the ``voice_id`` and ``user_id`` in your TTS requests:

.. code-block:: python

   # HTTP endpoint
   response = requests.post(
       "https://ori-tts-test.oriserve.com/v1/audio/speech",
       headers={
           "Authorization": "Bearer your-api-token",
           "Content-Type": "application/json"
       },
       json={
           "input": "Hello with my custom voice!",
           "voice": voice_id,  # Use the cloned voice ID
           "user_id": "user123", # required to used cloned voices
           "language": "en-IN"
       }
   )
