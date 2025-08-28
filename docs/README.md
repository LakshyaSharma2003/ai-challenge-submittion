# Technical Documentation

-----

# AI Vision Chatbot - Technical Documentation

## 📄 Project Overview

This project is a sophisticated, single-page AI chatbot that brings multimodal conversational AI directly into a web browser. It combines real-time computer vision, voice interaction, and advanced language understanding through the Google Gemini API to create a rich and context-aware user experience.

The entire application is self-contained in a single HTML file, making it incredibly portable and easy to run without any complex backend setup or dependencies.

-----

## ✨ Our Approach

Our approach is centered on creating a **"serverless" or client-centric AI experience**. The goal was to build a powerful multimodal agent that runs almost entirely in the user's browser, maximizing privacy and minimizing latency.

What makes this solution unique is the fusion of two distinct AI functionalities on the client-side:

1.  **Live Environmental Awareness**: Instead of just analyzing static images, the application uses **TensorFlow.js** to run a lightweight computer vision model (MobileNet) directly in the browser. This gives the AI a continuous, real-time understanding of what the user is pointing their camera at.
2.  **Advanced Reasoning with Context**: This live "vision context" is then intelligently packaged with user prompts (text, voice, or a specific screenshot) and sent to a powerful large vision model (**Google Gemini**). This allows for much richer interactions than a standard chatbot, as the AI can reason about what the user is currently seeing.

This architecture creates a dynamic and interactive experience where the AI is not just a passive respondent but an active participant in the user's environment.

-----

## 🛠️ Technical Stack

The project relies exclusively on modern browser APIs and open-source JavaScript libraries. No backend server is required.

