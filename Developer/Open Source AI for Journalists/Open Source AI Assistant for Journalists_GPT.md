
# Open Source AI Assistant for Journalists – Design and Implementation

## Introduction

Journalists increasingly rely on digital tools for research, content creation, and multimedia production. This report presents a versatile AI assistant software suite tailored for newsroom use. The solution integrates open source components to support journalists in online research, AI-assisted writing, video editing, photo editing, text-to-speech (TTS), and transcription tasks. It is designed to be fully deployable on Windows OS and accessible to multiple users over a local network, emphasizing an intuitive experience for beginners. Key design goals include a user-friendly graphical interface, a modular architecture for easy updates, and strict attention to data privacy and security (all tools are self-hosted, with no requirement to send sensitive data to external cloud services).

Why Open Source? Open source tools allow full control over the software stack and data. Users can inspect or modify the code and avoid vendor lock-in. Crucially, open source licenses (e.g. GPL, Apache, MIT) permit free deployment and customization in a newsroom environment. Each chosen component is hosted on GitHub or similar platforms, ensuring transparency and an active community for support.

Below, we detail each functional module of the assistant, including recommended open source tools, installation and usage guidance, UI design considerations, deployment architecture for multi-user access, and comparisons of alternative tools. A summary table of candidate tools is provided for each functionality, covering features, licenses, and system requirements.

## System Architecture and Deployment

The AI assistant is built with a modular client–server architecture to accommodate multiple users. At its core is a central server application (running on a Windows machine) that hosts the AI and coordination modules. Journalists on the local network access the assistant via a web-based interface or dedicated client app, meaning several users can use the system concurrently through their web browser or a lightweight frontend. This design ensures a single installation can serve an entire newsroom.

Architecture Overview: The central server encompasses several modules: a web UI backend, an AI text generation engine, a search/retrieval module, a TTS generator, and a transcription service. These modules communicate internally but are loosely coupled (each can be updated or replaced independently). Users connect over HTTP within the LAN to the server, which handles user requests (e.g. running a web search or generating a text draft) and returns results. For heavier interactive tasks like video and image editing, the assistant leverages desktop applications (open source editors) launched on each user’s PC, since these involve complex GUIs and real-time user interaction.

Each journalist’s workstation would have the open source editing tools (video editor, photo editor) installed locally, while the server provides centralized AI services and coordination. Shared assets (like research data or media files) can be stored on a network drive or the server, allowing collaboration and easy access.

Key benefits of this approach include:

- Multi-user support: The web-based interface inherently supports concurrent use; the server can queue or parallelize AI tasks for multiple requests.
    
- Modularity: Each functional service (search, writing AI, TTS, etc.) runs as an independent module or service. For example, the transcription engine can be updated (to a new model) without affecting other components.
    
- Platform compatibility: Only the server needs to be Windows (per requirement), but the open source tools chosen all support Windows clients. Users can connect from Windows PCs (or even other OS with a browser if needed).
    
- Data privacy: All data processing happens on the local network. Web searches are proxied through a self-hosted engine (no user tracking), and AI models run locally, so sensitive information (draft stories, source material, etc.) never leaves the premises.
    

Security Considerations: The server can be configured to require user authentication (so only authorized staff access it). Network communication can be over HTTPS on the LAN for encryption. Because all tools are open source and self-hosted, there are no built-in telemetry or data-sharing with third parties. The search module, for instance, does not profile users, and the video editor does not require Internet access to function. Regular software updates can be applied as needed, and the IT team retains full control over the environment.

