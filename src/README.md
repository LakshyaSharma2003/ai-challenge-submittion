<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Chatbot - AI Assistant</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@3.11.0/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/mobilenet@2.1.0/dist/mobilenet.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        
        /* Custom Dark Mode Palette */
        :root {
            --bg-primary: #121212;
            --bg-secondary: #1E1E1E;
            --bg-surface: #252525;
            --text-primary: #E0E0E0;
            --text-secondary: #B0B0B0;
            --accent-main: #5D8A82; /* Muted teal-blue */
            --accent-light: #7AE6FF; /* Lighter blue for highlights */
            --accent-red: #EF4444; /* Alert color */
        }
        
        body {
            background-color: var(--bg-primary);
            color: var(--text-primary);
            display: flex;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
        }

        .container-panel {
            background: linear-gradient(145deg, var(--bg-secondary), #151515);
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }

        .chat-panel {
            background-color: var(--bg-surface);
            box-shadow: inset 0 2px 5px rgba(0,0,0,0.3);
        }

        .chat-panel::-webkit-scrollbar { width: 6px; }
        .chat-panel::-webkit-scrollbar-track { background: var(--bg-surface); }
        .chat-panel::-webkit-scrollbar-thumb { background: var(--accent-main); border-radius: 3px; }

        .chat-input {
            background-color: #333;
            border: 1px solid #444;
            transition: border-color 0.2s, box-shadow 0.2s;
        }

        .chat-input:focus {
            outline: none;
            border-color: var(--accent-main);
            box-shadow: 0 0 0 2px rgba(93, 138, 130, 0.4);
        }

        .mic-button {
            background-color: var(--accent-main);
            transition: background-color 0.2s, transform 0.2s;
        }
        .mic-button:hover:not(:disabled) {
            background-color: #6C9F95;
            transform: scale(1.05);
        }
        .mic-button:disabled { background-color: #444; cursor: not-allowed; }

        .action-button {
            background-color: var(--bg-surface);
            color: var(--text-secondary);
            transition: background-color 0.2s, transform 0.2s;
        }
        .action-button:hover:not(:disabled) {
            background-color: #383838;
            transform: translateY(-2px);
        }

        .blinking { animation: blinker 1.5s linear infinite; }
        @keyframes blinker { 50% { opacity: 0; } }
        
        /* Video mirror for front camera */
        #webcam { transform: scaleX(-1); }

        .loader-spin {
            animation: spin 1.5s linear infinite;
        }
        @keyframes spin {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }

        #settings-modal { transition: opacity 0.3s ease; }
    </style>
</head>
<body class="bg-gray-900 text-white flex items-center justify-center min-h-screen p-6">
    <div class="w-full h-full max-w-7xl mx-auto flex flex-col lg:flex-row p-6 gap-6 container-panel rounded-xl">

        <div class="flex-grow lg:w-2/3 flex flex-col bg-black rounded-xl overflow-hidden shadow-2xl">
            <div class="relative w-full aspect-video rounded-xl overflow-hidden">
                <video id="webcam" autoplay playsinline muted class="absolute inset-0 w-full h-full object-cover"></video>
                <canvas id="screenshot-canvas" class="hidden"></canvas>
                <div id="loader" class="absolute inset-0 bg-black bg-opacity-75 flex flex-col items-center justify-center z-30">
                    <svg class="loader-spin h-12 w-12 text-accent-main" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
                    <p id="loader-text" class="mt-4 text-xl font-medium text-text-secondary">Initializing AI Engine...</p>
                </div>
                <div class="absolute top-4 left-4 right-4 flex justify-between items-center z-20">
                    <h1 class="text-3xl font-bold text-white tracking-wide" style="text-shadow: 2px 2px 8px rgba(0,0,0,0.8);">AI Chatbot</h1>
                    <button id="settings-button" class="p-3 bg-black bg-opacity-40 rounded-full hover:bg-opacity-60 transition-colors">
                        <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
                    </button>
                </div>
                <div id="listening-status" class="absolute top-5 right-20 text-center px-4 py-2 z-20 bg-accent-red bg-opacity-90 rounded-full hidden"><p class="text-sm font-semibold text-white blinking">Listening</p></div>
                <div id="prediction-container" class="absolute bottom-20 left-4 right-4 bg-black bg-opacity-60 backdrop-blur-sm p-3 rounded-xl text-center hidden z-20">
                    <p id="prediction" class="text-lg font-semibold text-text-primary">...</p>
                </div>
                <div class="absolute bottom-4 left-4 flex gap-3 z-20">
                    <button id="screen-share-button" class="action-button p-3 rounded-full shadow-lg disabled:bg-gray-500 disabled:cursor-not-allowed">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.75 17L9 20l-1 1h8l-1-1v-3.25M7 11V9a2 2 0 012-2h6a2 2 0 012 2v2m-.447 1.616L18 13.5V17a1 1 0 01-1 1h-6a1 1 0 01-1-1v-3.5l1.447-1.884zM12 4v7"></path></svg>
                    </button>
                    <button id="screenshot-button" class="action-button p-3 rounded-full shadow-lg disabled:bg-gray-500 disabled:cursor-not-allowed">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg>
                    </button>
                </div>
                <button id="mic-button" class="mic-button absolute bottom-4 left-1/2 -translate-x-1/2 z-20 p-4 rounded-full shadow-lg disabled:bg-gray-500 disabled:cursor-not-allowed">
                    <svg class="w-7 h-7 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a3 3 0 01-3-3V5a3 3 0 116 0v6a3 3 0 01-3 3z"></path></svg>
                </button>
            </div>
        </div>

        <div class="lg:w-1/3 flex flex-col gap-4 container-panel rounded-xl p-6">
            <h2 class="text-xl font-semibold text-accent-main text-center pb-2 border-b border-gray-600">AI Assistant</h2>
            <div id="chat-display" class="chat-panel flex-grow rounded-xl p-4 overflow-y-auto space-y-4"></div>
            <div class="flex gap-3">
                <input type="text" id="chat-input" class="chat-input flex-grow rounded-full px-5 py-3 text-text-primary placeholder-text-secondary" placeholder="Ask me anything...">
                <button id="send-button" class="mic-button rounded-full p-4 transition-colors">
                    <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 10l7-7m0 0l7 7m-7-7v18"></path></svg>
                </button>
            </div>
        </div>
    </div>

    <div id="settings-modal" class="hidden fixed inset-0 bg-black bg-opacity-80 flex items-center justify-center z-50">
        <div class="bg-bg-secondary container-panel rounded-xl p-8 w-full max-w-md shadow-2xl">
            <h2 class="text-2xl font-bold mb-4 text-accent-light">Settings</h2>
            <p class="text-text-secondary mb-6">Please enter your Google Gemini API key from Google AI Studio. The key is only stored for this session.</p>
            <label for="api-key-input" class="block text-sm font-medium text-text-primary mb-2">Google Gemini API Key</label>
            <input type="password" id="api-key-input" class="block w-full bg-bg-surface border border-gray-700 rounded-lg shadow-sm py-3 px-4 text-text-primary focus:outline-none focus:ring-2 focus:ring-accent-main focus:border-accent-main">
            <div class="mt-8 flex justify-end gap-4">
                <button id="cancel-settings" class="px-6 py-3 bg-gray-700 text-text-primary rounded-lg hover:bg-gray-600 transition-colors">Cancel</button>
                <button id="save-settings" class="px-6 py-3 bg-accent-main text-white rounded-lg hover:bg-accent-light transition-colors">Save</button>
            </div>
        </div>
    </div>

    <script>
        // --- Element References ---
        const video = document.getElementById('webcam');
        const screenshotCanvas = document.getElementById('screenshot-canvas');
        const ctx = screenshotCanvas.getContext('2d');
        const loader = document.getElementById('loader');
        const loaderText = document.getElementById('loader-text');
        const predictionContainer = document.getElementById('prediction-container');
        const predictionText = document.getElementById('prediction');
        const micButton = document.getElementById('mic-button');
        const screenShareButton = document.getElementById('screen-share-button');
        const screenshotButton = document.getElementById('screenshot-button');
        const listeningStatus = document.getElementById('listening-status');
        const chatDisplay = document.getElementById('chat-display');
        const chatInput = document.getElementById('chat-input');
        const sendButton = document.getElementById('send-button');
        const settingsButton = document.getElementById('settings-button');
        const settingsModal = document.getElementById('settings-modal');
        const apiKeyInput = document.getElementById('api-key-input');
        const saveSettingsButton = document.getElementById('save-settings');
        const cancelSettingsButton = document.getElementById('cancel-settings');
        
        // --- State Variables ---
        let model; // For Mobilenet object detection
        let currentPrediction = '';
        let geminiApiKey = '';
        let currentStream; // To keep track of the active media stream (webcam or screen)
        let screenshotDataUrl = null; // Stores the last captured screenshot

        // --- Speech API Setup ---
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        const recognition = SpeechRecognition ? new SpeechRecognition() : null;
        const synth = window.speechSynthesis;

        // --- Main Application Logic ---
        async function run() {
            micButton.disabled = true;
            screenshotButton.disabled = true;
            try {
                await setupWebcam(); // Start with webcam
                loaderText.innerText = 'Loading AI Model...';
                model = await mobilenet.load();
                setupEventListeners();
                loader.classList.add('hidden');
                predictionContainer.classList.remove('hidden');
                micButton.disabled = false;
                screenshotButton.disabled = false; // Enable screenshot after setup
                addBotMessage("I am online. Please enter your Google Gemini API key in the settings to enable full chat capabilities. You can now use the webcam, share your screen, or take screenshots!");
                predictLoop(); // Start object detection loop
            } catch (error) {
                console.error("Initialization failed:", error);
                addBotMessage("Initialization failed. Please grant camera permissions and refresh the page.");
            }
        }

        function setupEventListeners() {
            if (recognition) {
                recognition.continuous = false;
                recognition.lang = 'en-US';
                recognition.onresult = event => {
                    const command = event.results[event.results.length - 1][0].transcript.trim();
                    addUserMessage(command);
                    handleInteraction(command.toLowerCase());
                };
                recognition.onerror = event => console.error('Speech recognition error:', event.error);
                recognition.onstart = () => listeningStatus.classList.remove('hidden');
                recognition.onend = () => listeningStatus.classList.add('hidden');
                micButton.addEventListener('click', () => {
                    if (synth.speaking) synth.cancel();
                    recognition.start();
                });
            } else {
                micButton.style.display = 'none';
            }

            // New event listeners for screen sharing and screenshot
            screenShareButton.addEventListener('click', toggleScreenShare);
            screenshotButton.addEventListener('click', captureAndSendScreenshot);

            sendButton.addEventListener('click', sendMessage);
            chatInput.addEventListener('keydown', e => e.key === 'Enter' && sendMessage());
            
            settingsButton.addEventListener('click', () => settingsModal.classList.remove('hidden'));
            cancelSettingsButton.addEventListener('click', () => settingsModal.classList.add('hidden'));
            saveSettingsButton.addEventListener('click', () => {
                geminiApiKey = apiKeyInput.value.trim();
                if (geminiApiKey) {
                    addBotMessage("API Key saved for this session. You can now chat with me!");
                }
                settingsModal.classList.add('hidden');
            });
        }
        
        function sendMessage() {
            const message = chatInput.value.trim();
            if (message) {
                addUserMessage(message);
                // Check if a screenshot is available to send with the message
                handleInteraction(message.toLowerCase(), screenshotDataUrl);
                chatInput.value = '';
                screenshotDataUrl = null; // Clear screenshot after sending
            }
        }

        function handleInteraction(text, imageData = null) {
            // Prioritize local commands first
            if (text.includes("what do you see") || text.includes("what is this")) {
                // FIX 2: Added backticks (`) to create a template literal string
                respond(`I see a ${currentPrediction}`);
                return;
            }
            if (text.includes("what time is it")) {
                const time = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
                // FIX 3: Added backticks (`) to create a template literal string
                respond(`The current time is ${time}.`);
                return;
            }

            // If no local command matches, use Gemini API, potentially with image data
            getGeminiResponse(text, imageData);
        }

        async function getGeminiResponse(prompt, imageData = null) {
            if (!geminiApiKey) {
                respond("Please set your Google Gemini API key in the settings first.");
                return;
            }

            addBotMessage("Thinking...", true);
            
            // FIX 4: Added backticks (`) to make the URL a valid template literal string
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash-latest:generateContent?key=${geminiApiKey}`;
            
            const parts = [];
            // Add image part if imageData is provided
            if (imageData) {
                // Remove the "data:image/jpeg;base64," prefix
                const base64EncodedImage = imageData.split(',')[1];
                parts.push({
                    "inline_data": {
                        "mime_type": "image/jpeg",
                        "data": base64EncodedImage
                    }
                });
                addBotMessage("Image sent for analysis.", false); // Confirm image sent
            }

            // Add text part
            parts.push({
                text: `You are a helpful AI assistant. Be concise and helpful. ${imageData ? 'The following image is provided for context.' : ''} Context: The user is currently looking at an object identified as '${currentPrediction}'. User's query: "${prompt}"`
            });

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        contents: [{ parts: parts }]
                    })
                });

                if (!response.ok) {
                    const errorData = await response.json();
                    throw new Error(errorData.error.message || "An unknown API error occurred.");
                }

                const data = await response.json();
                
                // Check for valid response structure before accessing parts
                if (data.candidates && data.candidates[0] && data.candidates[0].content && data.candidates[0].content.parts && data.candidates[0].content.parts[0]) {
                    const botReply = data.candidates[0].content.parts[0].text;
                    updateLastBotMessage(botReply);
                    speak(botReply);
                } else {
                    // Handle cases where the response might be blocked or empty
                    throw new Error("Received an empty or invalid response from the API. The prompt may have been blocked for safety reasons.");
                }

            } catch (error) {
                console.error("Gemini API error:", error);
                // FIX 5: Added backticks (`) to create a template literal string
                updateLastBotMessage(`Sorry, I encountered an error: ${error.message}`);
            }
        }
        
        function respond(text) {
            addBotMessage(text);
            speak(text);
        }

        function speak(text) {
            if (synth.speaking) synth.cancel();
            const utterance = new SpeechSynthesisUtterance(text);
            synth.speak(utterance);
        }

        function addUserMessage(text) {
            const messageElement = document.createElement('div');
            messageElement.className = 'flex justify-end';
            messageElement.innerHTML = `<div class="bg-accent-main rounded-lg p-3 max-w-xs text-white">${text}</div>`;
            chatDisplay.appendChild(messageElement);
            chatDisplay.scrollTop = chatDisplay.scrollHeight;
        }

        function addBotMessage(text, isThinking = false) {
            const messageElement = document.createElement('div');
            messageElement.className = 'flex justify-start';
            if (isThinking) {
                messageElement.id = 'thinking-message';
            }
            messageElement.innerHTML = `<div class="bg-gray-700 rounded-lg p-3 max-w-xs text-text-primary">${text}</div>`;
            chatDisplay.appendChild(messageElement);
            chatDisplay.scrollTop = chatDisplay.scrollHeight;
        }

        function addImageMessage(imageDataUrl, isUser = true) {
            const messageElement = document.createElement('div');
            // FIX 6: Added backticks (`) to create a template literal string for the className
            messageElement.className = `flex ${isUser ? 'justify-end' : 'justify-start'}`;
            messageElement.innerHTML = `
                <div class="${isUser ? 'bg-accent-main' : 'bg-gray-700'} rounded-lg p-1 max-w-xs">
                    <img src="${imageDataUrl}" alt="Screenshot" class="rounded-lg max-w-full h-auto">
                </div>
            `;
            chatDisplay.appendChild(messageElement);
            chatDisplay.scrollTop = chatDisplay.scrollHeight;
        }

        function updateLastBotMessage(text) {
            const thinkingMessage = document.getElementById('thinking-message');
            if (thinkingMessage) {
                thinkingMessage.querySelector('div').innerText = text;
                thinkingMessage.id = '';
            } else {
                addBotMessage(text);
            }
        }

        async function predictLoop() {
            if (model && video.srcObject) { // Only run if model loaded and video stream active
                try {
                    const predictions = await model.classify(video);
                    if (predictions && predictions.length > 0) {
                        currentPrediction = predictions[0].className.split(',')[0];
                        // FIX 7: Added backticks (`) to create a template literal string
                        predictionText.innerText = `${currentPrediction} (${(predictions[0].probability * 100).toFixed(1)}%)`;
                    } else {
                        predictionText.innerText = "No objects detected.";
                    }
                } catch (error) { 
                    predictionText.innerText = "Object detection paused.";
                }
            } else {
                predictionText.innerText = "Waiting for camera/screen...";
            }
            requestAnimationFrame(predictLoop);
        }
        
        async function setupWebcam() {
            stopCurrentStream(); // Stop any active stream first
            loaderText.innerText = 'Accessing Camera...';
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'user' } });
                currentStream = stream;
                video.srcObject = stream;
                video.style.transform = 'scaleX(-1)'; // Mirror for front camera
                return new Promise(resolve => video.onloadedmetadata = resolve);
            } catch (userFacingError) {
                try {
                    const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
                    currentStream = stream;
                    video.srcObject = stream;
                    video.style.transform = 'scaleX(1)'; // No mirror for back camera
                    return new Promise(resolve => video.onloadedmetadata = resolve);
                } catch (environmentFacingError) {
                    console.error("Could not access any camera:", environmentFacingError);
                    throw environmentFacingError;
                }
            }
        }

        // Function to stop any active video stream
        function stopCurrentStream() {
            if (currentStream) {
                currentStream.getTracks().forEach(track => track.stop());
                video.srcObject = null;
                currentStream = null;
            }
        }

        // Toggle screen sharing
        async function toggleScreenShare() {
            if (currentStream && currentStream.getVideoTracks().some(track => track.kind === 'video' && track.getSettings().displaySurface)) {
                // If currently screen sharing, switch back to webcam
                addBotMessage("Stopping screen share, switching to webcam.");
                await setupWebcam();
                screenShareButton.classList.remove('bg-accent-red', 'hover:bg-red-600');
                screenShareButton.classList.add('action-button');
            } else {
                // Start screen sharing
                addBotMessage("Starting screen share...");
                stopCurrentStream(); // Stop webcam if active
                try {
                    const stream = await navigator.mediaDevices.getDisplayMedia({ video: true });
                    currentStream = stream;
                    video.srcObject = stream;
                    video.style.transform = 'scaleX(1)'; // No mirror for screen share

                    // Listen for when screen sharing stops
                    stream.getVideoTracks()[0].onended = () => {
                        addBotMessage("Screen sharing stopped. Switching back to webcam.");
                        setupWebcam(); // Automatically switch back to webcam
                        screenShareButton.classList.remove('bg-accent-red', 'hover:bg-red-600');
                        screenShareButton.classList.add('action-button');
                    };

                    screenShareButton.classList.remove('action-button');
                    screenShareButton.classList.add('bg-accent-red', 'hover:bg-red-600'); // Indicate active screen share
                    addBotMessage("Screen sharing active!");
                } catch (err) {
                    console.error("Error starting screen share:", err);
                    addBotMessage("Could not start screen share. Please grant permissions.");
                    await setupWebcam(); // Fallback to webcam if screen share fails
                }
            }
        }

        // Capture and send a screenshot
        function captureAndSendScreenshot() {
            if (!video.srcObject) {
                addBotMessage("No active video stream to capture a screenshot from.");
                return;
            }

            screenshotCanvas.width = video.videoWidth;
            screenshotCanvas.height = video.videoHeight;
            ctx.drawImage(video, 0, 0, video.videoWidth, video.videoHeight);
            
            // Get image data as base64
            screenshotDataUrl = screenshotCanvas.toDataURL('image/jpeg', 0.9); // JPEG for efficiency
            addImageMessage(screenshotDataUrl); // Display the screenshot in chat

            addBotMessage("Screenshot captured! Type your query and press send to analyze it with the image.", false);
        }

        run();
    </script>
</body>
</html>
