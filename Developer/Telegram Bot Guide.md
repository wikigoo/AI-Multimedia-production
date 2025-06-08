# A Practical Guide to Creating Telegram Bot Scripts Using Python

Telegram bots have emerged as powerful tools for automating tasks, delivering services, and engaging with users directly within the popular messaging application. Python, renowned for its simplicity and extensive collection of libraries, provides an excellent platform for developing these bots efficiently. This guide offers a practical walkthrough for individuals with basic Python knowledge to create their own Telegram bot scripts.

## I. Introduction to Telegram Bots and Python

### A. What is a Telegram Bot?

A Telegram bot is essentially a third-party application that runs inside Telegram. Users can interact with bots by sending them messages, commands, and inline requests.1 These bots are special accounts that do not require an additional phone number to set up.1 They can perform a wide array of functions, from providing automated responses and information to integrating with other services and managing group chats. Their popularity stems from their ability to automate processes, offer on-demand services, and create engaging user experiences directly within the Telegram interface [User Query].

### B. Why Use Python for Telegram Bot Development?

Python stands out as a preferred language for Telegram bot development due to several key advantages. Its clear syntax and readability make it accessible, even for those relatively new to programming [User Query]. More importantly, Python boasts a rich ecosystem of libraries, with python-telegram-bot being a prominent example specifically designed for interacting with the Telegram Bot API.3 This library simplifies many of the complexities involved in bot creation.

Furthermore, Python has strong community support, meaning ample resources, tutorials, and forums are available to assist developers.2 The asynchronous capabilities inherent in modern Python, often leveraged by libraries like python-telegram-bot through asyncio, make it well-suited for I/O-bound applications like bots, which need to handle multiple network requests (incoming messages, API calls) concurrently without getting bogged down.4

### C. Overview of the Telegram Bot API

The Telegram Bot API is an HTTP-based interface that allows developers to connect their bots to Telegram's servers.1 All communications with the API occur over HTTPS and require a unique authentication token for each bot.1 This token acts as the bot's password and is essential for authorizing its requests.

The API supports a wide range of functionalities, enabling bots to:

- Send and receive text messages, images, videos, documents, and other media types.1
    
- Respond to user commands (messages starting with /).
    
- Offer custom keyboards with predefined reply options.
    
- Implement inline keyboards, which are buttons attached directly to messages.
    
- Work in inline mode, allowing users to interact with the bot from any chat without adding it as a member.7
    

Bots can receive updates from Telegram in two primary ways:

1. Long Polling: The bot periodically sends requests to the Telegram server asking for new messages. This method is generally simpler to set up, especially during development.6 This guide will primarily focus on long polling.
    
2. Webhooks: The bot provides a publicly accessible HTTPS URL, and Telegram sends updates to this URL as soon as they arrive. Webhooks are typically more efficient for production bots but require a server and HTTPS setup.2
    

Telegram offers extensive documentation for its Bot API, which serves as a comprehensive reference for all available methods and types.1

## II. Setting Up Your Development Environment

Before writing any code, a proper development environment is necessary. This involves ensuring Python is installed, creating an isolated space for project dependencies, and installing the required library.

### A. Python Installation

This guide assumes a working installation of Python. The python-telegram-bot library, which will be used extensively, requires Python version 3.7 or newer.3 To check the installed Python version, one can open a terminal or command prompt and type python --version or python3 --version. If Python is not installed or the version is outdated, it should be downloaded from the official Python website and installed.

### B. Creating a Virtual Environment

Using virtual environments is a crucial best practice in Python development. A virtual environment creates an isolated space for each project, ensuring that its dependencies do not conflict with those of other projects or the system-wide Python installation.9

The venv module, included in Python's standard library, is commonly used for this purpose. To create a virtual environment:

1. Navigate to the project directory in the terminal.
    
2. Run the command:  
    Bash  
    python -m venv my_telegram_bot_env  
    (Replace my_telegram_bot_env with a preferred name for the environment directory, e.g., .venv 12).
    
3. Activate the virtual environment:
    

- On Linux or macOS:  
    Bash  
    source my_telegram_bot_env/bin/activate  
      
    
- On Windows:  
    Bash  
    my_telegram_bot_env\Scripts\activate  
      
    

Once activated, the terminal prompt will usually change to indicate the active environment.12 All subsequent package installations using pip will be confined to this environment.

### C. Installing the python-telegram-bot Library

The python-telegram-bot library is a popular, feature-rich wrapper for the Telegram Bot API in Python. It simplifies interactions with the API and provides convenient high-level classes for building bots.3

With the virtual environment activated, install the library using pip:

  

Bash

  
  

pip install python-telegram-bot --upgrade  
  

This command installs the latest stable version of the library and its necessary dependencies.3 The --upgrade flag ensures that if an older version is present, it gets updated.

The python-telegram-bot library is actively maintained and supports recent versions of the Telegram Bot API.13 While it offers optional dependencies for advanced features like job scheduling (job-queue) or Telegram Passport integration (passport), these are not required for the basic functionalities covered in the initial parts of this guide.3

## III. Your First Telegram Bot: A Simple Echo Bot

With the environment set up, the next step is to create a basic bot. This "echo bot" will simply reply to any text message it receives with the same text.

### A. Obtaining a Bot Token from BotFather

Every Telegram bot needs an authentication token to communicate with the Telegram API. This token is obtained from BotFather, a special bot provided by Telegram for managing other bots.2

Follow these steps to get a token:

1. Open the Telegram application.
    
2. Search for the user @BotFather. Ensure it's the official, verified account (it usually has a blue checkmark).14
    
3. Start a chat with BotFather and send the command /newbot.2
    
4. BotFather will ask for a display name for the bot. This can be anything.
    
5. Next, BotFather will ask for a unique username for the bot. This username must end with "bot" (e.g., MyEchoTestBot or my_echo_test_bot).2
    
6. If the name and username are valid and available, BotFather will respond with a message containing the API token.2 This token looks something like 1234567890:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw.
    

Crucially, this token is like a password for the bot. It must be kept secret and secure. Anyone with access to this token can control the bot.2

### B. Basic Bot Script Structure

A Python script for a Telegram bot using python-telegram-bot typically follows a common structure. For a simple echo bot, the script will involve importing necessary modules, setting up logging, defining the bot's token, creating handler functions for different types of updates, and then starting the bot's polling mechanism.

Here's the foundational structure:

  

Python

  
  

import logging  
import os # For accessing environment variables  
  
from telegram import ForceReply, Update  
from telegram.ext import Application, CommandHandler, ContextTypes, MessageHandler, filters  
  
# Enable logging  
# This basic configuration helps in observing the bot's behavior and diagnosing issues.  
# Many examples provided by the python-telegram-bot library include logging setup from the start,  
# indicating its importance in development. [3, 18, 19]  
logging.basicConfig(  
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO  
)  
# set higher logging level for httpx to avoid all GET and POST requests being logged  
logging.getLogger("httpx").setLevel(logging.WARNING)  
  
logger = logging.getLogger(__name__)  
  
# It is STRONGLY recommended to use environment variables for your bot token in real applications!  
# For this initial example, we'll show how it might be assigned directly, but then demonstrate  
# the environment variable approach which is much more secure.  
# BOT_TOKEN = "YOUR_BOT_TOKEN_HERE" # Replace with your actual token  
  
# Secure way to get the bot token (recommended)  
BOT_TOKEN = os.environ.get("TELEGRAM_BOT_TOKEN")  
if not BOT_TOKEN:  
    logger.error("TELEGRAM_BOT_TOKEN environment variable not set!")  
    # In a real application, you might exit here or raise an error  
    # For this guide, we'll use a placeholder if not set, but emphasize it won't work.  
    BOT_TOKEN = "YOUR_BOT_TOKEN_MUST_BE_SET_AS_ENVIRONMENT_VARIABLE"  
  
  
# Define command handlers and message handlers here (covered in next sections)  
  
