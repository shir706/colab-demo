Write a Google Colab notebook using Python, LangGraph, and langchain-openai to build a Support Ticket Router.

Technical Requirements & Environment:
- Install packages: `!pip -q install langgraph langchain-openai`
- Draw the graph using `draw_mermaid_png()`.
- Load the OpenAI API key securely using Colab Secrets ONLY:
  ```python
  import os
  from google.colab import userdata
 os.environ["OPENAI_API_KEY"] = userdata.get("OPENAI_API_KEY")

The code logic is perfect. Now, please do two things:
1. Update Cell 7 to run app.invoke() on a list of the 6 specific test tickets required by the original prompt (refund billing, upgrade billing, technical crash, technical login error, general thanks, and the ambiguous ticket: "The app crashes every time I try to pay"). Loop through them and clearly print the 'category', 'team', and 'reply' for each.
2. Generate a downloadable .ipynb file containing all this updated code. CRITICAL: Ensure the JSON structure of the .ipynb file is completely valid and that absolutely NO Python line breaks (\n) are lost in the source code cells this time.

The Python code is perfect, but the .ipynb file you generated is broken because it squashes all the code into a single line. Please generate the downloadable .ipynb file again, and make absolutely sure to preserve all line breaks (\n) in the JSON structure.