| Technology / Library | Link | Purpose |
| :--- | :--- | :--- |
| **Google Gemini API** | [ai.google.dev/docs/gemini\_api\_overview](https://ai.google.dev/docs/gemini_api_overview) | Powers the core conversational AI and image analysis. |
| **TensorFlow.js** | [tensorflow.org/js](https://www.tensorflow.org/js) | The core library for running machine learning models in the browser. |
| **MobileNet Model** | [github.com/tensorflow/tfjs-models/tree/master/mobilenet](https://github.com/tensorflow/tfjs-models/tree/master/mobilenet) | A pre-trained, lightweight model for fast and efficient object detection. |
| **TailwindCSS** | [tailwindcss.com](https://tailwindcss.com) | A utility-first CSS framework for rapidly building the modern UI. |
| **Web Speech API** | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API) | Provides native browser support for speech-to-text and text-to-speech. |
| **MediaDevices API** | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices) | Used to access the user's camera (`getUserMedia`) and screen (`getDisplayMedia`). |

-----

## 🏗️ Technical Architecture

The application operates on a client-side data flow, processing as much as possible within the browser before querying the external Gemini API.

flowchart TD
    subgraph "User's Browser (Client-Side)"
        A[Camera / Screen Share] -->|Video Stream| B(HTML <video> Element);
        B -->|Frames| C{TensorFlow.js & MobileNet};
        C -->|'Current Prediction'| D[UI State];
        
        E[Microphone] -->|Audio| F[Web Speech API: Speech-to-Text];
        G[Keyboard Input] -->|Text| H{Interaction Handler};
        F -->|Transcript| H;
        
        I[Screenshot Button] -->|Base64 Image| H;
        
        D -->|Context| H;
        
        H -->|Formatted Prompt (Text + Image?)| J[API Request Module];
    end

    subgraph "Google Cloud"
        K(Google Gemini API);
    end

    J -->|HTTPS Request| K;
    K -->|AI Response| J;
    J -->|Parsed Text| L[UI Update & Speech Synthesis];
    
    L --> M[Chat Display];
    L --> N[Web Speech API: Text-to-Speech];
    
    style A fill:#bde0fe,stroke:#333
    style E fill:#bde0fe,stroke:#333
    style G fill:#bde0fe,stroke:#333
    style I fill:#bde0fe,stroke:#333
```

**Data Flow Explained:**

1.  **Video Input**: The `MediaDevices` API streams video from the user's camera or screen into a `<video>` element.
2.  **Real-Time Prediction**: `TensorFlow.js` continuously grabs frames from the video stream and feeds them to the **MobileNet** model. The top prediction (e.g., "laptop") is stored as the `currentPrediction` state.
3.  **User Interaction**: The user can interact via text, voice, or by taking a screenshot.
4.  **Context Aggregation**: The `handleInteraction` function gathers the user's direct input along with the contextual data available at that moment (`currentPrediction` and any attached screenshot).
5.  **API Call**: The data is formatted into a JSON payload and sent to the **Google Gemini API**. If a screenshot is attached, it's converted to a base64 string and included in the request.
6.  **Response Handling**: The API's response is parsed, displayed in the chat window, and spoken aloud using the **Web Speech API**.

-----

## ⚙️ Implementation Details

The core logic is contained within a single `<script>` tag in the HTML file.

### Key Functions:

  * **`run()`**: The main asynchronous function that initializes the application. It requests camera permissions, loads the MobileNet model, and sets up all event listeners.
  * **`predictLoop()`**: A continuous loop using `requestAnimationFrame` that feeds video frames to the MobileNet model for classification. This function is what enables the live "vision."
    ```javascript
    async function predictLoop() {
        if (model && video.srcObject) { 
            const predictions = await model.classify(video);
            if (predictions && predictions.length > 0) {
                currentPrediction = predictions[0].className.split(',')[0];
                predictionText.innerText = `${currentPrediction}`;
            }
        }
        requestAnimationFrame(predictLoop);
    }
    ```
  * **`getGeminiResponse(prompt, imageData)`**: This function constructs and sends the request to the Gemini API. It dynamically builds the `parts` array to include the text prompt and, optionally, the base64-encoded image data.
    ```javascript
    // Inside getGeminiResponse()
    const parts = [];
    if (imageData) {
        const base64EncodedImage = imageData.split(',')[1];
        parts.push({
            "inline_data": { "mime_type": "image/jpeg", "data": base64EncodedImage }
        });
    }
    parts.push({
        text: `Context: The user is looking at a '${currentPrediction}'. User's query: "${prompt}"`
    });
    // ...fetch call with these parts
    ```

-----

## 🚀 Installation Instructions

This project is designed for simplicity. No build steps or server hosting are required.

1.  **Get a Gemini API Key**:

      * Go to [Google AI Studio](https://aistudio.google.com/).
      * Click on **"Get API key"** and create a new key.

2.  **Download the Code**:

      * Save the provided code as an `index.html` file on your computer.

3.  **Run the Application**:

      * Open the `index.html` file in a modern web browser (like Google Chrome or Firefox).
      * The browser will ask for permission to use your camera and microphone. Please **Allow** them.
      * Click the **settings icon** (⚙️) in the top-right corner.
      * Paste your Gemini API key into the input field and click **Save**.

The application is now ready to use\!

-----

## 📖 User Guide

The interface is designed to be intuitive and straightforward.

  * **Chatting with Text**: Type your question in the input box at the bottom and press Enter or the send button (🔼).
  * **Chatting with Voice**: Click the large **microphone button** (🎤) at the bottom center. It will glow red to indicate it's listening. Speak your command, and it will be transcribed automatically.
  * **Capturing a Screenshot**: Click the **camera button** (📷) on the bottom left. This freezes the current video frame and displays it in the chat. You can then ask a question about that specific image.
  * **Sharing Your Screen**: Click the **monitor button** (🖥️) on the bottom left to switch the video source from your webcam to your screen. The button turns red to indicate screen sharing is active. Click it again to stop.

-----

## 🌟 Salient Features

  * **Zero Installation**: Runs from a single HTML file in any modern browser.
  * **Serverless Architecture**: All processing is done client-side, enhancing user privacy and speed.
  * **Multimodal Inputs**: Accepts text, voice, and visual (screenshot/live video) inputs for rich interaction.
  * **Real-Time Context**: The AI is constantly aware of the primary object in the camera's view, allowing it to answer questions like "What can I do with this?".
  * **Screen Share Analysis**: The AI can "watch" your screen and answer questions, making it a useful tool for pair programming or troubleshooting.
  * **Voice-Enabled**: Full hands-free experience with voice commands and audible responses.
