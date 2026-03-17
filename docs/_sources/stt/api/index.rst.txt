STT REST API
============

The OriSTT REST API provides a simple HTTP-based interface for audio transcription.
Upload an audio file via a single POST request and receive the transcription back — either
as a one-shot JSON response or as a streaming Server-Sent Events (SSE) stream.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   api_reference
   python_client

Overview
--------

* **Simple HTTP POST** — no persistent connection required
* **OpenAI-compatible endpoint** — easy drop-in for existing integrations
* **Streaming support** — receive transcription word-by-word via SSE
* **Indic language support** — Hindi and Hinglish

Authentication
--------------

All requests require a Bearer token in the ``Authorization`` header:

.. code-block:: text

   Authorization: Bearer <YOUR_API_KEY>

Contact the Oriserve AI Team to obtain your API server URL and API key.
