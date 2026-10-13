# ✈️ AI Travel Assistant with Ticket Prices, Travel Times, Image & Voice

A Python travel assistant that answers chat messages and uses function calling to retrieve **ticket prices** and **travel times** from a local SQLite database.

The assistant generates spoken replies through OpenAI text-to-speech and can create a destination image in a vibrant pop-art style. A Gradio interface displays the conversation, audio, and image.

Built with Python, the OpenAI SDK, SQLite, pandas, Pillow, and Gradio.

---

## 🚀 Features

### 🗄️ SQLite Ticket Database
- Creates `prices.db` and a `prices` table if they do not already exist.
- Stores city, country, ticket price, and travel time in hours.
- Uses the city as the primary key.
- Inserts or updates records through `set_ticket_price(city, country, price, eta)`.
- Converts city and country names to lowercase before storing them.
- Converts `HH:MM:SS` strings or numeric values into decimal hours using `to_hours()`.
- Displays the stored records using pandas and notebook `display()`.

### 🔍 Price & Travel-Time Tools
- Retrieves ticket prices through `get_ticket_price()` and reports values in rupees.
- Retrieves travel times through `get_ticket_eta()` and reports values in hours.
- Exposes both functions to the chat model with a required `destination_city` argument.
- Returns a message when the requested city has no matching data.

### 💬 Tool-Calling Conversation
- Uses the configured chat model `gpt-5.4-mini`.
- Includes `system_message` and the conversation history in each chat request.
- Processes price and travel-time tool calls and sends their results back to the model.
- Repeats this process while the model requests tools, then displays the final reply.

### 🔊 Spoken Replies
- Converts each final assistant reply into audio.
- Uses `gpt-4o-mini-tts` with the `onyx` voice.
- Returns audio bytes to a Gradio audio component configured for autoplay.

### 🎨 Destination Images
- Uses `gpt-image-1-mini` to generate one `1024×1024` image.
- Requests tourist spots and features unique to the destination in a vibrant pop-art style.
- Decodes the Base64 image response into a Pillow image.
- Generates an image when the final tool-call batch includes a ticket-price lookup, using its last price-lookup city.

### 🖥️ Gradio Interface
- Displays a chat panel and a non-interactive image panel, both with height `500`.
- Includes an audio player and a message textbox.
- Adds submitted messages to the conversation before calling the assistant.
- Launches with browser opening, sharing, and username/password authentication enabled.

---

## 🧩 Tech Stack

| Component | Tool / configuration |
|---|---|
| Programming language | Python |
| Chat client | OpenAI SDK; configured model: `gpt-5.4-mini` |
| Image generation | OpenAI SDK; model: `gpt-image-1-mini` |
| Text-to-speech | OpenAI SDK; model: `gpt-4o-mini-tts`; voice: `onyx` |
| Database | Python `sqlite3`; local file: `prices.db` |
| Data inspection | pandas; `pd.read_sql_query()` and notebook `display()` |
| Image decoding | `base64`, `io.BytesIO`, and Pillow |
| Tool arguments | Python `json` |
| API key configuration | `python-dotenv`; `OPENAI_API_KEY` |
| User interface | Gradio Blocks, Chatbot, Image, Audio, and Textbox |

The model names above reproduce the supplied code.

---

## 🛠️ Installation

### 1. Prepare the project files

Place the supplied code in a Python notebook. The code uses these files:

| File | Purpose |
|---|---|
| `.env` | Supplies `OPENAI_API_KEY` through `load_dotenv()` |
| `prices.db` | SQLite database created automatically when the initialization code runs |

The supplied snippet references `system_message` but does not define it. Define that variable before running `chat()`; no system-prompt text is supplied in the code.

### 2. Set up Python dependencies

Install the third-party packages imported by the code:

```bash
python -m pip install openai python-dotenv gradio pandas pillow
```

The code also uses Python's standard-library modules `os`, `json`, `sqlite3`, `base64`, and `io`.

### 3. Configure your API key

Create a `.env` file containing:

```dotenv
OPENAI_API_KEY=your_api_key_here
```

The code loads environment variables using `load_dotenv(override=True)` and prints whether the key is present. If present, it prints the first eight characters.

### 4. Prepare ticket records

Use the supplied `set_ticket_price(city, country, price, eta)` function to populate the database. The price is converted to a float, and travel time accepts either a numeric hour value or an `HH:MM:SS` string.

The supplied code creates the table but contains no calls that insert ticket records.

---

## ▶️ Usage

1. Configure `OPENAI_API_KEY` and define the missing `system_message` variable.
2. Run the initialization and function-definition cells in order.
3. Populate the database using `set_ticket_price()`.
4. Inspect the records through the pandas query and `display(df1)`.
5. Run the Gradio interface and callback setup, followed by `ui.launch()`.
6. Sign in using the authentication pair configured in the supplied launch call.
7. Enter a message in **Chat with our AI Assistant:** and submit it.
8. Review the reply, generated audio, and any destination image.

The launch call uses `inbrowser=True`, `share=True`, and a hardcoded authentication pair. Each completed chat reply triggers text-to-speech; image generation depends on the tool-call behavior described above.

---

## 🔄 Message Flow

| Step | Action | Context / output |
|---|---|---|
| 1 | Add the submitted user message | `put_message_in_chatbot()` clears the textbox and updates the chat history |
| 2 | Request a model response | `chat()` sends the system message, conversation history, and both tool definitions |
| 3 | Execute requested database tools | The handler queries ticket prices or travel times and returns tool messages with matching IDs |
| 4 | Continue the tool loop | Tool results are appended to the request messages until the model returns a final reply |
| 5 | Generate audio | `talker()` converts the final reply into speech |
| 6 | Generate an image when applicable | `artist()` uses the last price-lookup city in the final tool-call batch |
| 7 | Update the interface | The callback returns chat history, audio bytes, and an image or `None` |

Tool-call messages are used within the current request loop. The returned chat history contains user and assistant messages rather than the database tool messages.

---

## 📝 Current Limitations

- `system_message` is referenced but not defined in the supplied snippet.
- The database is initially empty unless records already exist; no seed records or import routine are supplied.
- Prices and travel times come from stored SQLite records; the code contains no live travel-data lookup or booking function.
- Country is stored but is not returned by either database tool.
- Image generation is tied to price lookups. Travel-time-only lookups do not add cities for image generation.
- Each tool-call batch replaces the `cities` list, so cities from earlier batches are not retained for image selection.
- `display(df1)` relies on a notebook-provided `display()` function that the snippet does not import.
- API calls, tool arguments, and travel-time parsing have no exception or retry handling.
- Chat history is passed through the interface; the code contains no conversation export or database persistence for chat messages.
- The launch call contains hardcoded authentication credentials, and initialization prints part of the API key.

---

## 🔮 Possible Enhancements

The following adjustments are grounded in functions and configuration already present in the code; they are not implemented in the supplied snippet.

- **System prompt:** Define the `system_message` variable used by `chat()`.
- **Ticket records:** Populate or update the database through the existing `set_ticket_price()` function.
- **Voice selection:** Replace `onyx` with `alloy` or `coral`, as suggested in the code comment.
- **Image appearance:** Adjust the existing destination-image prompt or its pop-art styling.
- **Interface configuration:** Adjust the panel heights and the launch settings already defined in the Gradio interface.
