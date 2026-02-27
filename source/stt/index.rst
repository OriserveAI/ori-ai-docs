OriSTT Documentation
====================

Welcome to the OriSTT (Oriserve Speech-to-Text) API documentation. OriSTT provides real-time speech transcription via WebSocket connections with support for Hindi and Hinglish languages.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   quickstart
   api_reference
   python_client
   examples
   troubleshooting

Overview
--------

OriSTT is a real-time speech-to-text service that uses WebSocket connections for streaming audio transcription. Key features include:

* **Real-time Streaming**: Process audio in 20ms chunks for low-latency transcription
* **Multi-language Support**: Hindi and Hinglish transcription
* **Noise Reduction**: Optional audio filtering for improved accuracy
* **Word Boosting**: Enhance recognition of specific vocabulary
* **Flexible Audio Formats**: Support for LINEAR16 and μ-law encoding

Supported Languages
-------------------

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Code
     - Language
   * - ``hi``
     - Hindi
   * - ``en``
     - Hinglish (Hindi-English mix)

Supported Models
----------------

* **ori-prime-v2.3**: Latest production model with improved accuracy

Getting Started
---------------

To get started with OriSTT:

1. :doc:`quickstart` - Set up your first connection
2. :doc:`api_reference` - Understand the API parameters
3. :doc:`python_client` - Use the Python client library
4. :doc:`examples` - See practical examples

Contact
-------

For API access and the WebSocket URL, please contact the Oriserve AI Team.