Deployment Example: Imagine a central Windows Server in the newsroom running the AI assistant. Each journalist opens their browser to the assistant’s start page (e.g. http://news-assistant.local/). From there, they can enter a query to research via the integrated search, use an AI writing tool to draft an article, or request a transcript of an audio file. When they need to edit a video or image, the assistant provides an interface to launch the task – for example, clicking “Edit Video” might download or open the file in Shotcut on the user’s PC. This hybrid approach leverages the server for AI and coordination, and the user’s PC for heavy GUI tasks, achieving an efficient division of labor.

## User Interface Design

A top priority is an intuitive GUI appropriate for non-technical users. The interface is designed as a unified dashboard where journalists can access all functionalities from one place. Key design features include:

- A navigation sidebar or top menu with clearly labeled sections: “Research”, “Writing Assistant”, “Video Editing”, “Photo Editing”, “Text-to-Speech”, “Transcription”, etc. Beginners can easily find the tool they need.
    
- Dashboard Home: an opening screen with icons or buttons for each main function, possibly with brief tips. For example, a “New Research” button to start a web search session, or “Draft an Article” to begin AI-assisted writing.
    
- Consistent layout: Each module opens within the same application window (or browser tab) in a sub-page or panel. This consistency means users do not juggle multiple disparate applications – the assistant feels like one application.
    
- Beginner-friendly prompts and defaults: The UI guides the user on what to do. For instance, the research section might have a search bar labeled “Enter topic or keywords…”, and the writing section might show a template like “Ask the AI to draft an introduction…”. Common actions are represented with large, recognizable icons (e.g., a magnifying glass for search, a microphone icon for transcription).
    
- Integrated results and editors: Search results could be displayed directly in the research panel (using an embedded browser view), with the ability to click and highlight text to save notes. The writing assistant could feature a text editor where AI suggestions appear in a sidebar or by highlighting text (similar to how grammar suggestions appear in a word processor). Simplicity is key: for example, an “Insert suggestion” button for the AI’s draft, rather than requiring copy-paste.
    
- Visual consistency: The color scheme and typography are kept consistent across modules, reducing cognitive load. Large, legible fonts and appropriately sized UI elements cater to users who may not be tech-savvy.
    

For the multimedia tools, since we rely on existing open source applications (Shotcut, GIMP), the strategy is to integrate them as seamlessly as possible. The assistant’s UI can provide an “Open in Editor” button. For example, if a user selects an image in the assistant’s interface and clicks Edit Photo, the system will launch GIMP with that image loaded. Likewise for video: selecting a video file (or the result of a screen recording or download) and clicking Edit Video launches Shotcut with that file. This avoids overwhelming novices with having to manually open files from within those complex applications.

To help beginners, we include documentation and tutorials accessible directly in the UI. A help icon in each module can show a quick how-to (for instance, how to trim video clips in Shotcut, or how to adjust brightness in GIMP). These can be static text/image guides or even short videos. Because Shotcut and OpenShot include tutorials, we can link to or embed those resources too.

Finally, the UI is designed to be responsive and accessible. If accessed via web, it should work on various screen sizes (though primarily desktop). We use clear labels instead of jargon (e.g. say “Translate Audio to Text” instead of “ASR” for automatic speech recognition). Keyboard shortcuts and other advanced features can be available but not required. Overall, the interface aims to hide the complexity of the underlying tools, presenting just the necessary controls to the journalist.

(No separate image mockups are provided here, but the design would resemble a simple portal with module icons, and each tool’s section designed with minimal clutter. The video editing interface example below demonstrates the kind of timeline UI provided by Shotcut/OpenShot, which the user would see when editing a video.)

## Online Research Assistance via Browser Automation

Journalists often need to gather information from the web quickly and safely. The assistant’s Research module provides an integrated way to search online sources and even scrape data as needed. This module addresses tasks like web searching, clipping content from sites, and monitoring specific webpages.

Metasearch Engine: We integrate a self-hosted instance of SearxNG, which is an open source metasearch engine that aggregates results from multiple search providers (Google, Bing, Wikipedia, etc.). By using SearxNG, users can enter a query once and get a unified list of results, rather than repeating searches on different sites. Importantly, users are neither tracked nor profiled when using SearxNG, preserving privacy. The search results page opens within the assistant’s interface. From there, the user can click on a result to open the webpage in an embedded browser tab.

Browser Automation & Scraping: For cases where deeper extraction is needed (e.g., retrieving all instances of a term from a site, or scraping data tables), the assistant employs browser automation behind the scenes. We utilize Selenium WebDriver (an open source browser automation framework) which can simulate a user’s browser to load pages and interact with them programmatically. Selenium supports all major browsers on Windows and can be driven by Python, Java, etc., under the hood (we would likely use a Python integration). For example, a journalist could input a list of URLs and ask the assistant to “scrape the main content of these pages” – the Selenium module would launch a headless browser, navigate to each URL, and extract text or other data (possibly using DOM selectors or scripts). Selenium’s Apache 2.0 license allows free integration.

For more targeted scraping of static content, we can also incorporate Scrapy, a high-level web crawling framework (BSD-licensed) that is efficient for extracting structured data. Scrapy can be used to define “spiders” for specific websites (for instance, to routinely fetch new articles from a news site’s RSS feed or to gather all entries from a database).

Beginner Interface: The research UI offers a simple search bar (powered by SearxNG) and perhaps an “Advanced search” option to specify filters (date range, specific domain, etc.). The results appear much like a normal search engine. For automation tasks, rather than exposing raw Selenium or Scrapy functionality (which requires coding), the assistant provides predefined actions. For example, a user might tick a box “Deep-scan this site for PDFs” after performing a search, which triggers a crawling script in the background. Any data extracted (text, files) can be saved and shown to the user in a readable format.

Additionally, the assistant could include a browser extension or bookmarklet to quickly send the current page to the assistant’s archive or summarizer. This can streamline research: while browsing manually, a journalist can click “Send to Assistant” and the page content will be fed to the system (perhaps to be summarized or indexed).

Privacy in Research: Because the search queries go through our self-hosted SearxNG, there's no logging by third parties. The assistant could even route traffic via a local proxy or VPN if desired, but that might not be necessary. All scraped data stays on the local server.

Table: Comparison of Open Source Research Tools

|   |   |   |   |   |
|---|---|---|---|---|
|Tool/Framework|Purpose in Assistant|Key Features|License|Platform/Requirements|
|SearxNG (self-hosted)|Metasearch engine for web queries|Aggregates results from 70+ services; highly customizable search categories; no user tracking or profiling; web UI for queries|AGPL-3.0|Server runs on Python (Windows-compatible via Python3); Browser access for users|
|Selenium WebDriver|Browser automation for data collection|Automates real web browsers to simulate user actions (click, scroll, fill forms); can scrape dynamic JavaScript-heavy sites; supports Chrome, Firefox, Edge etc.|Apache-2.0|Runs on Windows with corresponding browser drivers installed (ChromeDriver, geckodriver, etc.); supports many programming languages (we use Python bindings)|
|Scrapy|Web scraping framework for structured data|Fast crawling of multiple pages, with pipelines for extracting data (e.g. CSV/JSON output); good for repetitive spider tasks (e.g. scraping all articles of a site)|BSD 3-Clause|Python-based, works on Windows; requires Python 3 and pip dependencies|

In practice, the Research module will combine these: SearxNG for general search UI, and Selenium/Scrapy in the backend for advanced scraping tasks. Beginners can initiate these through simple UI controls (no coding needed).

## AI-Powered News Writing Support

A central feature of the assistant is helping journalists draft and edit news articles using AI. This module leverages state-of-the-art open source language models that can generate human-like text, summarize information, and refine writing. All AI processing runs locally to ensure confidentiality of journalistic content.

AI Writing Model: We integrate a large language model (LLM) such as Dolly 2.0 or LLaMA 2 to power the writing assistant. Dolly 2.0 (from Databricks) is a 12-billion-parameter model that is open source and instruction-tuned for following human prompts. In other words, Dolly can produce ChatGPT-like interactive responses entirely offline, and it’s licensed for commercial use with no restrictive clauses. This makes it viable for a newsroom assistant – journalists can use it to generate article outlines, suggest headlines, or even write full draft paragraphs given some bullet points. Dolly 2.0 was trained on a high-quality dataset of human-written Q&A/instructions, which means it's skilled at following user instructions for writing tasks.

Alternatively (or additionally), LLaMA 2 from Meta AI (in 7B or 13B parameter variants) can be used, which has strong language generation capabilities and is available for commercial use (with acceptance of Meta’s license). LLaMA-2 and its fine-tuned chat versions are known for their fluency and can also serve as a backend for drafting. The choice of model can be modular – the assistant could allow swapping between available models (if, for instance, the organization later acquires a more powerful model or a specialized one for journalism).

Integration and Use: The writing assistant interface presents something akin to a word processor or rich text editor. The journalist can type normally but also has the option to “Ask AI for draft” or “Auto-complete”. For example, after typing a few notes, the user can press a “Complete this paragraph” button. The assistant then takes the current text and the user’s prompt (e.g. “Write a news story about…”) and the AI module generates a draft. The draft appears in the editor, where the user can accept it as-is or edit further. The tool can also highlight suggested changes – acting as an AI editor for tone or grammar.

Other features include:

- Headline and Lead suggestions: Given an article topic or a raw draft, the AI can propose a catchy headline or refine the lede paragraph.
    
- Style adjustments: The assistant can be instructed to rewrite text in AP style or simplify language for a general audience, etc.
    
- Summarization: The user can paste a source document (like a press release or report) and ask the AI to summarize or extract key points, aiding in research for writing.
    
- Fact-check reminders: While the AI might not fact-check itself with external data (since it’s offline), the interface can remind users to verify factual claims. (In a future extension, one could integrate a knowledge base or retrieval system, but that’s beyond scope here.)
    

The AI model runs via an open source library (e.g., Hugging Face Transformers or a local inference engine like llama.cpp for efficiency). Running a 12B model on Windows might require a decent GPU (for faster results). We recommend equipping the server with an NVIDIA GPU (with at least 16 GB VRAM) to handle the model, or using optimized quantized versions of the model to possibly run on CPU if GPU is unavailable (with some speed tradeoff).

Beginner Interface: The key is to make the AI features opt-in and transparent. The UI might have a sidebar with tips like “Type @ai draft to have the assistant continue writing from here” or simply buttons for common actions. Users should always review and edit the AI’s output – the tool is an assistant, not an autopilot. Therefore, the text editor could highlight AI-generated text in a different color until it’s reviewed or modified by the human journalist.

Privacy: All text generation happens locally; no content is sent to external APIs. This is crucial for protecting unpublished stories or sensitive assignments (for example, investigative pieces). By using open models like Dolly 2.0, we ensure that even the model itself is under a permissive license with no phoning home.

Below is a comparison of open source AI model options for the writing assistant:

Table: Open Source AI Models for Writing Support

|   |   |   |   |   |
|---|---|---|---|---|
|Model|Parameters & Capabilities|Key Features and Performance|License|System Requirements|
|Dolly 2.0 (Databricks)|12B params, instruction-tuned|ChatGPT-like interactivity for following prompts (e.g. “Write an introduction about X”); fine-tuned on human-generated Q&A, capable of content generation and summarization; first truly open instruction-following LLM (commercially usable).|Open Source (Creative Commons license for weights & data, commercial use allowed)|GPU recommended (≈16–24 GB VRAM for full model inference); can run on CPU with 8-bit quantization (slower). Windows support via Python (Transformers or ONNX runtime).|
|LLaMA 2 (Meta)|7B or 13B params (chat fine-tuned)|High-quality general-purpose LLM with strong English proficiency; available in smaller sizes (7B) that can run on modest hardware, or larger (13B+) for better output; can generate and refine text with few-shot prompting.|Custom Meta license (free for research & commercial use with acceptance)|7B can run on CPU (with 8+ GB RAM) or modest GPU; 13B ideally on GPU (12+ GB VRAM). Windows support via libraries (Transformers, llama.cpp).|

Both models would be deployed locally. Dolly 2.0 is preferred for its fully open license. LLaMA 2 offers another option if multiple models are needed (and has a larger 13B version for improved quality). HuggingFace Transformers (Apache-2.0 licensed) can be used to integrate these models easily.

## Full-Featured Video Editing Module (Shotcut Integration)

Figure: Screenshot of OpenShot video editor’s user interface, illustrating a simple timeline-based layout and preview window (suitable for beginners). Shotcut offers a similar multi-track timeline with a beginner-friendly design.

Video journalism is critical in modern news, so the assistant includes a video editing module leveraging open source software. We will integrate Shotcut, a powerful yet user-friendly video editor. Shotcut is a free, open source, cross-platform video editing program for Windows, macOS, and Linux. It supports a wide range of video formats via FFmpeg and offers a non-linear editing timeline with multiple tracks for video, audio, and images. Shotcut can handle up to 4K resolution video and includes numerous filters and effects (color grading, transitions, audio filters, etc.) that journalists might need. It’s licensed under GPLv3, which aligns with our open source mandate.

Integration Approach: Rather than reinventing a video editor UI, the assistant will launch Shotcut as an external application when the user needs to edit video. The workflow would be: through the assistant’s UI, a user might select or upload a video file (for instance, a recording of an interview). On clicking “Edit Video”, the system opens that file in Shotcut on the user’s PC. If multiple users are on different PCs, each needs Shotcut installed locally (we will provide easy installation instructions). The assistant can pass along project files or initial settings to Shotcut if needed (Shotcut supports opening with a specific file or even XML project).

For beginners, Shotcut’s interface is quite approachable: it has a preview pane, a timeline at the bottom, and panels for media files and filters. We will ensure the first-run experience is smooth: for example, shipping pre-made Shotcut project presets for common output formats (like 1080p news segment, with correct frame rate). The assistant’s documentation will include a quick tutorial on basic editing in Shotcut (cutting clips, adding subtitles, exporting video). Shotcut’s official site also offers tutorials and an FAQ which we can reference.

Alternative Option: We will also make available OpenShot as an option. OpenShot Video Editor is another GPLv3-licensed, cross-platform editor focusing on simplicity and a built-in tutorial system. Some users find OpenShot’s interface even more straightforward (fewer advanced controls visible by default). However, OpenShot can be less stable with very large projects. The assistant could allow the user to choose their preferred editor (both being open source and fairly similar in usage).

Multi-user Consideration: Each user works on their video projects independently on their machine. If collaboration is needed, project files (Shotcut uses .mlt XML project files) can be shared via the network drive. Because Shotcut does not have a networked collaborative mode, journalists would coordinate the old-fashioned way (passing project files or editing in turns). The assistant could help by storing media files on the server so everyone accesses the same source videos if needed.

System Requirements: Video editing is resource-intensive. Shotcut’s requirements include a 64-bit Windows OS and a reasonably powerful CPU/GPU for smooth editing. For HD video, at least a quad-core CPU and 8 GB of RAM are recommended, and for 4K, 16 GB RAM and a faster multi-core CPU (or even GPU acceleration) are suggested. We note this in the documentation so users plan accordingly. That said, even a mid-range modern PC or laptop can handle basic editing in Shotcut.

Table: Open Source Video Editing Tools

|   |   |   |   |   |
|---|---|---|---|---|
|Tool|Features & Usability|License|OS Support|System Requirements (approximate)|
|Shotcut|Professional-grade video editor with multi-track timeline, support for almost any format (via FFmpeg), 4K resolution support, lots of effects/filters (color correction, transitions, keyframes). Good hardware integration (can use GPU for playback). Interface is fairly beginner-friendly for basic tasks (drag-and-drop editing).|GPL-3.0|Windows, macOS, Linux|64-bit OS; Dual-core 2+ GHz CPU (quad-core for HD); 4 GB RAM minimum (8 GB for HD, 16 GB for 4K); GPU with OpenGL 2.0 support for acceleration.|
|OpenShot|Easy-to-use UI (designed for beginners with a simplified workflow); cross-platform; supports many formats (FFmpeg-based) and unlimited tracks; offers keyframe animations and effects (chroma key, transitions). Comes with a built-in tutorial and has a simpler default layout which new users appreciate.|GPL-3.0|Windows, macOS, Linux, ChromeOS|64-bit OS; Dual-core CPU (recommend 4+ cores); 4 GB RAM (8+ GB better); no dedicated GPU required but helps for high-res previews.|
|Kdenlive|Advanced open-source editor (part of KDE project) with a wide array of effects and transitions, robust titling tool, and support for proxy editing (to handle 4K smoothly). Slightly more complex UI, but very powerful (suitable if some users are more experienced).|GPL-3.0 (part of KDE)|Windows, Linux, macOS, BSD|64-bit OS; multi-core CPU; 8 GB RAM recommended; GPU optional (Kdenlive can use GPU for certain effects).|

Recommendation: Shotcut is the primary choice due to its balance of power and usability. OpenShot can be offered as a fallback for absolute beginners or for quick tasks, and Kdenlive for those requiring advanced features. All three are actively maintained open source projects.

## Photo Processing and Editing Module (GIMP Integration)

Photo editing is handled through GIMP (GNU Image Manipulation Program), the flagship open source raster graphics editor. GIMP is a cross-platform image editor available for GNU/Linux, macOS, Windows and more. It is free software (GPLv3 license), meaning we can bundle it and even customize it if needed. GIMP provides a comprehensive set of tools for photo retouching, composition, and image authoring – essentially an open source alternative to Photoshop.

For journalists, typical tasks might include: cropping and resizing photos, adjusting brightness/contrast or color levels for publication, annotating images (circling something, adding text labels), blurring faces (for privacy), and exporting in web-friendly formats. GIMP can do all of these. It supports layers, various image formats, filters, and has an extensible plugin system.

Integration Approach: Similar to the video module, the assistant will use GIMP as an external application triggered from the central interface. A journalist can select an image (or drop one into the assistant’s UI) and click “Edit Photo”. The system will launch GIMP on the user’s machine, loading that image. If the user doesn’t have GIMP installed yet, our installation guide (discussed later) will cover that; we could also detect and prompt installation if the button is clicked and GIMP isn’t found.

We might include some custom presets or scripts in GIMP to help beginners with common tasks (for example, a script to automatically export an image in multiple resolutions for web vs print). GIMP allows scripting in Python, so we could ship a small plugin that perhaps connects back to the assistant (though not strictly necessary).

Beginner Considerations: GIMP, while powerful, can be a bit daunting due to its many tools and windows. We will set it to single-window mode for simplicity (GIMP has an option to use one unified window interface, which is easier for newcomers). We will also provide a short cheat-sheet: e.g., how to crop (with the crop tool), how to use the paintbrush to annotate, and how to export to JPEG/PNG. These can be included in the assistant’s help section. For quick edits (like simple crops or rotations), we might incorporate those directly in the assistant UI using a simpler library (for instance, a lightweight image preview with a crop selection). However, for anything beyond the basics, launching GIMP is the recommended route since it has the full toolset.

Privacy & Security: Editing images with GIMP is entirely local. No data leaves the machine. This is important if, say, journalists are editing sensitive photos. GIMP does not send any usage data out by default. It even supports opening files from remote locations (like an SMB network share) if images are stored on the central server or NAS.

Alternate Tools: While GIMP is the most comprehensive, we can mention Krita as another open source graphics editor (more geared towards drawing/illustration but also capable of photo editing tasks). Krita (GPL licensed) might be preferred by some users for its UI, but since GIMP is explicitly mentioned and is more widely known for photo retouching, it remains our primary choice. Another lightweight option is Paint.NET (freeware on Windows) – however, it’s not fully open source (its source is partially available with some restrictions), so we won’t focus on it due to the open source requirement.

Summary of Photo Editing Tools:

|   |   |   |   |   |
|---|---|---|---|---|
|Tool|Purpose & Features|License|OS Support|Notes on Use|
|GIMP|Full-featured raster graphics editor. Supports advanced photo manipulation (retouching, layers, masks), drawing tools, and plugins. Suitable for tasks from simple crops to complex compositing. Often compared to Photoshop in capabilities.|GPL-3.0|Windows, macOS, Linux, others|Best for comprehensive editing. Might have a learning curve, but numerous tutorials exist. We enable beginner-friendly settings (single-window mode, etc.).|
|Krita|Open source graphics editor focused on digital painting, also usable for image editing. Has a modern UI and supports things like HDR images, but fewer photo-specific filters than GIMP.|GPL-3.0|Windows, macOS, Linux|Good if users need drawing tablet support or prefer its interface. We primarily recommend GIMP unless a user is already comfortable with Krita.|

GIMP is the recommended tool for photo editing, given its extensive capabilities and cross-platform support. Its open source nature allows customization and it has been used in professional workflows, ensuring it can handle newsroom image tasks.

## Text-to-Speech for Podcast Production

To assist with podcast production and other audio needs, the assistant includes a Text-to-Speech (TTS) module. This allows journalists to input written scripts (e.g., news articles or podcast narrations) and generate spoken audio in a natural-sounding voice. All TTS is done locally to maintain control over the audio content and voice models.

TTS Engine: We integrate an open source neural TTS system, specifically Coqui TTS or Piper. Coqui TTS is a powerful library and toolkit for text-to-speech that includes pretrained models for over 1,100 languages and many voices. It’s the successor to Mozilla’s TTS project and allows fine-tuning and voice cloning as well. Coqui’s TTS library is MPL-2.0 licensed and can run on Windows (it uses Python and can leverage GPU for faster voice synthesis). We can choose an English voice model that sounds professional and clear for podcast use (Coqui provides some high-quality pretrained voices).

Another excellent option is Piper (the successor to Mycroft’s Mimic 3). Mimic 3 was an offline neural TTS engine by Mycroft that was privacy-focused and ran locally. Piper is the newer iteration, optimized in C++ for fast performance even on modest hardware (even a Raspberry Pi). Piper is Apache-licensed and supports multiple languages and voices. It yields very natural speech and is quite fast in generating audio. Importantly, Piper/Mimic 3 can run as a standalone server with an API. This fits our multi-user architecture well: we can run a Piper TTS server on the central machine, and each user’s request for TTS will be handled by that service concurrently.

Functionality: Through the assistant’s UI, a user can select a piece of text (for example, the script of a podcast episode or a snippet of an article) and click “Convert to Speech”. They can choose from available voices (e.g., a standard male or female narrator voice, possibly more if we install multiple voice models). The TTS module then synthesizes the audio. The resulting audio file (e.g., a WAV or MP3) can be played back in the interface for review and then downloaded. This audio can be used in podcasts or video voiceovers. Because the system is local, turnaround is quick and there's no limit or cost per character (unlike some cloud TTS APIs).

We will optimize for audio quality suitable for broadcast. Many open source TTS voices today are nearly human-like. For instance, Mycroft’s Mimic voices were noted as “the most natural sounding TTS on Linux” by users, and Coqui has multi-speaker models that can produce different styles. We might include a default voice that is neutral and clear, then optionally allow advanced users to train a custom voice (if, say, the newsroom wants a voice that matches their brand, though that requires voice data and training).

Podcast Optimization: For podcast use, consistency and correct pronunciation are key. We can take advantage of the SSML support in these tools (both Coqui TTS and Piper support a subset of SSML markup for things like pronunciation hints, pauses, etc.). The assistant’s UI might include fields to adjust speech rate or emphasis for certain words (or we can expose a simple markup like putting a word in quotes and selecting “spell out” if needed for acronyms).

Privacy: Since everything is local, using TTS on sensitive text (like a script that embargoed until publication) is safe. The voices and models are stored locally too. There is no risk of a cloud provider logging the text.

Table: Open Source Text-to-Speech Tools

|   |   |   |   |   |
|---|---|---|---|---|
|Tool / Engine|Features and Voice Quality|License|Deployment|Notes|
|Coqui TTS|Cutting-edge TTS library with many pretrained models (multi-lingual and multi-speaker). Can produce very natural-sounding speech with proper model selection. Supports training new voices and fine-tuning. Python-based; can utilize GPU for faster synthesis or run on CPU.|MPL-2.0|Runs as a Python library or server on Windows; model files can be large (hundreds of MB) and loaded into memory.|Highly flexible – we can experiment with different models. We might use a specific pre-trained voice known for clarity. Requires installing Python dependencies (TensorFlow/PyTorch backend).|
|Piper (Mimic 3)|Fast, local neural TTS focused on efficiency. Provides multiple voices for different languages, with very good quality (near human for many voices). Designed to run offline even on low-end hardware. Can operate as a small HTTP server for TTS requests.|Apache-2.0|Native C++ binary or Docker; Windows build available. Can be run as a service listening on a port (for multi-user).|Lightweight and optimized. Fewer total voices available than Coqui’s repository, but quite sufficient and easier to set up. Good choice if we want a standalone service that all clients use.|

We will likely implement Piper as the default due to its simplicity and speed, and keep Coqui TTS available for more languages or custom voice training. Both ensure podcast scripts can be turned into audio without relying on any cloud service.

## Audio and Video Transcription Module (Speech-to-Text)

Transcription is invaluable for turning interviews, speeches, or podcast recordings into text. The assistant’s transcription module uses open source Automatic Speech Recognition (ASR) to convert audio (or video audio tracks) into text. This helps in quickly generating transcripts of interviews for article writing, or transcribing a recorded podcast to create show notes or captions.

ASR Engine: We use OpenAI’s Whisper model, which was open-sourced in late 2022 under the MIT license. Whisper is a highly accurate speech recognition model that supports many languages and is adept at transcribing even noisy audio and various accents. The open source code and model weights allow us to run Whisper locally. In tests, Whisper has often been a top performer in speech-to-text quality, rivaling commercial APIs. The model is somewhat heavy (the large model is ~1.5GB and needs a GPU for realtime transcription), but even the medium model (~500MB) can achieve very good accuracy with slightly slower speed on CPU.

For our purposes, we can incorporate Whisper in a couple of ways:

- Use a GUI tool like “Buzz” which is an open source app wrapping Whisper for easy use on PC. (Buzz runs Whisper offline and has an MIT license). However, since we have our own interface, we’d more directly integrate the Whisper model via Python in the backend.
    
- Provide a simple upload or record interface: The user can upload an audio file (mp3/wav) or even a video file (the system will extract audio). Then click “Transcribe”. The server runs the Whisper model on it and outputs text. The text is displayed to the user in the browser, with options to download it as a text file or copy it. We can also allow live recording via microphone for quick voice notes transcription.
    

Alternate Engine: Another tool is Vosk (by Alpha Cephei), an offline open source speech recognition toolkit that is lightweight and supports 20+ languages. Vosk models are only ~50 MB and can do real-time transcription with low latency. It also has features like speaker identification and customizable vocabulary, which could be handy for identifying speakers in an interview (though that requires some additional setup). Vosk, being Apache-2.0 licensed, is a good alternative when computational resources are limited or if real-time streaming transcription is needed. The accuracy is not as high as Whisper on complex audio, but for clear audio it’s quite serviceable and extremely fast.

We might set up both and let the user choose: “High accuracy transcription (Whisper model)” vs “Fast transcription (Vosk model)”. Whisper for final transcripts, Vosk for quick live notes perhaps.

Using the Transcription Module: The UI will have a section “Transcription”. The user can either upload a file or even select from files on the server (if, say, a podcast episode audio was produced, they could select it directly). After starting, a progress bar or spinner indicates processing. Once done, the text appears in a text box. We will also attempt basic punctuation and casing (Whisper does this by default; Vosk transcripts might be all lowercase by default, but we can post-process or use language models to punctuate if needed).

For long audio, Whisper can produce an SRT or VTT subtitle file with timestamps; we can offer that as a download to be used for captioning videos.

Privacy: Again, all transcription is local. Tools like Whisper were originally from OpenAI, but the open source release means no connection to OpenAI’s servers is needed – we run it ourselves. The MIT license of Whisper’s code and model means it’s free to use in our product. Journalists can thus transcribe sensitive recordings (investigative interviews, etc.) without uploading them to any third-party service.

Table: Open Source Speech-to-Text Tools

|   |   |   |   |   |
|---|---|---|---|---|
|Tool/Model|Description & Capabilities|License|Accuracy & Performance|Notes for Deployment|
|OpenAI Whisper|Transformer-based ASR model known for high accuracy on diverse audio (handles different languages and accents). Comes in multiple sizes (tiny to large); larger models have better accuracy. Provides automatic punctuation and even translation if needed. Completely offline and scriptable. (Buzz example uses Whisper offline, MIT license).|MIT|Accuracy: Excellent (near state-of-art) even on noisy audio; understands context for punctuation. Performance: Real-time or slower depending on model and CPU/GPU (the small model can be near real-time on CPU; large model requires a GPU for speed).|We’ll likely use the medium or large model for English for best results. Requires installing PyTorch and the model weights. Can run on Windows (possibly via Python or a compiled package).|
|Vosk|Lightweight ASR toolkit using Kaldi models. Supports 20+ languages with small models. Fast and can do streaming (word-by-word output). Lower resource usage – can run on CPU easily, even on a Raspberry Pi.|Apache-2.0|Accuracy: Good on clear speech, struggles more with heavy noise or very fast speech; vocabulary can be customized which helps in specific domains. Performance: Real-time on CPU even for long audio (zero latency streaming available).|Easy to integrate as a library (available for Python, C#, Java, etc.). Ideal for quick transcriptions or devices without a GPU. We might use it for live transcription feature. Models are downloaded (~50 MB each).|

In practice, Whisper will be the go-to for final high-quality transcripts, while Vosk is an optional secondary tool for real-time needs. Both are free and open, so the newsroom can use them without limits or fees.

## Installation and Usage on Windows

We provide comprehensive documentation to install and configure the entire assistant on Windows. The installation process comprises setting up the server environment and the client tools:

Server Installation (Windows 10/11 64-bit):

1. Prerequisites: Install Python 3 (for running backend services like SearxNG, Whisper, etc.), and ensure PowerShell is updated (if using scripts). If using GPU for AI, install CUDA drivers appropriate for the GPU.
    
2. Download Assistant Package: We will bundle the server-side software (possibly as a zip or installer). This includes our web interface application (could be a Flask or Node.js app), the machine learning model files (for Dolly, Whisper, etc., which might be a separate downloadable due to size), and configuration files.
    
3. Open Source Dependencies: The installer will also include or prompt to install open source dependencies:
    

- SearxNG: Could run via Docker or directly via Python. We’ll document how to install it. E.g., pip install searxng and use our config, or use a pre-built Docker image. On Windows, possibly the Python route is simpler.
    
- Selenium: Install browser drivers (ChromeDriver, geckodriver). We’ll include a note to download these and put them in PATH.
    
- TTS Engines: For Piper, we can include the Windows binary in our package and some default voices. For Coqui TTS, we’d use Python pip to install it and download a model.
    
- Whisper ASR: This can be installed via pip (pip install openai-whisper or via HuggingFace transformers). It will download model weights on first run; we might also provide a direct link to a particular model to download and place in a directory to avoid repetitive downloads.
    
- The AI writing model (Dolly 2.0 or LLaMA 2): we will provide instructions to download the model weights from a trusted source (e.g., Hugging Face) since they are large. Possibly we distribute a quantized smaller version for convenience.
    
- Web UI: If it’s a Python Flask app, we list required packages (Flask, FastAPI or similar, plus front-end libraries). If it’s an Electron app, we provide an installer for it.
    

5. Configuration: A guided setup script or wizard can ask for some settings – e.g., what port to run the server on, enabling/disabling certain modules. By default, we choose sensible defaults (port 8080, run at startup, etc.). For multi-user, the server machine should ideally have a static IP or hostname on the LAN – we’ll document how to set that (so colleagues can connect via http://news-assist-server:8080 for example, perhaps using the hosts file or local DNS).
    
6. Service Mode: We can set the server to run as a Windows service in the background. Instructions will detail how to register it (maybe using nssm or similar to run our script at boot, or a simpler approach: a shortcut in Startup folder if GUI-based).
    

Client Installation (User PCs):

Each journalist’s PC needs the GUI applications for video and image editing, and a modern web browser for accessing the assistant interface:

- Shotcut: We link to the official Shotcut installer for Windows (or include it). It’s a standard installer (.exe) that users can run. No special configuration needed, though we might provide a recommended settings export (Shotcut allows sharing profiles).
    
- GIMP: Similarly, provide installer link (or MSI) for GIMP. We could pre-configure single-window mode by supplying a settings file or instructions (the documentation will say “After first launching GIMP, go to Windows > Single-Window Mode” or we script it).
    
- OpenShot/Kdenlive (optional): If offering those, we include their installers as optional components.
    

We ensure all these are the latest stable versions as of 2025.

Network Access: The documentation emphasizes that the server and client PCs must be on the same local network. We mention firewall settings – e.g., Windows Defender Firewall might prompt to allow the server app to communicate on local network; the user should accept. Alternatively, instruct how to add an inbound rule for our server port.

Usage Guide: For everyday usage:

- Users open the assistant by visiting a local URL in their browser (if we went web-based) or launching a provided client app (if we built an Electron app, for instance). They log in if authentication is set; otherwise, they see the dashboard.
    
- The guide then walks through each module’s basic use. For example, “To perform web research: go to the Research tab, enter a query, and press Search. Click results to view pages, etc.” We include screenshots of each step to be clear.
    
- For writing assistant: how to open a new document, invoke AI suggestions.
    
- For video editing: how to send a file to Shotcut, plus a reminder that editing happens in Shotcut’s window.
    
- For transcription: how to upload a file and get text.
    
- How to save or export outputs (where files go by default, etc.). Perhaps transcripts and TTS outputs are saved on the server and a link given to download; our docs clarify that location.
    

We also provide a troubleshooting section: If something doesn’t work (e.g., TTS voice not playing sound, or model fails to load), common fixes (update GPU driver, check that model files are in correct folder, etc.).

Given that some components are resource-heavy, we advise on performance tuning: e.g., Whisper large model might be slow on CPU – we’ll explain that using medium model or adding a GPU will help. Or using proxy clips in Shotcut for 4K editing.

Beginner-Friendliness: The installation is packaged to be as one-click as possible. If we can, we’d create a unified installer using something like Inno Setup or Nsis that installs our server and offers to install the client apps. However, even if separate, the documentation is step-by-step with illustrations. We assume beginner users may have IT support in a newsroom, but we write the guide so that even a journalist with basic computer knowledge could follow it (e.g., “double-click install.bat to start installation”, etc.).

## Data Privacy and Security Focus

Throughout the design of this assistant, user data privacy and security have been paramount:

- Local-Only Processing: All functionalities (research, AI generation, editing, TTS, STT) are hosted locally. No journalistic data (queries, drafts, audio files, images) is sent to any cloud API or third-party server. This mitigates risks of leaks or breaches of confidential material. For example, when using the AI writer, unlike using an online service where content might be logged, here the model runs on hardware the newsroom controls, and under an open license with no usage reporting.
    
- No Tracking or Analytics: The open source tools chosen do not include hidden tracking. SearxNG explicitly does not track users or store queries. Shotcut does not require an internet connection to function and does not phone home. Similarly, our web interface will not include any external analytics scripts. If usage metrics are needed internally, it can be done in-house, but by default we keep it minimal.
    
- Encryption and Access Control: Within the network, it’s recommended to run the web interface over HTTPS (we can generate a self-signed certificate or use a LAN certificate authority). The documentation can guide setting up HTTPS for the server (or at least using Windows authentication/NTLM if integrated in an intranet). For multiple users, we can integrate with Windows domain accounts or have a simple user account system in the assistant to prevent unauthorized access.
    
- User Roles: Possibly we allow admin and normal user roles. Admin (perhaps an IT lead) can update models or change settings, whereas normal journalists just use the tools. This prevents accidental misconfiguration.
    
- Regular Updates: Using open source means updates are frequent. We encourage keeping the tools updated for security patches. Our modular setup (e.g., Python packages for AI) means it’s easy to update one part without reinstalling everything. We will provide guidance on how to update – e.g., “to update Dolly model to a new version, replace the model file; to update SearxNG, pull the latest Docker image or pip upgrade”.
    
- Backups: The architecture should be backed up regularly (especially any user-generated content stored on the server). The documentation advises on backup strategies for the server’s data directory (which may include saved transcripts, or configured prompts, etc.).
    
- Scalability and Failover: For a small local network, one machine is fine. If the user base grows, we mention the possibility of running components on separate servers (since modular – e.g., run the Whisper ASR on a different machine if needed).
    
- Legal Compliance: All tools are properly licensed; using them avoids pirated software and ensures compliance. We list licenses for transparency. The newsroom’s IT should review licenses like AGPL (SearxNG) to ensure compliance (AGPL would require sharing source if the system was exposed outside, but within internal use it’s fine). For any content generated by AI, since the models are open, there are usually no usage restrictions (just perhaps attribution if using Dolly dataset, but Dolly 2.0’s outputs are not encumbered). We mention that as needed.
    

In summary, by self-hosting and using open platforms, the assistant gives the newsroom full ownership of its data and processes. This not only protects privacy but also builds an in-house capability that can be modified or extended without vendor intervention.

## Conclusion

By combining these open source tools into an integrated package, we deliver a comprehensive AI assistant for journalism that meets the requirements:

- Windows Deployable: All components run on Windows, with installers or instructions provided.
    
- Multi-user via Network: A client-server model allows concurrent access, with central services accessible over a local web interface.
    
- Beginner-Friendly UI: A unified interface and thoughtful defaults lower the learning curve, as demonstrated by the design of each module’s interaction.
    
- Modular & Open: Each functionality is a separate open source module (search, LLM, editing software, etc.), making it easy to update parts independently and integrate new tools in the future.
    

This approach empowers journalists with AI and multimedia tools while keeping them in control of their technology and data. With clear documentation and the supporting tables of tool comparisons, the newsroom’s technical staff can confidently set up and maintain the system. Crucially, reporters and editors can focus on their stories – researching faster, writing with AI assistance, and producing polished multimedia content – without being limited by proprietary software or privacy concerns. The open source AI assistant becomes a reliable teammate in the modern digital newsroom, scalable and adaptable for years to come.

  
**