Request Stitching
=================

When converting large bodies of text to speech, you may need to split the text into multiple
requests. Without stitching, each request is synthesized independently, which can cause abrupt
changes in prosody (rhythm, intonation, and pacing) at the boundaries between chunks.

**Request stitching** solves this by linking sequential WebSocket requests together using a shared
``speechReqId``. The TTS engine uses context from previous stitched requests to maintain consistent
voice prosody across the entire text, producing a natural and seamless listening experience.

.. warning::

   Request stitching is **only available via the WebSocket endpoint** (``/ori_tts_socket``).
   Passing ``stitch_request`` or ``speechReqId`` to the HTTP POST endpoint (``/v1/audio/speech``)
   will result in an error.

How It Works
------------

1. **Generate a shared request ID** — Create a unique ``speechReqId`` that will be used across all
   chunks of the same text.

2. **Send the first chunk** — Send your first text chunk over the WebSocket with
   ``stitch_request: true`` and the shared ``speechReqId``.

3. **Send subsequent chunks** — For each additional chunk, send a new message with the **same**
   ``speechReqId`` and ``stitch_request: true``. The engine uses the context from previous chunks
   to maintain natural prosody.

4. **Collect and concatenate audio** — Each response returns audio chunks as usual. Concatenate
   the audio from all responses in order to produce the final output.

.. note::

   Each stitched request must complete (``audio_streaming_complete: true``) before sending the
   next chunk with the same ``speechReqId``. The engine needs the full context of the previous
   chunk to condition the next one.

Parameters
----------

.. list-table::
   :header-rows: 1
   :widths: 20 15 65

   * - Parameter
     - Type
     - Description
   * - ``stitch_request``
     - boolean
     - Set to ``true`` to enable request stitching. When enabled, ``speechReqId`` is **required**.
   * - ``speechReqId``
     - string
     - A unique identifier shared across all chunks of the same text. Every stitched request in
       the sequence must use the same ``speechReqId``.

When to Use Request Stitching
-----------------------------

- **Long-form content** — Articles, blog posts, or documents that exceed the recommended input
  length for a single request.
- **Streaming conversational agents** — When generating speech for a dialogue system that produces
  text incrementally, stitching ensures each turn flows naturally from the previous one.
- **Chunked pipelines** — Any workflow where text is split into segments (e.g., paragraph by
  paragraph) and synthesized separately.

Python Example
--------------

.. code-block:: python

   import asyncio
   import websockets
   import json
   import base64
   import uuid

   async def stitch_tts(paragraphs, voice_id, language="hi"):
       uri = "wss://ori-tts-test.oriserve.com/ori_tts_socket"
       headers = {"Authorization": "Bearer your-api-token"}
       speech_req_id = str(uuid.uuid4())

       async with websockets.connect(uri, extra_headers=headers) as ws:
           all_audio = b""

           for paragraph in paragraphs:
               # Send stitched request
               request = {
                   "text": paragraph,
                   "voice_id": voice_id,
                   "language": language,
                   "encoding": "pcm_24000",
                   "speed": 1.0,
                   "stitch_request": True,
                   "speechReqId": speech_req_id,
               }
               await ws.send(json.dumps(request))

               # Collect audio until this chunk is complete
               while True:
                   response = await ws.recv()
                   data = json.loads(response)

                   for chunk in data.get("audio_chunks", []):
                       all_audio += base64.b64decode(chunk)

                   if data.get("audio_streaming_complete"):
                       break

           return all_audio

   # Usage
   paragraphs = [
       "Technology has transformed countless sectors.",
       "Education stands out as one of the most significantly impacted fields.",
       "From interactive tools to personalised learning, the possibilities are endless.",
   ]

   audio = asyncio.run(stitch_tts(paragraphs, "0fQopeE3S42LxYNQSllH"))

   with open("output.pcm", "wb") as f:
       f.write(audio)

JavaScript Example
------------------

.. code-block:: javascript

   const WebSocket = require('ws');
   const { v4: uuidv4 } = require('uuid');

   async function stitchTTS(paragraphs, voiceId, language = 'hi') {
     const ws = new WebSocket('wss://ori-tts-test.oriserve.com/ori_tts_socket', [], {
       headers: { 'Authorization': 'Bearer your-api-token' }
     });
     const speechReqId = uuidv4();
     const audioBuffers = [];

     await new Promise(resolve => ws.on('open', resolve));

     for (const paragraph of paragraphs) {
       ws.send(JSON.stringify({
         text: paragraph,
         voice_id: voiceId,
         language: language,
         encoding: 'pcm_24000',
         speed: 1.0,
         stitch_request: true,
         speechReqId: speechReqId,
       }));

       // Wait for this chunk to complete
       await new Promise((resolve) => {
         const handler = (event) => {
           const data = JSON.parse(event.data || event);
           data.audio_chunks.forEach(chunk => {
             audioBuffers.push(Buffer.from(chunk, 'base64'));
           });
           if (data.audio_streaming_complete) {
             ws.removeListener('message', handler);
             resolve();
           }
         };
         ws.on('message', handler);
       });
     }

     ws.close();
     return Buffer.concat(audioBuffers);
   }

   // Usage
   const paragraphs = [
     'Technology has transformed countless sectors.',
     'Education stands out as one of the most significantly impacted fields.',
     'From interactive tools to personalised learning, the possibilities are endless.',
   ];

   stitchTTS(paragraphs, '0fQopeE3S42LxYNQSllH').then(audio => {
     require('fs').writeFileSync('output.pcm', audio);
     console.log('Done');
   });

Without vs. With Stitching
---------------------------

.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - Without Stitching
     - With Stitching
   * - Each chunk is synthesized independently
     - Chunks share context via ``speechReqId``
   * - Prosody resets at each chunk boundary
     - Prosody flows naturally across chunks
   * - Noticeable audio "seams" between segments
     - Seamless, natural-sounding output
   * - Works on both HTTP and WebSocket
     - WebSocket only

FAQ
---

**Do I need to wait for each chunk to finish before sending the next?**
   Yes. Each stitched request must fully complete (``audio_streaming_complete: true``) before you
   send the next chunk. The engine needs the full audio context of the previous request.

**Can I use request stitching with the HTTP POST endpoint?**
   No. Request stitching is only available via the WebSocket endpoint. Passing ``stitch_request``
   to the HTTP endpoint will return an error.

**How should I split my text?**
   Split at natural boundaries like sentences or paragraphs. Avoid splitting in the middle of a
   sentence, as this can reduce the quality of the stitching.

**Can I reuse the same speechReqId for different texts?**
   You should generate a new ``speechReqId`` for each distinct piece of content. Reusing IDs
   across unrelated texts may produce unexpected prosody.
