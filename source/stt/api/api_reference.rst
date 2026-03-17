API Reference
=============

Endpoint
--------

.. code-block:: text

   POST https://{server-url}/openai/v1/audio/transcriptions

**Note**: Contact the Oriserve AI Team for the ``{server-url}`` value.

Authentication
--------------

Include your API key as a Bearer token in the request header:

.. code-block:: text

   Authorization: Bearer <YOUR_API_KEY>

Request
-------

The request uses ``multipart/form-data`` encoding.

Form Parameters
~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 20 15 15 50

   * - Parameter
     - Type
     - Required
     - Description
   * - ``file``
     - file
     - ✓
     - Audio file to transcribe (WAV, MP3, etc.)
   * - ``model``
     - string
     - ✓
     - Model identifier. Use ``ori-prime-v2.3``
   * - ``language``
     - string
     - ✓
     - Language code: ``hi`` (Hindi) or ``en`` (Hinglish)
   * - ``stream``
     - string
     - ✓
     - ``"true"`` for SSE streaming, ``"false"`` for a single JSON response
   * - ``temperature``
     - string
     - No
     - Decoding temperature. Default ``"0.0"`` (deterministic)

Supported Languages
~~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Code
     - Language
   * - ``hi``
     - Hindi (pure Hindi)
   * - ``en``
     - Hinglish (Hindi-English mix)

Supported Models
~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Model
     - Description
   * - ``ori-prime-v2.3``
     - Latest production model

cURL Example
~~~~~~~~~~~~

.. code-block:: bash

   curl --location 'https://{server-url}/openai/v1/audio/transcriptions' \
     --header 'Authorization: Bearer <YOUR_API_KEY>' \
     --form 'file=@"/path/to/audio.wav"' \
     --form 'language="hi"' \
     --form 'stream="false"' \
     --form 'model="ori-prime-v2.3"' \
     --form 'temperature="0.0"'

Response
--------

Non-Streaming Response (``stream="false"``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Returns a single JSON object when the full transcription is ready.

.. code-block:: json

   {
     "text": "हां जी",
     "usage": {
       "type": "duration",
       "seconds": 2
     }
   }

.. list-table::
   :header-rows: 1
   :widths: 25 20 55

   * - Field
     - Type
     - Description
   * - ``text``
     - string
     - The complete transcribed text
   * - ``usage.type``
     - string
     - Billing unit type (``"duration"``)
   * - ``usage.seconds``
     - number
     - Duration of audio processed in seconds

Streaming Response (``stream="true"``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Returns a sequence of Server-Sent Events (SSE). Each event delivers a chunk of
the transcription incrementally as it is produced by the model.

**Event Format**

.. code-block:: text

   data: <JSON object>

**Example stream**

.. code-block:: text

   data: {"id":"transcribe-987f541b44ecfaee","object":"transcription.chunk","created":1773395231,"model":"ori-stt","choices":[{"delta":{"content":"ह"}}]}

   data: {"id":"transcribe-987f541b44ecfaee","object":"transcription.chunk","created":1773395231,"model":"ori-stt","choices":[{"delta":{"content":"ां"}}]}

   data: {"id":"transcribe-987f541b44ecfaee","object":"transcription.chunk","created":1773395231,"model":"ori-stt","choices":[{"delta":{"content":" जी"},"finish_reason":"stop","stop_reason":null}]}

   data: [DONE]

**Chunk Schema**

.. list-table::
   :header-rows: 1
   :widths: 30 15 55

   * - Field
     - Type
     - Description
   * - ``id``
     - string
     - Unique transcription request ID
   * - ``object``
     - string
     - Always ``"transcription.chunk"``
   * - ``created``
     - integer
     - Unix timestamp of the chunk
   * - ``model``
     - string
     - Model used (``"ori-stt"``)
   * - ``choices[].delta.content``
     - string
     - Transcription text fragment for this chunk
   * - ``choices[].finish_reason``
     - string or null
     - ``"stop"`` on the final chunk, ``null`` otherwise

The stream ends with the sentinel line:

.. code-block:: text

   data: [DONE]

Error Handling
--------------

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - HTTP Status
     - Meaning
   * - ``401``
     - Invalid or missing API key
   * - ``422``
     - Missing or invalid form parameters
   * - ``500``
     - Internal server error

Rate Limits
-----------

Contact the Oriserve AI Team for information about rate limits and quotas.
