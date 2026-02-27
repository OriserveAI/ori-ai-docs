Troubleshooting
===============

This guide helps you diagnose and resolve common issues with OriSTT.

Common Issues
-------------

Connection Errors
~~~~~~~~~~~~~~~~~

**Problem**: Cannot connect to WebSocket server

**Possible Causes**:

1. **Incorrect WebSocket URL**

   - Verify the URL format: ``wss://{url}/connect?{parameters}``
   - Contact the AI Team for the correct URL

2. **Invalid API Key**

   - Check that your API key is correctly set in the ``.env`` file
   - Verify the Authorization header format: ``Bearer {api_key}``
   - Ensure the API key hasn't expired

3. **Network Issues**

   - Check your internet connection
   - Verify firewall settings allow WebSocket connections
   - Try connecting from a different network

**Solution**:

.. code-block:: python

   import os
   from dotenv import load_dotenv

   load_dotenv()
   api_key = os.getenv("ASR_API_KEY")

   if not api_key:
       print("ERROR: ASR_API_KEY not found in environment")
   else:
       print(f"Using API key: {api_key[:10]}...")

Audio Format Issues
~~~~~~~~~~~~~~~~~~~

**Problem**: No transcription or poor quality results

**Possible Causes**:

1. **Incorrect chunk size**

   Each audio chunk must be exactly 20 ms:

   - 8 kHz LINEAR16: 160 bytes (80 samples × 2 bytes)
   - 16 kHz LINEAR16: 320 bytes (160 samples × 2 bytes)
   - 8 kHz μ-law: 80 bytes
   - 16 kHz μ-law: 160 bytes

2. **Wrong sample rate**

   - Verify the ``sample_rate`` parameter matches your audio
   - Use 8000 or 16000 Hz only

3. **Improper base64 encoding**

   - Ensure audio data is correctly base64-encoded
   - No padding or formatting issues

**Solution**:

.. code-block:: python

   import numpy as np
   import base64

   sample_rate = 8000
   chunk_duration_ms = 20
   samples_per_chunk = int(sample_rate * (chunk_duration_ms / 1000.0))

   # For LINEAR16
   bytes_per_chunk = samples_per_chunk * 2

   print(f"Expected chunk size: {bytes_per_chunk} bytes")
   print(f"Samples per chunk: {samples_per_chunk}")

Parameter Validation
~~~~~~~~~~~~~~~~~~~~

**Problem**: Connection fails with parameter errors

**Required Parameters**:

- ``model`` - Must be ``ori-prime-v2.3``

**Valid Parameter Values**:

.. code-block:: python

   valid_params = {
       "model": ["ori-prime-v2.3"],
       "language": ["hi", "en"],
       "sample_rate": [8000, 16000],
       "filter": ["true", "false"],
       "temperature": "float between 0.0 and 1.0",
       "interruption_words": "integer >= 1"
   }

**Check Parameters**:

.. code-block:: python

   def validate_url_params(url):
       """Validate WebSocket URL parameters."""
       from urllib.parse import urlparse, parse_qs

       parsed = urlparse(url)
       params = parse_qs(parsed.query)

       # Check required parameters
       if 'model' not in params:
           print("ERROR: 'model' parameter is required")
           return False

       # Check model value
       if params['model'][0] not in ['ori-prime-v2.3']:
           print(f"ERROR: Invalid model '{params['model'][0]}'")
           return False

       # Check language
       if 'language' in params:
           if params['language'][0] not in ['hi', 'en']:
               print(f"ERROR: Invalid language '{params['language'][0]}'")
               return False

       # Check sample rate
       if 'sample_rate' in params:
           sr = int(params['sample_rate'][0])
           if sr not in [8000, 16000]:
               print(f"ERROR: Invalid sample_rate '{sr}'")
               return False

       print("✓ URL parameters are valid")
       return True

JSON Schema Issues
~~~~~~~~~~~~~~~~~~

**Problem**: Server rejects messages

**Input Schema Validation**:

.. code-block:: python

   def validate_input_message(msg):
       """Validate input message format."""
       import json

       try:
           data = json.loads(msg) if isinstance(msg, str) else msg
       except json.JSONDecodeError:
           print("ERROR: Invalid JSON format")
           return False

       # Check required fields
       if "audio" not in data:
           print("ERROR: 'audio' field is required")
           return False

       if not isinstance(data["audio"], str):
           print("ERROR: 'audio' must be a string (base64-encoded)")
           return False

       # Check optional fields
       if "time" in data and not isinstance(data["time"], (int, float)):
           print("ERROR: 'time' must be a number")
           return False

       print("✓ Input message is valid")
       return True

**Example Valid Input**:

.. code-block:: json

   {
     "audio": "BASE64_ENCODED_AUDIO_DATA_HERE",
     "time": 1718184093.435
   }

Debugging Checklist
-------------------

Before reporting an issue, verify the following:

1. ☐ WebSocket URL is correct and matches server configuration
2. ☐ API key is valid and properly set
3. ☐ Required ``model`` parameter is provided
4. ☐ Audio chunk size is exactly 20 ms
5. ☐ Sample rate matches audio data (8000 or 16000 Hz)
6. ☐ Audio is properly base64-encoded
7. ☐ JSON messages follow the correct schema
8. ☐ All parameter values are valid
9. ☐ Network connectivity is stable
10. ☐ Latest client code is being used