def main() -> None:  
    """Start the bot."""  
    # Create the Application and pass it your bot's token.  
    # The Application class is central to PTB v20+ for setting up the bot.  
    application = Application.builder().token(BOT_TOKEN).build()  
  
    # Add command handlers (e.g., for /start, /help)  
    # Add message handlers (e.g., for echoing text)  
    # These will be detailed in the following subsections.  
  
    # Start the bot by polling for updates.  
    # application.run_polling() is the mechanism that continuously fetches new messages  
    # and other updates from Telegram for the bot to process. Without this,  
    # the script would define the bot's behavior but wouldn't actively listen for interactions. [18, 19]  
    logger.info("Starting bot...")  
    application.run_polling(allowed_updates=Update.ALL_TYPES)  
    logger.info("Bot stopped.")  
  
# This standard Python construct ensures that main() is called only when the script  
# is executed directly, not when it's imported as a module into another script.  
# This promotes modularity and is a common practice in Python scripting. [18, 19]  
if __name__ == "__main__":  
    main()  
  

To use the environment variable method for BOT_TOKEN, one would set TELEGRAM_BOT_TOKEN in their system's environment before running the script. For example, on Linux/macOS: export TELEGRAM_BOT_TOKEN="your_actual_token_here". On Windows: set TELEGRAM_BOT_TOKEN="your_actual_token_here" (in Command Prompt) or $env:TELEGRAM_BOT_TOKEN="your_actual_token_here" (in PowerShell). Using environment variables prevents the token from being accidentally committed to version control systems like Git.2

### C. Handling the /start and /help Commands

Bots commonly respond to /start (when a user first interacts with the bot) and /help (to provide usage instructions) commands. Handler functions for these commands are defined as asynchronous functions because operations like sending messages involve network I/O, and python-telegram-bot leverages asyncio for non-blocking operations.18

  

Python

  
  

# Define the start command handler  
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Sends a welcome message when the command /start is issued."""  
    user = update.effective_user  # Gets information about the user who sent the command  
    # update.message.reply_html allows sending messages with HTML formatting.  
    # user.mention_html() creates a clickable mention of the user.  
    await update.message.reply_html(  
        rf"Hi {user.mention_html()}! I am your friendly echo bot.",  
        # ForceReply can be used to prompt the user for a direct reply.  
        reply_markup=ForceReply(selective=True),  
    )  
    logger.info(f"User {user.first_name} ({user.id}) started the bot.")  
  
# Define the help command handler  
async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Sends a help message when the command /help is issued."""  
    # update.message.reply_text sends a plain text message.  
    await update.message.reply_text("I am an echo bot. Just send me any text message, and I will repeat it back to you!")  
    logger.info(f"User {update.effective_user.first_name} requested help.")  
  

(Code adapted from 18)

These functions need to be registered with the application in the main() function. The CommandHandler class is specifically designed to listen for and react to user commands.18 This clear separation of concerns—using CommandHandler for explicit user commands and other handlers for different types of interactions—makes the bot's logic more organized.

  

Python

  
  

# In the main() function, after `application` is created:  
application.add_handler(CommandHandler("start", start))  
application.add_handler(CommandHandler("help", help_command))  
  

(From 18)

### D. Echoing User Messages

To make the bot echo messages, a handler is needed that processes regular text messages that are not commands. The MessageHandler class, combined with appropriate filters, achieves this.

  

Python

  
  

# Define the echo message handler  
async def echo(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Echoes the user's text message."""  
    # update.message.text contains the text of the message sent by the user.  
    # update.message.reply_text sends a reply containing that same text.  
    logger.info(f"Echoing message from {update.effective_user.first_name}: {update.message.text}")  
    await update.message.reply_text(update.message.text)  
  

(From 18)

This echo function is registered using MessageHandler. The filters module provides various criteria to select which messages this handler should process. For an echo bot, it should respond to text messages (filters.TEXT) but not to commands (~filters.COMMAND, where ~ means NOT). The & operator combines these filters (AND logic).18 This careful filtering is important; without ~filters.COMMAND, the echo handler might also try to process commands like /start as regular text to be echoed, leading to redundant or confusing behavior.

  

Python

  
  

# In the main() function, after command handlers:  
# This handler will process messages that are text AND are NOT commands.  
application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, echo))  
  

(From.18 Filter explanation based on 23)

### E. Running Your Bot

Once the bot token is set, and the main() function is populated with the application setup and handlers, the application.run_polling() line is what brings the bot to life.18 This method continuously fetches updates (new messages, commands, etc.) from Telegram's servers and dispatches them to the registered handlers. For development and simpler bots, polling is a straightforward way to receive updates, as it doesn't require setting up a public HTTPS server, which webhooks would necessitate.6

To run the bot:

1. Save the complete script (e.g., echo_bot.py).
    
2. Ensure the TELEGRAM_BOT_TOKEN environment variable is set with the actual bot token.
    
3. Open a terminal, navigate to the directory where the script is saved, and run:  
    Bash  
    python echo_bot.py  
      
    
4. If everything is set up correctly, messages like "Starting bot..." will appear in the console.
    
5. Open Telegram, find the bot by its username, and send it a /start command, a /help command, or any text message to see it respond. The bot will continue running and responding to messages until the script is manually stopped (e.g., by pressing Ctrl+C in the terminal).
    

## IV. Essential Bot Functionalities with Examples

Beyond a simple echo bot, most Telegram bots incorporate more diverse functionalities. This section explores how to send different message types, handle commands that include arguments, and create interactive inline keyboards.

### A. Sending Different Types of Messages

The python-telegram-bot library provides methods for sending various types of content. These methods, such as send_message, send_photo, and send_document, often share common parameters like chat_id, caption, parse_mode, and reply_markup, which provides a consistent interface for developers.27 This consistency simplifies learning; once a developer understands these common parameters for one sending method, that knowledge is largely transferable to others.

#### 1. Text Messages

Sending text messages is a fundamental capability. As seen previously, update.message.reply_text() is a convenient shortcut to reply directly to an incoming message. Alternatively, context.bot.send_message(chat_id=update.effective_chat.id, text="...") can be used for more explicit control or when sending a message not as a direct reply.1

Telegram supports basic text formatting using either HTML or MarkdownV2. This is controlled by the parse_mode parameter.

  

Python

  
  

from telegram.constants import ParseMode # Required for specifying parse modes  
from telegram.ext import CommandHandler, ContextTypes, Update # Assuming Application setup is elsewhere  
  
#... (logger setup)...  
  
