Agents
======

This section provides detailed documentation for the agent system in AgenticPay.

Agent Overview
--------------

Agents are the core participants in negotiation scenarios. Each agent uses an LLM 
to generate contextually appropriate responses based on the conversation history 
and their internal objectives.

Base Agent
----------

All agents inherit from the ``BaseAgent`` class:

.. code-block:: python

   from agenticpay.agents.base_agent import BaseAgent

   class BaseAgent:
       def __init__(self, model):
           """
           Initialize the agent.
           
           Args:
               model: LLM model instance for generating responses
           """
           self.model = model
       
       def respond(self, conversation_history, current_state):
           """
           Generate a response based on conversation history and state.
           
           Args:
               conversation_history: List of previous messages
               current_state: Current negotiation state dictionary
           
           Returns:
               str: Agent's response message
           """
           raise NotImplementedError

Buyer Agent
-----------

The ``BuyerAgent`` represents a customer in the negotiation.

Initialization
~~~~~~~~~~~~~~

.. code-block:: python

   from agenticpay.agents.buyer_agent import BuyerAgent

   buyer = BuyerAgent(
       model=model,
       buyer_max_price=120.0,  # Maximum price willing to pay
   )

Parameters
~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 25 15 60

   * - Parameter
     - Type
     - Description
   * - ``model``
     - LLM
     - Language model for response generation
   * - ``buyer_max_price``
     - float
     - Maximum acceptable price (confidential)

Usage
~~~~~

.. code-block:: python

   # Generate buyer response
   buyer_response = buyer.respond(
       conversation_history=observation["conversation_history"],
       current_state=observation
   )

   print(f"Buyer says: {buyer_response}")

Behavior
~~~~~~~~

The buyer agent:

- Tries to negotiate the lowest possible price
- Uses user requirements and preferences
- May make counter-offers below the seller's asking price
- Will reject deals above ``buyer_max_price``
- May walk away if negotiation isn't progressing

Seller Agent
------------

The ``SellerAgent`` represents a merchant in the negotiation.

Initialization
~~~~~~~~~~~~~~

.. code-block:: python

   from agenticpay.agents.seller_agent import SellerAgent

   seller = SellerAgent(
       model=model,
       seller_min_price=80.0,  # Minimum acceptable price
   )

Parameters
~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 25 15 60

   * - Parameter
     - Type
     - Description
   * - ``model``
     - LLM
     - Language model for response generation
   * - ``seller_min_price``
     - float
     - Minimum acceptable price (confidential)

Usage
~~~~~

.. code-block:: python

   # Generate seller response
   seller_response = seller.respond(
       conversation_history=observation["conversation_history"],
       current_state=observation
   )

   print(f"Seller says: {seller_response}")

Behavior
~~~~~~~~

The seller agent:

- Tries to maximize the sale price
- Uses product information to justify pricing
- May offer limited discounts or promotions
- Will reject offers below ``seller_min_price``
- Highlights product features and value

LLM Models
----------

AgenticPay supports multiple LLM backends:

SGLang
~~~~~~

High-performance serving framework for single GPU setups.

.. code-block:: python

   from agenticpay.models.sglang_lm import SGLangLM

   model = SGLangLM(
       model_path="path/to/model",
   )

vLLM
~~~~

Fast LLM inference with PagedAttention, ideal for multi-GPU setups.

.. code-block:: python

   from agenticpay.models.vllm_lm import VLLMLM

   model = VLLMLM(
       model_path="path/to/model",
       trust_remote_code=True,
       gpu_memory_utilization=0.9,
       tensor_parallel_size=4,  # Number of GPUs
   )

OpenAI API
~~~~~~~~~~

For using OpenAI's hosted models.

.. code-block:: python

   from agenticpay.models.openai_lm import OpenAILLM

   model = OpenAILLM(
       model_name="gpt-4",
       api_key="your-api-key",  # Or use OPENAI_API_KEY env var
   )

HuggingFace
~~~~~~~~~~~

For using HuggingFace models directly.

.. code-block:: python

   from agenticpay.models.huggingface_lm import HuggingFaceLLM

   model = HuggingFaceLLM(
       model_name="meta-llama/Llama-2-7b-chat-hf",
       device="cuda",
   )

User Profiles
-------------

User profiles allow personalizing buyer behavior:

.. code-block:: python

   user_profile = """
   User prefers business/professional style and likes to compare prices.
   In negotiations, they may mention comparing other options and seek better deals.
   They are budget-conscious but value quality.
   """

   observation, info = env.reset(
       user_requirement="I need a laptop",
       product_info={...},
       user_profile=user_profile,
   )

The buyer agent incorporates this profile when generating responses, making 
the negotiation more realistic and personalized.

Creating Custom Agents
----------------------

You can create custom agents by extending ``BaseAgent``:

.. code-block:: python

   from agenticpay.agents.base_agent import BaseAgent

   class AggressiveBuyerAgent(BaseAgent):
       """A buyer that negotiates aggressively for lower prices."""
       
       def __init__(self, model, buyer_max_price, aggression_level=0.8):
           super().__init__(model)
           self.buyer_max_price = buyer_max_price
           self.aggression_level = aggression_level
       
       def respond(self, conversation_history, current_state):
           # Build custom prompt with aggressive negotiation style
           prompt = self._build_aggressive_prompt(
               conversation_history, 
               current_state
           )
           
           # Generate response using the model
           response = self.model.generate(prompt)
           
           return response
       
       def _build_aggressive_prompt(self, history, state):
           # Custom prompt building logic
           return f"""
           You are an aggressive negotiator. 
           Your maximum budget is ${self.buyer_max_price}.
           Always push for at least 30% off the asking price.
           ...
           """

   # Use the custom agent
   aggressive_buyer = AggressiveBuyerAgent(
       model=model,
       buyer_max_price=100.0,
       aggression_level=0.9
   )

Multi-Agent Scenarios
---------------------

For environments with multiple agents:

.. code-block:: python

   # Multiple buyers
   buyers = [
       BuyerAgent(model=model, buyer_max_price=120.0),
       BuyerAgent(model=model, buyer_max_price=115.0),
       BuyerAgent(model=model, buyer_max_price=110.0),
   ]

   # Multiple sellers
   sellers = [
       SellerAgent(model=model, seller_min_price=80.0),
       SellerAgent(model=model, seller_min_price=85.0),
   ]

   # Use in multi-agent environment
   env = make(
       "Task2_parallel_three_buyer_two_seller_negotiation-v0",
       buyer_agents=buyers,
       seller_agents=sellers,
       max_rounds=20,
   )

Each agent maintains its own objectives and negotiation strategy, creating 
complex and realistic multi-party negotiations.
