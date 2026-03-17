OriSTT Documentation
====================

Welcome to the OriSTT (Oriserve Speech-to-Text) documentation. OriSTT provides
two integration methods: a simple **REST API** for file-based transcription and a
**WebSocket API** for real-time streaming — both supporting Hindi and Hinglish.

.. toctree::
   :maxdepth: 1
   :caption: Getting Started

   quickstart

.. toctree::
   :maxdepth: 2
   :caption: REST API

   api/index
   api/api_reference
   api/python_client

.. toctree::
   :maxdepth: 2
   :caption: WebSocket API

   websocket
   python_client
   examples
   troubleshooting

Overview
--------

OriSTT offers two integration paths:

**REST API**
  Upload an audio file via a single HTTP POST request and receive the
  transcription as a JSON response or a streaming SSE feed. Ideal for
  batch or on-demand transcription workflows.

**WebSocket API**
  Stream raw audio in 20 ms chunks over a persistent WebSocket connection
  for real-time, low-latency transcription. Ideal for live call-centre
  and voice-bot integrations.

Supported Languages
-------------------

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
----------------

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Model
     - Use with
   * - ``ori-prime-v2.3``
     - REST API
   * - ``ori-prime-v2.3``
     - WebSocket API

Contact
-------

Contact the Oriserve AI Team to obtain your server URL and API key.