Enable Detailed Logging
------------------------

Add logging to diagnose issues:

.. code-block:: python

   import logging

   # Enable WebSocket debug logging
   logging.basicConfig(
       level=logging.DEBUG,
       format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
   )

   logger = logging.getLogger("websockets")
   logger.setLevel(logging.DEBUG)

Verify Audio Processing
------------------------

Test your audio processing pipeline:

.. code-block:: python

   import soundfile as sf
   import numpy as np
   import base64

   def verify_audio_processing(audio_path):
       """Verify audio file processing."""
       print(f"Checking: {audio_path}")

       # Load audio
       try:
           audio, sr = sf.read(audio_path)
           print(f"✓ Loaded audio: {len(audio)} samples at {sr} Hz")
       except Exception as e:
           print(f"✗ Error loading audio: {e}")
           return False

       # Check duration
       duration = len(audio) / sr
       print(f"✓ Duration: {duration:.2f} seconds")

       # Check format
       if audio.dtype != np.float32 and audio.dtype != np.int16:
           print(f"⚠ Unusual audio format: {audio.dtype}")

       # Simulate processing
       target_sr = 8000
       chunk_size = target_sr // 1000 * 20  # 20ms

       # Convert to int16
       if audio.dtype == np.float32 or audio.dtype == np.float64:
           audio_int16 = (audio * 32767).astype(np.int16)
       else:
           audio_int16 = audio.astype(np.int16)

       # Create first chunk
       first_chunk = audio_int16[:chunk_size].tobytes()
       encoded = base64.b64encode(first_chunk).decode()

       print(f"✓ First chunk size: {len(first_chunk)} bytes")
       print(f"✓ Base64 length: {len(encoded)} characters")
       print(f"✓ Expected chunks: {len(audio_int16) // chunk_size}")

       return True

   # Usage
   verify_audio_processing("test_audio.wav")

Connection Test
---------------

Test basic WebSocket connectivity:

.. code-block:: python

   import asyncio
   import websockets
   import json

   async def test_connection(url, api_key):
       """Test WebSocket connection."""
       print(f"Testing connection to: {url}")

       try:
           headers = {"Authorization": f"Bearer {api_key}"}
           async with websockets.connect(
               url,
               additional_headers=headers,
               ping_interval=30,
               ping_timeout=10
           ) as ws:
               print("✓ Connected successfully")

               # Send test message
               test_msg = {
                   "audio": "dGVzdA==",  # base64 "test"
                   "time": 0.0
               }
               await ws.send(json.dumps(test_msg))
               print("✓ Sent test message")

               # Wait for response
               response = await asyncio.wait_for(ws.recv(), timeout=10)
               print(f"✓ Received response: {response}")

               return True

       except websockets.exceptions.InvalidStatusCode as e:
           print(f"✗ Connection failed with status: {e.status_code}")
           if e.status_code == 401:
               print("  → Check your API key")
           elif e.status_code == 404:
               print("  → Check your WebSocket URL")
       except asyncio.TimeoutError:
           print("✗ Connection timeout")
       except Exception as e:
           print(f"✗ Connection error: {e}")

       return False

   # Usage
   url = "wss://ori-asr-test.oriserve.com/connect?model=ori-prime-v2.3&sample_rate=8000&language=hi"
   api_key = "your_api_key_here"
   asyncio.run(test_connection(url, api_key))

Performance Issues
------------------

Slow Transcription
~~~~~~~~~~~~~~~~~~

**Possible Causes**:

1. Network latency
2. Large audio files
3. Server load

**Solutions**:

- Use lower sample rate (8000 Hz vs 16000 Hz)
- Use μ-law encoding for bandwidth efficiency
- Process audio in smaller batches
- Check network speed

Memory Issues
~~~~~~~~~~~~~

**Problem**: High memory usage or out-of-memory errors

**Solutions**:

.. code-block:: python

   # Process audio in streaming fashion
   # instead of loading entire file into memory

   def process_large_file_streaming(filepath):
       """Stream large audio files in chunks."""
       import soundfile as sf

       chunk_frames = 8000 * 5  # 5 seconds at 8kHz

       with sf.SoundFile(filepath) as audio_file:
           while True:
               chunk = audio_file.read(chunk_frames)
               if len(chunk) == 0:
                   break

               # Process chunk
               yield chunk

Getting Help
------------

If you continue to experience issues:

1. **Check this troubleshooting guide**
2. **Review the API Reference**: :doc:`api_reference`
3. **Check code examples**: :doc:`examples`
4. **Contact Support**: Reach out to the Oriserve AI Team with:

   - Detailed error messages
   - Your WebSocket URL (without API key)
   - Audio file specifications
   - Client code version
   - Logs from failed attempts

Error Code Reference
--------------------

.. list-table::
   :header-rows: 1
   :widths: 15 85

   * - Status Code
     - Meaning
   * - 101
     - Switching Protocols (success)
   * - 400
     - Bad Request (invalid parameters)
   * - 401
     - Unauthorized (invalid API key)
   * - 403
     - Forbidden (insufficient permissions)
   * - 404
     - Not Found (incorrect URL)
   * - 426
     - Upgrade Required (WebSocket upgrade failed)
   * - 500
     - Internal Server Error
   * - 503
     - Service Unavailable
