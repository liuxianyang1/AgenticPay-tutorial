Quick Start
===========

This guide will help you run your first negotiation simulation with AgenticPay.

Running the Example Script
--------------------------

The quickest way to try AgenticPay is to run the provided example script:

.. code-block:: console

   python agenticpay/examples/single_buyer_product_seller/Task1_basic_price_negotiation.py

This runs a simple negotiation between a buyer and a seller for a single product.

Basic Negotiation Tutorial
--------------------------

Step 1: Import Required Modules
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   from agenticpay import make
   from agenticpay.agents.buyer_agent import BuyerAgent
   from agenticpay.agents.seller_agent import SellerAgent

   # For local models
   from agenticpay.models.sglang_lm import SGLangLM
   # Or: from agenticpay.models.vllm_lm import VLLMLM

Step 2: Initialize the Model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Option 1: SGLang (recommended for single GPU)
   model_path = "agenticpay/models/download_models/Qwen3-8B-Instruct"
   model = SGLangLM(model_path=model_path)

   # Option 2: vLLM (for multi-GPU setups)
   # from agenticpay.models.vllm_lm import VLLMLM
   # model = VLLMLM(
   #     model_path=model_path,
   #     trust_remote_code=True,
   #     gpu_memory_utilization=0.9,
   #     tensor_parallel_size=4,  # Number of GPUs
   # )

Step 3: Create Agents
~~~~~~~~~~~~~~~~~~~~~

Each agent has a confidential "bottom price" that influences their negotiation strategy:

.. code-block:: python

   # Buyer's maximum acceptable price (won't pay more than this)
   buyer_max_price = 120.0

   # Seller's minimum acceptable price (won't sell below this)
   seller_min_price = 80.0

   # Create agents
   buyer = BuyerAgent(model=model, buyer_max_price=buyer_max_price)
   seller = SellerAgent(model=model, seller_min_price=seller_min_price)

Step 4: Create the Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use the ``make()`` function to create a negotiation environment:

.. code-block:: python

   env = make(
       "Task1_basic_price_negotiation-v0",
       buyer_agent=buyer,
       seller_agent=seller,
       max_rounds=20,
       initial_seller_price=150.0,
       buyer_max_price=buyer_max_price,
       seller_min_price=seller_min_price,
       environment_info={
           "temperature": "warm",
           "season": "summer",
           "weather": "sunny",
       },
       price_tolerance=0.0,
   )

Step 5: Initialize and Run Negotiation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Optional: Define user profile for personalized negotiation
   user_profile = "User prefers business/professional style and likes to compare prices."

   # Reset environment with product information
   observation, info = env.reset(
       user_requirement="I need a high-quality winter jacket",
       product_info={
           "name": "Premium Winter Jacket",
           "brand": "Mountain Gear",
           "price": 180.0,
           "features": ["Waterproof", "Insulated", "Windproof"],
           "condition": "New",
       },
       user_profile=user_profile,
   )

   # Run negotiation loop
   done = False
   while not done:
       # Buyer responds first
       buyer_action = buyer.respond(
           conversation_history=observation["conversation_history"],
           current_state=observation
       )

       # Update conversation history
       updated_history = observation["conversation_history"].copy()
       if buyer_action:
           updated_history.append({
               "role": "buyer",
               "content": buyer_action,
               "round": observation.get("current_round", 0)
           })

       # Seller responds
       seller_action = seller.respond(
           conversation_history=updated_history,
           current_state=observation
       )

       # Execute step
       observation, reward, terminated, truncated, info = env.step(
           buyer_action=buyer_action,
           seller_action=seller_action
       )
       done = terminated or truncated

       # Display current state
       env.render()

   # Print results
   print(f"Negotiation ended: {info['status']}")
   print(f"Final price: ${info.get('seller_price', 'N/A')}")
   env.close()

Understanding the Output
------------------------

The negotiation can end in several ways:

- **Deal Reached**: Both parties agreed on a price
- **No Deal**: Parties couldn't agree within max rounds
- **Buyer Rejected**: Buyer explicitly rejected the offer
- **Seller Rejected**: Seller explicitly rejected the offer

The ``info`` dictionary contains detailed results:

.. code-block:: python

   {
       "status": "deal_reached",      # Outcome status
       "seller_price": 105.0,         # Final agreed price
       "buyer_savings": 15.0,         # Buyer saved from max price
       "seller_profit": 25.0,         # Seller profit above min price
       "rounds_used": 5,              # Number of negotiation rounds
   }

Next Steps
----------

- Learn about :doc:`core_concepts` to understand the framework architecture
- Explore different :doc:`environments` for various negotiation scenarios
- Check out more :doc:`examples` for advanced use cases
