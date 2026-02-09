Core Concepts
=============

This section explains the fundamental concepts and architecture of AgenticPay.

Framework Overview
------------------

AgenticPay is built around three main components:

1. **Environments**: Define the negotiation scenario and rules
2. **Agents**: LLM-powered buyer and seller entities
3. **Models**: LLM backends for agent reasoning

.. code-block:: text

   ┌─────────────────────────────────────────────────────────┐
   │                    Environment                          │
   │  ┌─────────────┐              ┌─────────────┐          │
   │  │   Buyer     │◄────────────►│   Seller    │          │
   │  │   Agent     │  Negotiation │   Agent     │          │
   │  └──────┬──────┘              └──────┬──────┘          │
   │         │                            │                  │
   │         └────────────┬───────────────┘                  │
   │                      │                                  │
   │              ┌───────▼───────┐                          │
   │              │  LLM Model    │                          │
   │              └───────────────┘                          │
   └─────────────────────────────────────────────────────────┘

Environment Types
-----------------

AgenticPay provides various environment types organized by complexity:

Single Buyer + Product + Seller
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Basic scenarios with one buyer, one product, and one seller.

- **Task1**: Basic Price Negotiation
- **Task2**: Close Price Negotiation (narrow price ranges)
- **Task3**: Close to Market Price Negotiation

Multi-Product Environments
~~~~~~~~~~~~~~~~~~~~~~~~~~

One buyer and seller negotiating over multiple products.

- **Task1**: General multi-product negotiation
- **Task2**: Two product negotiation
- **Task3**: Five product negotiation
- **Task4**: Select three from five products

Multi-Seller Environments
~~~~~~~~~~~~~~~~~~~~~~~~~

One buyer negotiating with multiple sellers.

- **Parallel**: Simultaneous negotiations with multiple sellers
- **Sequential**: One-by-one negotiations with sellers

Multi-Buyer Environments
~~~~~~~~~~~~~~~~~~~~~~~~

Multiple buyers competing for products.

- **Parallel**: Simultaneous buyer negotiations
- **Sequential**: One-by-one buyer negotiations

Complex Multi-Agent Environments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Combinations of multiple buyers, sellers, and products:

- Multi-Buyer + Multi-Seller
- Multi-Products + Multi-Seller
- Multi-Buyer + Multi-Products
- Multi-Buyer + Multi-Products + Multi-Seller

Agent Architecture
------------------

Base Agent
~~~~~~~~~~

All agents inherit from ``BaseAgent`` which provides:

- Model integration for LLM inference
- Response generation interface
- State management

.. code-block:: python

   class BaseAgent:
       def __init__(self, model):
           self.model = model
       
       def respond(self, conversation_history, current_state):
           # Generate response using LLM
           pass

Buyer Agent
~~~~~~~~~~~

The ``BuyerAgent`` represents a customer trying to purchase products.

**Key attributes:**

- ``buyer_max_price``: Maximum price the buyer is willing to pay (confidential)
- ``model``: LLM model for generating responses

**Behavior:**

- Negotiates to get the lowest possible price
- Uses user requirements and preferences
- May walk away if price exceeds maximum

Seller Agent
~~~~~~~~~~~~

The ``SellerAgent`` represents a merchant selling products.

**Key attributes:**

- ``seller_min_price``: Minimum acceptable price (confidential)
- ``model``: LLM model for generating responses

**Behavior:**

- Negotiates to maximize sale price
- Uses product information and market conditions
- May refuse offers below minimum price

Conversation Memory
-------------------

The ``ConversationMemory`` class manages dialogue history:

.. code-block:: python

   from agenticpay.memory import ConversationMemory

   memory = ConversationMemory()
   
   # Add messages
   memory.add_message(role="buyer", content="I'd like to buy this jacket")
   memory.add_message(role="seller", content="It's priced at $150")
   
   # Retrieve history
   full_history = memory.get_history()
   recent = memory.get_recent(n=5)

**Features:**

- Message storage with metadata (role, round, timestamp)
- Full or recent history retrieval
- Role-based filtering

State Management
----------------

The negotiation state tracks:

.. code-block:: python

   observation = {
       "conversation_history": [...],    # List of messages
       "current_round": 3,               # Current negotiation round
       "seller_price": 130.0,            # Current seller asking price
       "buyer_offer": 100.0,             # Latest buyer offer
       "product_info": {...},            # Product details
       "environment_info": {...},        # Environmental context
   }

Reward System
-------------

AgenticPay uses a configurable reward system:

.. code-block:: python

   reward_weights = {
       "buyer_savings": 1.0,    # Weight for buyer savings
       "seller_profit": 1.0,    # Weight for seller profit
       "time_cost": 0.1,        # Penalty for negotiation rounds
   }

**Reward Components:**

- **Buyer Savings**: ``buyer_max_price - final_price``
- **Seller Profit**: ``final_price - seller_min_price``
- **Time Cost**: Penalty based on rounds used

Registration System
-------------------

AgenticPay uses a Gymnasium-like registration system:

.. code-block:: python

   from agenticpay import make, register
   from agenticpay.envs import pprint_registry

   # List all registered environments
   pprint_registry()

   # Create environment by ID
   env = make("Task1_basic_price_negotiation-v0", ...)

   # Register custom environment
   register(
       id="MyCustomEnv-v0",
       entry_point="my_module:MyEnvClass",
       max_episode_steps=100,
   )