async def formatted_message_command(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Sends a message with HTML formatting."""  
    chat_id = update.effective_chat.id  
    html_text = "This is <b>bold</b>, <i>italic</i>, and a <a href='https://www.python.org'>link</a>."  
    await context.bot.send_message(chat_id, text=html_text, parse_mode=ParseMode.HTML)  
    logger.info(f"Sent formatted HTML message to chat {chat_id}")  
  
# In your main() function, you would add:  
# application.add_handler(CommandHandler("format_html", formatted_message_command))  
  

(Example based on 27)

#### 2. Photos

Bots can send photos using the context.bot.send_photo() method.27 Photos can be sent from a local file path or by providing a URL. When sending a local file, it's important to open the file in binary read mode ('rb').27 Parameters like caption allow adding text to the photo, parse_mode can format the caption, and has_spoiler can send the photo with a spoiler animation.27 The library offers flexibility in how files are sourced: local paths, URLs, or an existing file_id (if the file is already on Telegram's servers). Using a file_id is often the most efficient method for Telegram if the file has been previously uploaded, potentially offering performance benefits for bots that frequently resend the same media.33

Example (Sending a photo from a local file):

  

Python

  
  

#... (imports and logger setup)...  
  
async def send_local_photo_command(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Sends a photo from a local file path."""  
    chat_id = update.effective_chat.id  
    # Replace "path/to/your/image.jpg" with the actual path to an image file  
    photo_path = "path/to/your/image.jpg"  
    try:  
        with open(photo_path, "rb") as photo_file:  
            await context.bot.send_photo(  
                chat_id=chat_id,  
                photo=photo_file,  
                caption="Here is a local photo with a <b>bold</b> caption!",  
                parse_mode=ParseMode.HTML,  
                has_spoiler=False # Set to True to send as a spoiler  
            )  
        logger.info(f"Sent local photo {photo_path} to chat {chat_id}")  
    except FileNotFoundError:  
        await update.message.reply_text("Sorry, I couldn't find the photo at the specified path.")  
        logger.warning(f"Photo not found at {photo_path} for chat {chat_id}")  
    except Exception as e:  
        logger.error(f"Error sending photo to chat {chat_id}: {e}")  
        await update.message.reply_text("Sorry, an error occurred while sending the photo.")  
  
# In main():  
# application.add_handler(CommandHandler("sendphoto", send_local_photo_command))  
  

(Adapted from 27)

#### 3. Documents

Similarly, general files (documents) can be sent using context.bot.send_document().28 This is suitable for PDFs, text files, spreadsheets, etc. Like photos, documents can be sent from a local path by opening the file in binary read mode.28 Parameters such as caption and filename (to specify the name the user sees) are available.28

Example (Sending a document from a local file):

  

Python

  
  

#... (imports and logger setup)...  
  
async def send_local_document_command(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Sends a document from a local file path."""  
    chat_id = update.effective_chat.id  
    # Replace "path/to/your/document.pdf" with the actual path to a document file  
    document_path = "path/to/your/document.pdf"  
    try:  
        with open(document_path, "rb") as doc_file:  
            await context.bot.send_document(  
                chat_id=chat_id,  
                document=doc_file,  
                filename="MyImportantReport.pdf", # Custom filename for the user  
                caption="Here is the document you requested."  
            )  
        logger.info(f"Sent local document {document_path} as MyImportantReport.pdf to chat {chat_id}")  
    except FileNotFoundError:  
        await update.message.reply_text("Sorry, I couldn't find the document at the specified path.")  
        logger.warning(f"Document not found at {document_path} for chat {chat_id}")  
    except Exception as e:  
        logger.error(f"Error sending document to chat {chat_id}: {e}")  
        await update.message.reply_text("Sorry, an error occurred while sending the document.")  
  
# In main():  
# application.add_handler(CommandHandler("senddoc", send_local_document_command))  
  

(Adapted from 28)

### B. Handling User Commands with Arguments

Often, commands require additional parameters from the user. For instance, a /weather London command needs "London" as an argument. The CommandHandler automatically parses text following a command (split by spaces) and makes it available as a list of strings in context.args.7 This built-in feature simplifies argument parsing for common use cases, saving developers from writing boilerplate code to split command strings manually.

Example (A command /greet <name>):

  

Python

  
  

#... (imports and logger setup)...  
  
async def greet_user_command(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Greets the user with the name provided in arguments."""  
    # context.args is a list of strings, e.g., if user sends "/greet John Doe"  
    if context.args:  # Check if any arguments were provided  
        user_name = " ".join(context.args) # Join all arguments to form the name  
        await update.message.reply_text(f"Hello, {user_name}!")  
        logger.info(f"Greeted user {user_name} in chat {update.effective_chat.id}")  
    else:  
        await update.message.reply_text("Hello there! Please tell me your name using /greet YourName.")  
        logger.info(f"Greet command received without arguments from chat {update.effective_chat.id}")  
  
# In main():  
# application.add_handler(CommandHandler("greet", greet_user_command))  
  

(Based on 22)

While context.args provides the arguments as strings, the bot's logic is still responsible for validating their type, number (if not using the has_args parameter of CommandHandler), and content for robust behavior. For example, if a command expects a number, the string from context.args must be converted to an integer and validated.8 The CommandHandler also has a has_args parameter that can be set to True (any arguments), False (no arguments), or an integer (specific number of arguments) for basic argument count validation before the callback is even invoked.22

### C. Creating Interactive Inline Keyboards

Inline keyboards provide buttons attached directly to messages sent by the bot, offering a more guided and interactive user experience compared to free-form text commands.37 They are excellent for menus, multiple-choice questions, or triggering specific actions without requiring users to type commands.

#### 1. Defining InlineKeyboardMarkup and InlineKeyboardButton

To create an inline keyboard:

1. Define InlineKeyboardButton objects. Each button has a text (the label displayed to the user) and callback_data (a string that is sent back to the bot when the button is pressed).41 The callback_data is a crucial communication bridge; its design (e.g., simple identifiers or more complex serialized data) is key for routing actions in the bot's logic.
    
2. Arrange these buttons in a list of lists, where each inner list represents a row of buttons.
    
3. Create an InlineKeyboardMarkup object from this list of lists.
    
4. Pass this reply_markup to a message sending method (e.g., reply_text, send_message).
    

Example (Showing a simple inline menu):

  

Python

  
  

from telegram import InlineKeyboardButton, InlineKeyboardMarkup  
#... (other imports and logger setup)...  
  
async def show_inline_menu_command(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Sends a message with an inline keyboard menu."""  
    keyboard =,  
      ,  
       
    reply_markup = InlineKeyboardMarkup(keyboard)  
    await update.message.reply_text('Please choose an option from the menu:', reply_markup=reply_markup)  
    logger.info(f"Sent inline menu to chat {update.effective_chat.id}")  
  
# In main():  
# application.add_handler(CommandHandler("menu", show_inline_menu_command))  
  

(Adapted from 37)

#### 2. Handling CallbackQuery with CallbackQueryHandler

When a user clicks an inline button, Telegram sends a CallbackQuery update to the bot.37 The CallbackQueryHandler is used to process these specific updates.

Key actions within the callback function for a CallbackQueryHandler:

- Answer the query: Call await query.answer() where query = update.callback_query. This is essential to acknowledge the button press and remove the "loading" state on the user's client. Failing to do this results in a poor user experience, as the client will show a persistent loading indicator on the button.42
    
- Access callback_data: The query.data attribute contains the callback_data string associated with the pressed button.
    
- Respond or edit: The bot can then send a new message, edit the original message (e.g., change its text or remove/update the keyboard using await query.edit_message_text(...) or await query.edit_message_reply_markup(...)), or perform any other action based on query.data.37
    

Example (Handling button presses from the menu above):

  

Python

  
  

from telegram.ext import CallbackQueryHandler  
#... (other imports and logger setup)...  
  
async def button_callback_handler(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Parses the CallbackQuery and updates the message text."""  
    query = update.callback_query  # Get the CallbackQuery object from the update  
  
    # It's crucial to answer the callback query, even if no notification to the user is needed.  
    # This stops the loading animation on the button.  
    await query.answer()  
    logger.info(f"Received callback_data: {query.data} from user {query.from_user.first_name}")  
  
    response_text = f"You selected: {query.data}"  
    if query.data == 'info_opt_1':  
        response_text = "You chose Option 1. Here is some information..."  
    elif query.data == 'action_A':  
        response_text = "Action A has been triggered!"  
        # Perform Action A  
    elif query.data == 'action_B':  
        response_text = "Action B executed successfully."  
        # Perform Action B  
    elif query.data == 'help_inline':  
        response_text = "This is inline help. Choose an action or ask for more details."  
  
    # Edit the message text to reflect the choice or action taken.  
    # This replaces the original message text and keyboard (if not specified otherwise).  
    await query.edit_message_text(text=response_text)  
    # To only remove the keyboard, one could use:  
    # await query.edit_message_reply_markup(reply_markup=None)  
  
# In main():  
# application.add_handler(CallbackQueryHandler(button_callback_handler))  
  

(Adapted from 37)

### D. Managing Updates: Beyond Simple Messages

Bots can receive and process various types of updates beyond text messages and commands. The MessageHandler combined with the extensive filters module allows for granular control over which updates trigger specific actions.23 This enables the creation of highly context-aware bots.

#### 1. Handling Different Message Types (e.g., photos, documents)

The MessageHandler can be configured with filters like filters.PHOTO or filters.DOCUMENT to specifically react when users send these types of media.23

Example (Handling Photo Uploads by Users):

  

Python

  
  

#... (imports and logger setup)...  
  
async def handle_user_photo(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Handles photos sent by the user."""  
    # update.message.photo is a list of PhotoSize objects, typically the last one is the largest.  
    photo_file = await update.message.photo[-1].get_file()  
    # The file can be downloaded using: await photo_file.download_to_drive('user_photo.jpg')  
    await update.message.reply_text(  
        f"I received your photo! Its file ID is: {photo_file.file_id}. "  
        f"It has dimensions: {photo_file.width}x{photo_file.height}."  
    )  
    logger.info(f"User {update.effective_user.first_name} sent a photo with file_id {photo_file.file_id}")  
  
# In main():  
# application.add_handler(MessageHandler(filters.PHOTO, handle_user_photo))  
  

(Logic based on general PTB principles and filter knowledge from 45)

Example (Handling Document Uploads by Users):

  

Python

  
  

#... (imports and logger setup)...  
  
async def handle_user_document(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Handles documents sent by the user."""  
    document = update.message.document  
    # doc_file = await document.get_file() # To get file object for download  
    # await doc_file.download_to_drive(document.file_name)  
    await update.message.reply_text(  
        f"Received document: '{document.file_name}'. MIME type: {document.mime_type}. File ID: {document.file_id}"  
    )  
    logger.info(f"User {update.effective_user.first_name} sent document '{document.file_name}' (ID: {document.file_id})")  
  
# In main():  
# application.add_handler(MessageHandler(filters.DOCUMENT, handle_user_document))  
  

(Logic based on 46 and filter knowledge from 23)

#### 2. Tracking New and Left Chat Members (Using ChatMemberHandler)

For bots intended for community management, tracking when users join or leave a group is essential. The ChatMemberHandler is specifically designed for this purpose, processing ChatMemberUpdated objects.37 This handler is key for actions like welcoming new users or logging administrative changes.

The ChatMemberUpdated object, found in update.chat_member (for other users) or update.my_chat_member (for the bot itself), contains old_chat_member and new_chat_member attributes. By comparing their status fields (e.g., ChatMember.MEMBER, ChatMember.LEFT, ChatMember.ADMINISTRATOR), the bot can determine the nature of the change. It's important to distinguish between CHAT_MEMBER updates (concerning other users) and MY_CHAT_MEMBER updates (concerning the bot's own status in a chat), which can be specified when adding the handler.51

Example (Greeting new members in a group - simplified):

  

Python

  
  

from telegram import ChatMember, Update, ParseMode  
from telegram.ext import ChatMemberHandler, ContextTypes # Assuming Application setup is elsewhere  
  
#... (logger setup)...  
  
async def greet_new_chat_members(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Greets new members when they join the chat."""  
    chat_member_update = update.chat_member  
    if not chat_member_update:  
        return  
  
    new_member_info = chat_member_update.new_chat_member  
    old_member_info = chat_member_update.old_chat_member  
    chat = chat_member_update.chat  
    cause_user = chat_member_update.from_user # User who performed the action  
  
    # Check if a user joined (was not a member or was kicked, and is now a member)  
    # This is a simplified check. The chatmemberbot.py example has a more robust  
    # extract_status_change function for complex scenarios. [51]  
    was_member_before = old_member_info and old_member_info.status in  
    is_member_now = new_member_info.status in  
  
    if not was_member_before and is_member_now:  
        # Check if the user joining is not the bot itself  
        if new_member_info.user.id!= context.bot.id:  
            user_name_html = new_member_info.user.mention_html()  
            chat_title_html = html.escape(chat.title or "this chat") # Escape chat title  
             
            greeting_message = f"Welcome to {chat_title_html}, {user_name_html}!"  
            if cause_user.id!= new_member_info.user.id: # If added by someone else  
                adder_name_html = cause_user.mention_html()  
                greeting_message = f"{user_name_html} was added by {adder_name_html}. Welcome to {chat_title_html}!"  
  
            await context.bot.send_message(  
                chat_id=chat.id,  
                text=greeting_message,  
                parse_mode=ParseMode.HTML  
            )  
            logger.info(f"User {new_member_info.user.full_name} joined chat '{chat.title}' (ID: {chat.id})")  
  
# In main():  
# This handles updates about *other* users in chats where the bot is present.  
# application.add_handler(ChatMemberHandler(greet_new_chat_members, ChatMemberHandler.CHAT_MEMBER))  
  
# To track when the bot itself is added to or removed from a chat, use ChatMemberHandler.MY_CHAT_MEMBER:  
# async def track_bot_chats(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
#     # Logic from chatmemberbot.py's track_chats function [51]  
#     pass  
# application.add_handler(ChatMemberHandler(track_bot_chats, ChatMemberHandler.MY_CHAT_MEMBER))  
  

(Logic inspired by chatmemberbot.py 51)

This table summarizes the common handlers discussed:

Table 1: Common python-telegram-bot Handlers

|   |   |   |   |
|---|---|---|---|
|Handler Name|Brief Description|Typical Use Case|Key Associated Filters/Update Types|
|CommandHandler|Handles messages that are bot commands (e.g., /start).|Responding to specific user-initiated commands.|Command string (e.g., "start", "help").|
|MessageHandler|Handles various types of messages (text, media, etc.).|Processing general messages, media, status updates.|filters.TEXT, filters.PHOTO, filters.DOCUMENT, etc.|
|CallbackQueryHandler|Handles updates from inline keyboard button presses.|Processing interactions with inline keyboards.|CallbackQuery object (contains callback_data).|
|ChatMemberHandler|Handles updates about chat member status changes.|Welcoming new users, logging kicks/bans, tracking bot presence.|ChatMemberUpdated object (CHAT_MEMBER or MY_CHAT_MEMBER types).|

## V. Best Practices for Robust Telegram Bots

Developing a functional bot is just the first step. Ensuring it is secure, performs well, and is maintainable requires adherence to certain best practices. Effective bot development involves a layered approach, combining secure coding, understanding API behaviors, and respecting operational limits.

### A. Security Considerations

Security is paramount, especially when bots handle user data or perform sensitive actions. While Telegram provides a secure platform (e.g., HTTPS for API calls 20, general GDPR compliance 55), the developer bears the ultimate responsibility for the bot's secure and compliant operation.56

#### 1. Protecting Your Bot Token

The bot token is the key to controlling the bot; it must be treated with the same secrecy as a password.1

- Never hardcode tokens directly into scripts, especially if the code is stored in version control (like Git) or shared publicly.8
    
- Use environment variables: Store the token in an environment variable (e.g., TELEGRAM_BOT_TOKEN) and access it in the script using os.environ.get('TELEGRAM_BOT_TOKEN').8 This is a widely recommended practice.
    
- Configuration files: Alternatively, store the token in a separate configuration file that is not committed to version control (e.g., add it to .gitignore).10
    
- While Telegram API calls are made over HTTPS, encrypting the token during transit, its security in local storage and within the script's environment is the developer's responsibility.20
    

#### 2. Input Validation and Sanitization

All input received by the bot—whether from commands, messages, callback queries, or web app data—should be treated as potentially untrusted and validated accordingly.33 Failure to do so can lead to security vulnerabilities like injection attacks (if input is used in database queries or system commands without sanitization) or unexpected bot behavior due to malformed input.

- Check data types and formats: Ensure input matches expected types (e.g., integer, string) and formats (e.g., email address, date).39
    
- Use regular expressions: For complex patterns, regular expressions can validate input structure.40
    
- Escape output: When incorporating user input into messages sent by the bot, especially if using HTML or Markdown formatting, escape special characters to prevent injection of malicious markup (e.g., using html.escape() for HTML).39
    
- Prefer whitelisting: Allow only known, expected characters or patterns, rather than trying to blacklist all possible malicious inputs.39
    
- Telegram's API documentation specifically warns that data from Web Apps (WebAppData) can be arbitrary and sent by a "bad client," emphasizing the need for rigorous validation.33
    

Example (Simple argument type and range validation):

  

Python

  
  

#... (imports and logger setup)...  
  
async def set_age_command(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Sets a user's age after validating the input."""  
    if not context.args or len(context.args)!= 1:  
        await update.message.reply_text("Usage: /set_age <your_age (must be a number)>")  
        return  
    try:  
        age_str = context.args  
        age = int(age_str) # Attempt to convert to integer  
        if not (0 < age < 150): # Range validation  
            raise ValueError("Age must be between 1 and 149.")  
        # If validation passes, process the age  
        await update.message.reply_text(f"Your age has been recorded as {age}.")  
        logger.info(f"Age {age} set for user {update.effective_user.first_name}")  
    except ValueError:  
        await update.message.reply_text("Invalid age provided. Please enter a valid number between 1 and 149.")  
        logger.warning(f"Invalid age input from {update.effective_user.first_name}: {context.args}")  
  
# In main():  
# application.add_handler(CommandHandler("set_age", set_age_command))  
  

#### 3. Understanding Telegram's Privacy Mode

By default, bots added to group chats operate in "Privacy Mode".7

- In this mode, the bot will not receive all messages sent in the group. It will only receive:
    

- Commands explicitly meant for it (e.g., /command@botusername or /command if no other bot recently messaged).7
    
- Replies to the bot's own messages.7
    
- Messages sent via the bot (e.g., through inline mode).7
    

- To allow a bot to receive all messages in a group (necessary for features like moderation or keyword responses), privacy mode must be disabled for the bot via @BotFather.58
    
- The getMe API call returns a User object for the bot, which includes a can_read_all_group_messages field indicating if privacy mode is disabled.58
    
- Developers must be mindful of the data they collect, especially if privacy mode is disabled. Telegram as a platform is generally GDPR compliant 55, but bot developers are responsible for ensuring their specific bot's data handling practices comply with all applicable privacy laws and regulations, including providing a privacy policy if necessary.56
    

#### 4. Rate Limiting: Avoiding Flood Limits

Telegram imposes rate limits on API requests to prevent abuse and ensure fair usage for all bots.59

- Exceeding these limits can result in RetryAfter errors, where Telegram tells the bot to wait for a certain period, or even temporary blocks.
    
- General guidelines suggest limits around 30 messages per second globally for the bot, and around 20 messages per minute within a single group or channel.60 These are not hard-coded and can vary.
    
- The python-telegram-bot library offers telegram.ext.AIORateLimiter (installable via pip install "python-telegram-bot[rate-limiter]") as a built-in mechanism to proactively throttle outgoing requests and help stay within these limits.13 Using such a proactive limiter is generally preferable to simply reacting to RetryAfter exceptions, as it leads to smoother bot operation and better user experience.
    
- For bots sending a very high volume of messages (e.g., broadcasts), developers might need to implement more sophisticated custom rate-limiting strategies or carefully manage send rates, potentially using asyncio.sleep() between batches of messages. Telegram also offers a paid option (allow_paid_broadcast) for significantly higher throughput for broadcast messages, at a cost per message.60
    

This table summarizes key security best practices:

Table 2: Key Security Best Practices Summary

|   |   |   |
|---|---|---|
|Security Practice|Why it's Important|Key Recommendation(s)|
|Bot Token Protection|Prevents unauthorized control of the bot.|Never hardcode tokens. Use environment variables or secure configuration files. Keep tokens secret.|
|Input Validation & Sanitization|Protects against malicious input, injection attacks, and unexpected behavior.|Treat all user input as untrusted. Validate data types, formats, and ranges. Escape output appropriately (e.g., html.escape()). Prefer whitelisting.|
|Privacy Mode Awareness|Ensures compliance and appropriate data handling based on bot's access to messages.|Understand default privacy mode. Disable only if necessary. Comply with privacy laws (e.g., GDPR) and provide a privacy policy if needed.|
|Rate Limiting|Prevents API flood errors and temporary blocks from Telegram.|Be aware of Telegram's rate limits. Use built-in limiters like AIORateLimiter or implement custom throttling for high-volume bots.|

### B. Performance and Maintenance

Beyond security, ensuring the bot is responsive and easy to maintain is critical for long-term success.

#### 1. Leveraging asyncio for Efficiency

The python-telegram-bot library is built on Python's asyncio framework, making it inherently asynchronous.4 This is a significant advantage for I/O-bound applications like Telegram bots, which spend much of their time waiting for network operations (e.g., receiving messages from Telegram, sending replies).

- Non-Blocking Operations: Asynchronous programming allows the bot to handle multiple user requests or API calls concurrently without one operation blocking others. If a part of the code is waiting for a network response, the bot can switch to processing other tasks.
    
- async def and await: All handler functions should be defined as asynchronous functions using async def, and any calls to API methods (which are I/O-bound) should be preceded by await (as demonstrated in all previous code examples based on 18).
    
- Avoid Synchronous Blocking Code: A common pitfall for those new to asyncio is including long-running synchronous (blocking) code directly within an async handler. Operations like lengthy computations or traditional time.sleep() will freeze the entire asyncio event loop, making the bot unresponsive to all users. If such operations are necessary, they should be run in a separate thread or process pool, or asyncio-compatible alternatives should be used (e.g., await asyncio.sleep()).
    

#### 2. Effective Logging for Debugging and Monitoring

Comprehensive logging is invaluable for understanding the bot's behavior, debugging issues, and monitoring its health in production.3

- Basic Setup: The Python logging module can be configured easily for basic needs. The logging.basicConfig() method, often seen in python-telegram-bot examples, is a good starting point for console or file logging.3  
    Python  
    import logging  
      
    logging.basicConfig(  
        level=logging.INFO,  # Set to logging.DEBUG for more detailed output during development  
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',  
        # filename='bot.log',  # Uncomment to log to a file  
        # filemode='a'        # 'a' for append, 'w' for overwrite  
    )  
    logger = logging.getLogger(__name__)  
      
    # Example usage in bot code:  
    # logger.info("Bot started successfully.")  
    # logger.warning("User provided invalid input for command X.")  
    # logger.error("Failed to connect to external API.")  
      
    
- Log Levels: Utilize different log levels (DEBUG, INFO, WARNING, ERROR, CRITICAL) to categorize messages by severity. This allows for filtering logs based on importance (e.g., showing only ERROR and CRITICAL messages in a production monitoring setup).67
    
- What to Log: Log key events (bot start/stop, command received), important state changes, errors encountered, and any information useful for debugging (e.g., user IDs, chat IDs, relevant data from updates, but be mindful of privacy).
    
- Error Handler Logging: The global error handler (discussed next) should always log detailed information about exceptions, including tracebacks.69
    
- Advanced Logging: For more complex applications, consider using multiple log handlers (e.g., logging.FileHandler for persistent logs, logging.StreamHandler for console output), log rotation (logging.handlers.RotatingFileHandler), or even specialized libraries like telegram-bot-logger to send critical log messages to a designated Telegram chat for real-time alerts (though this is an advanced setup).67
    

#### 3. Basic Error Handling

Robust error handling prevents the bot from crashing due to unexpected situations and provides a better user experience.72

- try-except Blocks: Use try-except blocks around operations that are prone to failure, such as file I/O (e.g., FileNotFoundError), network requests to external services (e.g., requests.exceptions.RequestException), or data type conversions (e.g., ValueError when converting a string to an integer).72
    
- Global Error Handler: The python-telegram-bot library allows registering a global error handler using application.add_error_handler(). This function is called whenever an unhandled exception occurs within any of the bot's handlers. This is a much cleaner approach than wrapping every single handler function in its own try-except block, as it centralizes error reporting and ensures consistent handling of unexpected errors.37 A good global error handler will log detailed information about the error (including the update that caused it and the full traceback) and can optionally notify developers (e.g., by sending a message to a private Telegram chat).
    

Example of a custom global error handler (inspired by errorhandlerbot.py):

  

Python

  
  

import html  
import json  
import traceback  
from telegram import Update, ParseMode # Assuming Update and ParseMode are imported  
from telegram.ext import ContextTypes # Assuming ContextTypes is imported  
  
#... (logger setup)...  
  
# Replace with your actual chat ID or a developer group chat ID  
DEVELOPER_CHAT_ID = 123456789  
  
async def global_error_handler(update: object, context: ContextTypes.DEFAULT_TYPE) -> None:  
    """Logs the error and sends a Telegram message to the developer."""  
    # Log the error with traceback before doing anything else  
    logger.error(f"Exception while handling an update: {context.error}", exc_info=context.error)  
  
    # Format the traceback  
    tb_list = traceback.format_exception(None, context.error, context.error.__traceback__)  
    tb_string = "".join(tb_list)  
  
    # Gather information about the update and context  
    update_str = update.to_dict() if isinstance(update, Update) else str(update)  
     
    message_parts =  
     
    full_message = "".join(message_parts)  
  
    # Telegram messages have a character limit (4096 characters)  
    # Truncate the message if it's too long  
    if len(full_message) > 4096:  
        # Keep essential parts and truncate the traceback if necessary  
        essential_info = "".join(message_parts[:5]) # Update, Chat/User Data, Error  
        remaining_space = 4096 - len(essential_info) - len("\n... (traceback truncated)...")  
        truncated_traceback = html.escape(tb_string)[:remaining_space]  
        full_message = essential_info + "<pre>" + truncated_traceback + "</pre>\n... (traceback truncated)..."  
  
    try:  
        await context.bot.send_message(  
            chat_id=DEVELOPER_CHAT_ID,  
            text=full_message,  
            parse_mode=ParseMode.HTML  
        )  
    except Exception as e:  
        logger.error(f"Failed to send error notification to developer: {e}")  
  
# In your main() function:  
# application.add_error_handler(global_error_handler)  
  

(Adapted from errorhandlerbot.py logic 69)

This centralized error handling makes the bot more robust and easier to monitor for unexpected issues.

## VI. Conclusion and Further Exploration

This guide has provided a practical foundation for creating Telegram bot scripts using Python and the python-telegram-bot library. By following these steps, developers can set up their environment, build a basic functioning bot, implement common features like sending various message types and handling interactive elements, and understand essential best practices for security, performance, and maintenance.

### A. Recap of Key Learnings

The journey covered:

- Understanding the basics of Telegram bots and the Telegram Bot API.
    
- Setting up a Python development environment with virtual environments and installing the python-telegram-bot library.
    
- Obtaining a bot token from BotFather and structuring a basic bot script.
    
- Implementing core handlers: CommandHandler for specific commands like /start and /help, and MessageHandler for general text messages, including the echo functionality.
    
- Expanding bot capabilities to send formatted text, photos, and documents.
    
- Handling commands with arguments using context.args.
    
- Creating interactive inline keyboards with InlineKeyboardMarkup and InlineKeyboardButton, and processing responses using CallbackQueryHandler.
    
- Managing diverse updates such as user-uploaded media and tracking chat member changes with ChatMemberHandler.
    
- Adhering to crucial best practices regarding bot token security, input validation, understanding Telegram's privacy mode, rate limiting, leveraging asyncio, effective logging, and robust error handling.
    

### B. Ideas for Expanding Your Bot's Capabilities

The world of Telegram bot development is vast. With the fundamentals covered, consider exploring more advanced features and project ideas:

- Multi-Step Conversations: Use ConversationHandler to guide users through complex, multi-step interactions, like filling out a form or playing a game.37 The python-telegram-bot examples include conversationbot.py which is an excellent starting point.
    
- Integration with External APIs: Enhance the bot by fetching data from external services. Examples include:
    

- Weather bots that get forecasts from weather APIs.
    
- News bots that pull headlines from news APIs.
    
- Translation bots using translation APIs.5
    

- Data Persistence: Store user data, preferences, or chat states across bot restarts. This can be done using simple files (JSON, CSV), SQLite databases, or more complex database systems. The persistentconversationbot.py example demonstrates basic persistence.37
    
- Scheduled Tasks: Use the JobQueue feature (requires python-telegram-bot[job-queue]) to schedule tasks, such as sending daily reminders, news updates, or performing regular maintenance.5 The timerbot.py example showcases this.
    
- Real-World Applications:
    

- Customer Service: Automate responses to frequently asked questions.
    
- Notification Systems: Send alerts for system events, new orders, or updates.1
    
- Community Management: Automate welcoming new members, moderating messages, or polling users.10
    
- Utility Bots: Create bots for currency conversion 5, to-do lists, or file management.
    
- Educational Bots: Deliver quizzes or learning materials.
    
- Content Delivery: Share articles, images, or videos based on user preferences.
    

The examples provided by the python-telegram-bot library often show a clear progression from simple concepts like echobot.py to more intricate ones like conversationbot.py or inlinekeyboard2.py, offering a natural learning path for developers to gradually build their skills.37

### C. Useful Resources

Continuous learning and access to good documentation are vital in software development. For further exploration and troubleshooting:

- Official python-telegram-bot Documentation: The primary source for library-specific information, tutorials, and API references (3 - typically found at https://docs.python-telegram-bot.org/).
    
- python-telegram-bot Examples: A collection of practical examples showcasing various features (37 - usually available on GitHub within the library's repository or linked from the official documentation).
    
- Telegram Bot API Documentation: The official documentation from Telegram for the Bot API itself, detailing all methods, types, and limits (1 - found at https://core.telegram.org/bots/api).
    
- python-telegram-bot Community: The library often has a Telegram group or other community channels for support and discussion, which can be invaluable for getting help and sharing knowledge.3
    

By leveraging these resources and the knowledge gained from this guide, developers are well-equipped to build innovative and practical Telegram bots with Python. The journey of bot development is one of continuous learning, and the active community and comprehensive documentation are there to support it.

#### منابع مورداستناد

1. Telegram Bot API: An Introduction | Built In, زمان دسترسی: ژوئن 4, 2025، [https://builtin.com/software-engineering-perspectives/telegram-api](https://builtin.com/software-engineering-perspectives/telegram-api)
    
2. A Beginner Guide to Telegram Bot API - Apidog, زمان دسترسی: ژوئن 4, 2025، [https://apidog.com/blog/beginners-guide-to-telegram-bot-api/](https://apidog.com/blog/beginners-guide-to-telegram-bot-api/)
    
3. python-telegram-bot 13.14 - PyPI, زمان دسترسی: ژوئن 4, 2025، [https://pypi.org/project/python-telegram-bot/13.14/](https://pypi.org/project/python-telegram-bot/13.14/)
    
4. Transmitting Media Using Python-Telegram-Bot (Asynchronous) - Latenode community, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/transmitting-media-using-python-telegram-bot-asynchronous/3417](https://community.latenode.com/t/transmitting-media-using-python-telegram-bot-asynchronous/3417)
    
5. How to build a fully-fledged telegram bot in Python - DEV Community, زمان دسترسی: ژوئن 4, 2025، [https://dev.to/emiloju/how-to-build-a-fully-fledged-telegram-bot-in-python-2al0](https://dev.to/emiloju/how-to-build-a-fully-fledged-telegram-bot-in-python-2al0)
    
6. How to make a bot: a guide to your first Python chat bot for Telegram - MindK.com, زمان دسترسی: ژوئن 4, 2025، [https://www.mindk.com/blog/how-to-develop-a-chat-bot/](https://www.mindk.com/blog/how-to-develop-a-chat-bot/)
    
7. Telegram Bot Features, زمان دسترسی: ژوئن 4, 2025، [https://core.telegram.org/bots/features](https://core.telegram.org/bots/features)
    
8. Creating a Python Telegram bot in PyCharm - Helen Scott, زمان دسترسی: ژوئن 4, 2025، [https://www.helenjoscott.com/2023/03/08/creating-a-python-telegram-bot-in-pycharm/](https://www.helenjoscott.com/2023/03/08/creating-a-python-telegram-bot-in-pycharm/)
    
9. Telegram Bot & Group Chats messages through python - GitHub, زمان دسترسی: ژوئن 4, 2025، [https://github.com/Imod7/telegram-bot](https://github.com/Imod7/telegram-bot)
    
10. Creating a custom Telegram bot with Python: Beginner's help needed - Latenode community, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/creating-a-custom-telegram-bot-with-python-beginners-help-needed/19750](https://community.latenode.com/t/creating-a-custom-telegram-bot-with-python-beginners-help-needed/19750)
    
11. ChatterBot: Build a Chatbot With Python, زمان دسترسی: ژوئن 4, 2025، [https://realpython.com/build-a-chatbot-python-chatterbot/](https://realpython.com/build-a-chatbot-python-chatterbot/)
    
12. 12. Virtual Environments and Packages — Python 3.13.4 ..., زمان دسترسی: ژوئن 4, 2025، [https://docs.python.org/3/tutorial/venv.html](https://docs.python.org/3/tutorial/venv.html)
    
13. python-telegram-bot · PyPI, زمان دسترسی: ژوئن 4, 2025، [https://pypi.org/project/python-telegram-bot/](https://pypi.org/project/python-telegram-bot/)
    
14. How do I get the Telegram Token or Bot ID? | Superchat Help Center, زمان دسترسی: ژوئن 4, 2025، [https://help.superchat.com/en/articles/14901-how-do-i-get-the-telegram-token-or-bot-id](https://help.superchat.com/en/articles/14901-how-do-i-get-the-telegram-token-or-bot-id)
    
15. How To get Telegram Bot Token - Expertflow CX, زمان دسترسی: ژوئن 4, 2025، [https://docs.expertflow.com/cx/4.4/how-to-get-telegram-bot-token](https://docs.expertflow.com/cx/4.4/how-to-get-telegram-bot-token)
    
16. How to send messages to telegram using Python - Stack Overflow, زمان دسترسی: ژوئن 4, 2025، [https://stackoverflow.com/questions/75116947/how-to-send-messages-to-telegram-using-python](https://stackoverflow.com/questions/75116947/how-to-send-messages-to-telegram-using-python)
    
17. Bots: An introduction for developers - Telegram APIs, زمان دسترسی: ژوئن 4, 2025، [https://core.telegram.org/bots#botfather](https://core.telegram.org/bots#botfather)
    
18. echobot.py - python-telegram-bot v20.3, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/v20.3/examples.echobot.html](https://docs.python-telegram-bot.org/en/v20.3/examples.echobot.html)
    
19. basic telegram bot example - python - Stack Overflow, زمان دسترسی: ژوئن 4, 2025، [https://stackoverflow.com/questions/56923241/basic-telegram-bot-example](https://stackoverflow.com/questions/56923241/basic-telegram-bot-example)
    
20. How safe is a Telegram bot token when used in API calls? - Latenode community, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/how-safe-is-a-telegram-bot-token-when-used-in-api-calls/7486](https://community.latenode.com/t/how-safe-is-a-telegram-bot-token-when-used-in-api-calls/7486)
    
21. Creating a Python-based Telegram bot: Where to begin? - Latenode community, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/creating-a-python-based-telegram-bot-where-to-begin/8557](https://community.latenode.com/t/creating-a-python-based-telegram-bot-where-to-begin/8557)
    
22. CommandHandler - python-telegram-bot v21.7, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/stable/telegram.ext.commandhandler.html](https://docs.python-telegram-bot.org/en/stable/telegram.ext.commandhandler.html)
    
23. telegram.ext.filters Module — python-telegram-bot 13.10 documentation, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/v13.10/telegram.ext.filters.html](https://docs.python-telegram-bot.org/en/v13.10/telegram.ext.filters.html)
    
24. python-telegram-bot/telegram/ext/filters.py at master - GitHub, زمان دسترسی: ژوئن 4, 2025، [https://github.com/python-telegram-bot/python-telegram-bot/blob/master/telegram/ext/filters.py](https://github.com/python-telegram-bot/python-telegram-bot/blob/master/telegram/ext/filters.py)
    
25. AttributeError: module 'telegram.ext.filters' has no attribute 'text' - Stack Overflow, زمان دسترسی: ژوئن 4, 2025، [https://stackoverflow.com/questions/75096233/attributeerror-module-telegram-ext-filters-has-no-attribute-text](https://stackoverflow.com/questions/75096233/attributeerror-module-telegram-ext-filters-has-no-attribute-text)
    
26. MessageHandler - python-telegram-bot v22.0, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/v22.0/telegram.ext.messagehandler.html](https://docs.python-telegram-bot.org/en/v22.0/telegram.ext.messagehandler.html)
    
27. Bot - python-telegram-bot v22.0, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/stable/telegram.bot.html#telegram.Bot.send_photo](https://docs.python-telegram-bot.org/en/stable/telegram.bot.html#telegram.Bot.send_photo)
    
28. Bot - python-telegram-bot v22.0, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/stable/telegram.bot.html#telegram.Bot.send_document](https://docs.python-telegram-bot.org/en/stable/telegram.bot.html#telegram.Bot.send_document)
    
29. Bot - python-telegram-bot v22.0, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/stable/telegram.bot.html](https://docs.python-telegram-bot.org/en/stable/telegram.bot.html)
    
30. How to send messages using python-telegram-bot library? - Latenode community, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/how-to-send-messages-using-python-telegram-bot-library/9260](https://community.latenode.com/t/how-to-send-messages-using-python-telegram-bot-library/9260)
    
31. How to Send Images with Python Telegram Bot - Latenode community, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/how-to-send-images-with-python-telegram-bot/11450](https://community.latenode.com/t/how-to-send-images-with-python-telegram-bot/11450)
    
32. How to send images with Python Telegram Bot? - Latenode community, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/how-to-send-images-with-python-telegram-bot/10458](https://community.latenode.com/t/how-to-send-images-with-python-telegram-bot/10458)
    
33. Telegram Bot API - Telegram APIs, زمان دسترسی: ژوئن 4, 2025، [https://core.telegram.org/bots/api#security-considerations](https://core.telegram.org/bots/api#security-considerations)
    
34. Sending files to users via Python Telegram bot: Help needed - Latenode community, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/sending-files-to-users-via-python-telegram-bot-help-needed/12252](https://community.latenode.com/t/sending-files-to-users-via-python-telegram-bot-help-needed/12252)
    
35. What's the correct way to deliver a PDF file to a user via a Python Telegram bot?, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/whats-the-correct-way-to-deliver-a-pdf-file-to-a-user-via-a-python-telegram-bot/19747](https://community.latenode.com/t/whats-the-correct-way-to-deliver-a-pdf-file-to-a-user-via-a-python-telegram-bot/19747)
    
36. CommandHandler - python-telegram-bot v21.5, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/v21.5/telegram.ext.commandhandler.html](https://docs.python-telegram-bot.org/en/v21.5/telegram.ext.commandhandler.html)
    
37. Examples - python-telegram-bot v22.1, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/stable/examples.html](https://docs.python-telegram-bot.org/en/stable/examples.html)
    
38. How to handle user input after clicking an InlineKeyboardButton in Python-Telegram-Bot?, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/how-to-handle-user-input-after-clicking-an-inlinekeyboardbutton-in-python-telegram-bot/9615](https://community.latenode.com/t/how-to-handle-user-input-after-clicking-an-inlinekeyboardbutton-in-python-telegram-bot/9615)
    
39. How to implement input validation and sanitization in Cybersecurity - LabEx, زمان دسترسی: ژوئن 4, 2025، [https://labex.io/tutorials/wireshark-how-to-implement-input-validation-and-sanitization-in-cybersecurity-417883](https://labex.io/tutorials/wireshark-how-to-implement-input-validation-and-sanitization-in-cybersecurity-417883)
    
40. How to sanitize user input in Python - Educative.io, زمان دسترسی: ژوئن 4, 2025، [https://www.educative.io/answers/how-to-sanitize-user-input-in-python](https://www.educative.io/answers/how-to-sanitize-user-input-in-python)
    
41. Converting regular keyboard to inline keyboard in Python Telegram bot, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/converting-regular-keyboard-to-inline-keyboard-in-python-telegram-bot/9943](https://community.latenode.com/t/converting-regular-keyboard-to-inline-keyboard-in-python-telegram-bot/9943)
    
42. python-telegram-bot/examples/inlinekeyboard2.py at master - GitHub, زمان دسترسی: ژوئن 4, 2025، [https://github.com/python-telegram-bot/python-telegram-bot/blob/master/examples/inlinekeyboard2.py](https://github.com/python-telegram-bot/python-telegram-bot/blob/master/examples/inlinekeyboard2.py)
    
43. Handling inline keyboard button presses in a Python Telegram bot - Latenode community, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/handling-inline-keyboard-button-presses-in-a-python-telegram-bot/13875](https://community.latenode.com/t/handling-inline-keyboard-button-presses-in-a-python-telegram-bot/13875)
    
44. CallbackQuery - python-telegram-bot v21.7, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/v21.7/telegram.callbackquery.html](https://docs.python-telegram-bot.org/en/v21.7/telegram.callbackquery.html)
    
45. filters Module - python-telegram-bot v21.7, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/v21.7/telegram.ext.filters.html](https://docs.python-telegram-bot.org/en/v21.7/telegram.ext.filters.html)
    
46. filters Module - python-telegram-bot v21.7, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/stable/telegram.ext.filters.html](https://docs.python-telegram-bot.org/en/stable/telegram.ext.filters.html)
    
47. python-telegram-bot/examples/conversationbot.py at master - GitHub, زمان دسترسی: ژوئن 4, 2025، [https://github.com/python-telegram-bot/python-telegram-bot/blob/master/examples/conversationbot.py](https://github.com/python-telegram-bot/python-telegram-bot/blob/master/examples/conversationbot.py)
    
48. A custom Telegram Bot to share your precious photos (and allow users to filter by theme!), زمان دسترسی: ژوئن 4, 2025، [https://towardsdatascience.com/a-custom-telegram-bot-to-share-your-precious-photos-and-allow-users-to-filter-by-theme-6b056319a0ae/](https://towardsdatascience.com/a-custom-telegram-bot-to-share-your-precious-photos-and-allow-users-to-filter-by-theme-6b056319a0ae/)
    
49. MessageHandler - python-telegram-bot v22.0, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/stable/telegram.ext.messagehandler.html](https://docs.python-telegram-bot.org/en/stable/telegram.ext.messagehandler.html)
    
50. [QUESTION]new_chat_members can't be obtained in a group of 48000 people · Issue #3560 · python-telegram-bot/python-telegram-bot - GitHub, زمان دسترسی: ژوئن 4, 2025، [https://github.com/python-telegram-bot/python-telegram-bot/issues/3560](https://github.com/python-telegram-bot/python-telegram-bot/issues/3560)
    
51. chatmemberbot.py - python-telegram-bot v21.9, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/v21.9/examples.chatmemberbot.html](https://docs.python-telegram-bot.org/en/v21.9/examples.chatmemberbot.html)
    
52. python-telegram-bot/telegram/_chatmemberupdated.py at master - GitHub, زمان دسترسی: ژوئن 4, 2025، [https://github.com/python-telegram-bot/python-telegram-bot/blob/master/telegram/_chatmemberupdated.py](https://github.com/python-telegram-bot/python-telegram-bot/blob/master/telegram/_chatmemberupdated.py)
    
53. telegram.ext package - python-telegram-bot v21.9, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/stable/telegram.ext.html](https://docs.python-telegram-bot.org/en/stable/telegram.ext.html)
    
54. ChatMemberHandler - python-telegram-bot v21.8, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/stable/telegram.ext.chatmemberhandler.html](https://docs.python-telegram-bot.org/en/stable/telegram.ext.chatmemberhandler.html)
    
55. Telegram Compliance - A Regulatory Overview - Membit, زمان دسترسی: ژوئن 4, 2025، [https://membit.io/blog/telegram-compliance-a-regulatory-overview/](https://membit.io/blog/telegram-compliance-a-regulatory-overview/)
    
56. Telegram Bot Platform Developer Terms of Service, زمان دسترسی: ژوئن 4, 2025، [https://telegram.org/tos/bot-developers](https://telegram.org/tos/bot-developers)
    
57. Looking for an official documentation regarding security best practices and how to write secure code : r/Python - Reddit, زمان دسترسی: ژوئن 4, 2025، [https://www.reddit.com/r/Python/comments/tiq3ts/looking_for_an_official_documentation_regarding/](https://www.reddit.com/r/Python/comments/tiq3ts/looking_for_an_official_documentation_regarding/)
    
58. Telegram Bot API - Telegram APIs, زمان دسترسی: ژوئن 4, 2025، [https://core.telegram.org/bots/api#privacy-mode](https://core.telegram.org/bots/api#privacy-mode)
    
59. Python Telegram bot for remote macOS management: Seeking advice, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/python-telegram-bot-for-remote-macos-management-seeking-advice/15452](https://community.latenode.com/t/python-telegram-bot-for-remote-macos-management-seeking-advice/15452)
    
60. AIORateLimiter - python-telegram-bot v22.0, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/v22.0/telegram.ext.aioratelimiter.html](https://docs.python-telegram-bot.org/en/v22.0/telegram.ext.aioratelimiter.html)
    
61. BaseRateLimiter - python-telegram-bot v21.6, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/v21.6/telegram.ext.baseratelimiter.html](https://docs.python-telegram-bot.org/en/v21.6/telegram.ext.baseratelimiter.html)
    
62. github.com, زمان دسترسی: ژوئن 4, 2025، [https://github.com/python-telegram-bot/python-telegram-bot/wiki/Avoiding-flood-limits](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Avoiding-flood-limits)
    
63. python-telegram-bot v22.1, زمان دسترسی: ژوئن 4, 2025، [https://python-telegram-bot.readthedocs.io/en/stable/](https://python-telegram-bot.readthedocs.io/en/stable/)
    
64. Sharing images with users via python-telegram-bot (asynchronous) - Latenode community, زمان دسترسی: ژوئن 4, 2025، [https://community.latenode.com/t/sharing-images-with-users-via-python-telegram-bot-asynchronous/11117](https://community.latenode.com/t/sharing-images-with-users-via-python-telegram-bot-asynchronous/11117)
    
65. زمان دسترسی: ژانویهٔ 1, 1970، [https://github.com/python-telegram-bot/python-telegram-bot/wiki/Extensions---Your-first-Bot](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Extensions---Your-first-Bot)
    
66. زمان دسترسی: ژانویهٔ 1, 1970، [https://github.com/python-telegram-bot/python-telegram-bot/wiki/Concurrency](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Concurrency)
    
67. Advanced Python Logging: Mastering Configuration & Best Practices for Production, زمان دسترسی: ژوئن 4, 2025، [https://uptrace.dev/blog/python-logging](https://uptrace.dev/blog/python-logging)
    
68. Python Logging Tutorial: How-To, Basic Examples & Best Practices - Sematext, زمان دسترسی: ژوئن 4, 2025، [https://sematext.com/blog/python-logging/](https://sematext.com/blog/python-logging/)
    
69. errorhandlerbot.py - python-telegram-bot v20.0a5, زمان دسترسی: ژوئن 4, 2025، [https://docs.python-telegram-bot.org/en/v20.0a5/examples.errorhandlerbot.html](https://docs.python-telegram-bot.org/en/v20.0a5/examples.errorhandlerbot.html)
    
70. python-telegram-bot/examples/errorhandlerbot.py at master - GitHub, زمان دسترسی: ژوئن 4, 2025، [https://github.com/python-telegram-bot/python-telegram-bot/blob/master/examples/errorhandlerbot.py](https://github.com/python-telegram-bot/python-telegram-bot/blob/master/examples/errorhandlerbot.py)
    
71. telegram-bot-logger - PyPI, زمان دسترسی: ژوئن 4, 2025، [https://pypi.org/project/telegram-bot-logger/](https://pypi.org/project/telegram-bot-logger/)
    
72. Error Handling Strategies and Best Practices in Python - llego.dev, زمان دسترسی: ژوئن 4, 2025، [https://llego.dev/posts/error-handling-strategies-best-practices-python/](https://llego.dev/posts/error-handling-strategies-best-practices-python/)
    
73. 8. Errors and Exceptions — Python 3.13.4 documentation, زمان دسترسی: ژوئن 4, 2025، [https://docs.python.org/3/tutorial/errors.html](https://docs.python.org/3/tutorial/errors.html)
    
74. Error handling pattern - Python discussion forum, زمان دسترسی: ژوئن 4, 2025، [https://discuss.python.org/t/error-handling-pattern/55766](https://discuss.python.org/t/error-handling-pattern/55766)
    
75. github.com, زمان دسترسی: ژوئن 4, 2025، [https://github.com/python-telegram-bot/python-telegram-bot/wiki/Error-Handling](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Error-Handling)
    
76. Best Python Projects for 2025 – Work on Real-time Projects to Head Start Your Career, زمان دسترسی: ژوئن 4, 2025، [https://data-flair.training/blogs/python-project-ideas/](https://data-flair.training/blogs/python-project-ideas/)
    
77. Python Telegram Bot Development Made Easy - Sirvelia, زمان دسترسی: ژوئن 4, 2025، [https://sirvelia.com/en/telegram-bot-python/](https://sirvelia.com/en/telegram-bot-python/)
    
78. Telegram Chatbot: Use Cases and Examples - BotPenguin, زمان دسترسی: ژوئن 4, 2025، [https://botpenguin.com/glossary/telegram-chatbot](https://botpenguin.com/glossary/telegram-chatbot)
    
79. Telegram bot with python-telegram-bot v20 via serverless function - Stack Overflow, زمان دسترسی: ژوئن 4, 2025، [https://stackoverflow.com/questions/75985888/telegram-bot-with-python-telegram-bot-v20-via-serverless-function](https://stackoverflow.com/questions/75985888/telegram-bot-with-python-telegram-bot-v20-via-serverless-function)
    

**