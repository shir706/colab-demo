# Club Chatbot — Technical Specification for Codex

Build a Google Colab notebook (`.ipynb`) implementing a topic-locked LangGraph chatbot backed by a Supabase PostgreSQL checkpointer and a Gradio (`gr.Blocks`) interface.

## 1. Installation & Secrets
- Cell 1: Install packages using:
  `!pip -q install langgraph langchain langchain-openai langgraph-checkpoint-postgres "psycopg[binary]" gradio`
- Cell 2: Load both secrets from Google Colab Secrets using `google.colab.userdata.get`:
  - `os.environ["OPENAI_API_KEY"] = userdata.get("OPENAI_API_KEY")`
  - `SUPABASE_DB_URL = userdata.get("SUPABASE_DB_URL")`
  - Never hardcode any API key, password, or database URI.

## 2. Model Choice
- Use `init_chat_model("openai:gpt-4o-mini", temperature=0)` (chosen as a fast, low-cost OpenAI text model suitable for topic-locked Q&A). Keep the model name in a single constant `MODEL_NAME = "openai:gpt-4o-mini"` at the top of the cell.

## 3. LangGraph State, Node & Graph
- **Topics**: `["Astronomy", "Cooking", "Football", "World Geography", "Python Programming"]`
- **State (`ChatState`)**:
  - `messages`: `Annotated[list[AnyMessage], add_messages]`
  - `topic`: `str` (no reducer, so each new value replaces the previous one)
- **Single Node (`chatbot`)**:
  - Exactly one LLM call per user message.
  - Dynamically construct a `SystemMessage` from `state["topic"]` on every call. Do **not** store the `SystemMessage` in `state["messages"]`.
  - System message rules:
    - You are a helpful assistant for the `{topic}` club at Curious Minds.
    - Answer **only** questions strictly related to `{topic}`.
    - Politely refuse any off-topic question, explicitly mention that we are currently in the `{topic}` club, and suggest switching to another club from the dropdown (Astronomy, Cooking, Football, World Geography, Python Programming).
    - If the user asks a follow-up question about a previous topic that appears earlier in the conversation history, you **must** refuse it and remind them that we are now in the `{topic}` club.
  - Invoke the model with `[system_msg] + state["messages"]` and return `{"messages": [reply]}`.
- **Graph**: `START -> chatbot -> END`.

## 4. Supabase Checkpointer (PostgresSaver)
- Connect using `psycopg` and `PostgresSaver` (no `InMemorySaver`, no in-memory Python dict storing conversations):
```python
import psycopg
from psycopg.rows import dict_row
from langgraph.checkpoint.postgres import PostgresSaver

conn = psycopg.connect(
    userdata.get("SUPABASE_DB_URL"),
    autocommit=True,
    row_factory=dict_row,
    prepare_threshold=0,
)
checkpointer = PostgresSaver(conn)
checkpointer.setup()
graph = builder.compile(checkpointer=checkpointer)

Draw the graph in a dedicated cell using display(Image(graph.get_graph().draw_mermaid_png())).

5. Standalone Test Cell (Before Gradio)
Include a test cell that runs the graph directly without Gradio:

from langchain_core.messages import HumanMessage

config = {"configurable": {"thread_id": "test-1"}}

def ask(text, topic):
    result = graph.invoke({"messages": [HumanMessage(text)], "topic": topic}, config)
    print(f"[{topic}] {text}\n -> {result['messages'][-1].content}\n")

ask("How do I make pancakes?", "Cooking")
ask("Who won the 2022 World Cup?", "Cooking")
ask("Who won the 2022 World Cup?", "Football")
ask("And how long do I fry each pancake?", "Football")
graph.get_state(config).values

6. Gradio Interface (gr.Blocks)
Build a clean Gradio web interface with:

A Club / Topic dropdown (gr.Dropdown) with the 5 clubs. Changing the topic continues the same thread_id, and subsequent messages follow only the new topic.

A Saved Conversations dropdown (gr.Dropdown) populated directly by querying the Supabase checkpoints table (e.g., distinct thread_id values) and reading each thread's state via graph.get_state({"configurable": {"thread_id": tid}}).values. Display human-readable labels like "{topic} — {first_30_chars_of_first_user_message}" (mapped to the thread_id), not raw UUIDs. Selecting a saved conversation loads both its messages and its saved topic into the UI, and new messages continue that thread.

A + New conversation button (gr.Button) that generates a new str(uuid.uuid4()) as the active thread_id and clears the chat window.

A Chat window (gr.Chatbot) and Message input (gr.Textbox) that sends only the new user message and current topic to graph.invoke(...) with the active thread_id, then refreshes the chat and saved conversations list.

Keep the code clean and divided into clear notebook cells.

