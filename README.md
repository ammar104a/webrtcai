WebRTC AI Voice Chat

Goal:
This project should implement a WebRTC-based AI voice chat system. The system allows a user to connect via WebRTC, speak into their microphone, and have the speech processed by an AI model, which responds with synthesized speech. The synthesized speech is then played back to the user through the same WebRTC connection.

Features:
Speech Recognition: Using the Whisper model for converting speech to text.
AI Response Generation: Using Ollama (LLaMA3 model) to generate AI responses.
Speech Synthesis: Initially Uses Eleven Labs Api Text to Speech but temporarily Using Bark for converting AI-generated text to speech because of eleven labs quota issues and bugs.
WebRTC: Real-time communication for audio data exchange between the client and the server. Connects Using Ice Candidates, Sends and recieves audio files in .wav format.

Installation and Setup:
1) This needs ollama to run and llama3 model works  most efficiently.
pip install ollama
pip install -r requirements.txt
2. Pull Ai Model
ollama pull llama3
3. Run Server
python server.py
will run on http://localhost:8080 

Code Structure
1. server.py
sets up an HTTP server using aiohttp and handles WebRTC connections, Receives SDP offers from clients and returns SDP answers.
Establishes and maintains peer connections, including handling ICE candidates Handles incoming messages via data channels, such as commands to start/stop recording or change models/presets.
Manages audio recording and playback through the State and PlaybackStreamTrack classes.

3. state.py
Maintains the state of each WebRTC connection, including the management of audio tracks and buffer.
Captures and buffers incoming audio frames, Keeps track of the current recording state, buffers, and filenames.

3. playback_stream_track.py
Custom implementation of MediaStreamTrack for handling the playback of synthesized responses or silence.
Chooses between playing the generated response or silence based on the current state.
Handles the streaming of audio frames to the client through the WebRTC connection.

4. audio_utils.py
Provides utility classes for speech-to-text (Whisper) and text-to-speech (Bark(Temporary)) processing.
Whisper: transcribing audio into text.
Bark/ElevenLabs: used for generating speech from text with customizable(Bark only right now) voice presets.

5. chain.py
Sets up the LLama3 model for generating responses.
Allows switching between different models as specified by the user.
Allows to preset the Ais responses/personality.

7. client.js
Handles the client-side logic for establishing the WebRTC connection, sending/receiving data, and interacting with the user interface.
Initiates and manages the WebRTC connection with the server. Sends and receives messages through data channels, such as commands and AI-generated responses.
UI Interaction: Updates the UI based on the WebRTC connection state, recording status, and playback.

7. index.html
Audio Player: <audio id="remoteAudio"> is used to play back the synthesized response.BUT THIS IS NOT WORKING


Proposed Application Flow
1. The user accesses the UI via index.html in their web browser and the client-side JavaScript (client.js) initiates a WebRTC connection to the server.
2. The server.py receives the SDP offer from the client and responds with an SDP answer, completing the WebRTC handshake.
3. The client sends a start_recording message through the data channel, prompting the server to start recording the incoming audio stream.
4. Once recording stops, the server uses the Whisper model (Whisper class in audio_utils.py) to transcribe the recorded audio into text.
5. The transcribed text is passed to the LLaMA3 model through the Chain class, generating a response.
6. The response text is then passed to the Bark model (Bark class in audio_utils.py), which converts the text into a .wav file.
7. The server uses the PlaybackStreamTrack class to stream the synthesized .wav file back to the client via WebRTC. The audio plays on the client side through the <audio> element.
8. The process can loop if the user continues the conversation, or it can end when the WebRTC connection is closed.


Problems/Needed Features:
1) (Problem) Audio Not Playing Back And Only Saving To Latest Directory:
Ensured that the bark_out.wav file is correctly generated and in a standard WAV format and verified that the WebRTC connection is properly established and that the audio stream is correctly linked to the audio element in the HTML But Audio Is still Not Playing.
2) (Needed Feature) Eleven Labs Api Is Needed To Produce Ai Voices That Sound Human But Api Is Not Working Even Though Quota in Eleven Labs Account Is Going Up.
3) (Needed Feature) Silence Detection, as user should be able to use this service like a call so the recording buttons should be removed and silence detetion should be added @Asim is working on this.
4) (Needed Feature and Problem) Support For Multiple Connections) This feature shoul theoretically work as the code supports this but this is untested, I hosted this app on vercel, connection starts but getting an error.![image](https://github.com/user-attachments/assets/91d7ed5a-502c-4092-84d5-b90b4ed5a4bc)
5) (Needed Feature) Custom Url Generation: This is low Priority as this functionality can be added to chain.py at any time and the Ai will say something like, "Hi im {{chosen Name}} from {{chosen Restaraunt/link}}, what can i do for you today?".
