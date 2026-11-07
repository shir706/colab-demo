Write a complete Google Colab notebook using Python and LangGraph to build an order management system for a bakery.

Technical Requirements:
- Strictly use deterministic Python. NO LLM calls (no API keys needed).
- Draw the graph using `draw_mermaid_png()`.
- Input is a dict with: {"item": str, "quantity": int, "day": str, "city": str}
- Output must contain these exact keys: 'status' ("accepted" or "rejected"), 'product_actually_sold', 'delivery' ("pickup" or "delivery"), 'total' (final price with delivery), 'message' (warm, hard-coded string reply), and 'steps' (a list tracking every node visited).

Menu & Inventory State (Hardcode these dicts):
- prices: challah: 18, babka: 45, rugelach: 30, sourdough: 25
- inventory: challah: 10, babka: 0, rugelach: 3, sourdough: 4

Graph Nodes & Logic:
1. State: Define a TypedDict including all input and output fields. Initialize 'steps' as an empty list.
2. Node 1 - `validate_order`: If quantity <= 0 or item not in menu, set status="rejected", message="Politely: couldn't understand order". Else status="accepted", product_actually_sold=item. Append "validate_order" to 'steps'.
3. Node 2 - `inventory_and_subs` (Runs if accepted): Check if enough inventory for 'product_actually_sold'. If not, try max 2 substitutions in this exact order: babka -> rugelach -> sourdough -> challah. Substitution is valid ONLY if there is enough inventory for the FULL requested quantity. Update 'product_actually_sold'. If 2 suggestions fail or no stock for the full quantity, set status="rejected", message="Apologize, out of stock and cancelling order". Append "inventory_and_subs" to 'steps'.
4. Node 3 - `delivery_calc` (Runs if accepted): If city == "Jerusalem", delivery="delivery", delivery_fee=20. Else (or if city is empty), delivery="pickup", delivery_fee=0. Append "delivery_calc" to 'steps'.
5. Node 4 - `discount_calc` (Runs if accepted): Calculate base total (quantity * price of product_actually_sold). If base total > 150 AND day != "Friday", apply a 15% discount to the base total only. Set 'total' = (discounted base total) + delivery_fee. Append "discount_calc" to 'steps'.
6. Node 5 - `format_reply` (Runs if accepted): Generate a warm, hard-coded response string for 'message' confirming the final order details, substitutions (if any), and delivery method. Append "format_reply" to 'steps'.

Edges:
Create a graph starting at 'validate_order'. Use conditional edges after 'validate_order' and 'inventory_and_subs' to route to END if status == "rejected". Otherwise, flow sequentially through the nodes.

Finally, compile the graph and write Python code at the bottom of the notebook to run invoke() on 6 different test cases (including one normal, one rejected due to invalid input, one requiring substitution, one cancelled due to no subs, one with discount, one outside Jerusalem). Print the results beautifully.