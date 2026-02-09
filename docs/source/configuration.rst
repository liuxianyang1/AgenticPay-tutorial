Configuration
=============

This section covers all configuration options available in AgenticPay.

Environment Configuration
-------------------------

Basic Parameters
~~~~~~~~~~~~~~~~

.. code-block:: python

   env = make(
       "Task1_basic_price_negotiation-v0",
       
       # Required: Agent instances
       buyer_agent=buyer,
       seller_agent=seller,
       
       # Negotiation limits
       max_rounds=20,                    # Maximum negotiation rounds
       
       # Price settings
       initial_seller_price=150.0,       # Starting seller price
       buyer_max_price=120.0,            # Buyer's maximum (confidential)
       seller_min_price=80.0,            # Seller's minimum (confidential)
       price_tolerance=0.0,              # Threshold for price agreement
       
       # Context information
       environment_info={
           "temperature": "warm",
           "season": "summer",
           "weather": "sunny",
           "location": "downtown mall",
       },
       
       # Reward configuration
       reward_weights={
           "buyer_savings": 1.0,
           "seller_profit": 1.0,
           "time_cost": 0.1,
       },
   )

Reward Weights
~~~~~~~~~~~~~~

The reward system balances different objectives:

.. list-table::
   :header-rows: 1
   :widths: 25 60

   * - Weight
     - Description
   * - ``buyer_savings``
     - Encourages buyer to save money (``buyer_max_price - final_price``)
   * - ``seller_profit``
     - Encourages seller profit (``final_price - seller_min_price``)
   * - ``time_cost``
     - Penalizes long negotiations (fewer rounds = better)

Example configurations:

.. code-block:: python

   # Buyer-focused (maximize savings)
   reward_weights = {
       "buyer_savings": 2.0,
       "seller_profit": 0.5,
       "time_cost": 0.1,
   }

   # Balanced
   reward_weights = {
       "buyer_savings": 1.0,
       "seller_profit": 1.0,
       "time_cost": 0.1,
   }

   # Quick resolution focused
   reward_weights = {
       "buyer_savings": 1.0,
       "seller_profit": 1.0,
       "time_cost": 0.5,
   }

Agent Configuration
-------------------

Buyer Agent
~~~~~~~~~~~

.. code-block:: python

   buyer = BuyerAgent(
       model=model,
       buyer_max_price=120.0,    # Maximum acceptable price
   )

Seller Agent
~~~~~~~~~~~~

.. code-block:: python

   seller = SellerAgent(
       model=model,
       seller_min_price=80.0,    # Minimum acceptable price
   )

Model Configuration
-------------------

SGLang
~~~~~~

.. code-block:: python

   from agenticpay.models.sglang_lm import SGLangLM

   model = SGLangLM(
       model_path="agenticpay/models/download_models/Qwen3-8B-Instruct",
   )

vLLM
~~~~

.. code-block:: python

   from agenticpay.models.vllm_lm import VLLMLM

   model = VLLMLM(
       model_path="path/to/model",
       trust_remote_code=True,
       gpu_memory_utilization=0.9,     # GPU memory usage (0.0-1.0)
       tensor_parallel_size=4,          # Number of GPUs for parallelism
   )

OpenAI
~~~~~~

.. code-block:: python

   from agenticpay.models.openai_lm import OpenAILLM

   model = OpenAILLM(
       model_name="gpt-4",              # Model name
       api_key="sk-...",                # API key (or use env var)
       temperature=0.7,                 # Response randomness
       max_tokens=1024,                 # Maximum response length
   )

Reset Configuration
-------------------

When resetting an environment, you can configure:

Product Information
~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   observation, info = env.reset(
       user_requirement="I need a high-quality winter jacket",
       product_info={
           "name": "Premium Winter Jacket",
           "brand": "Mountain Gear",
           "price": 180.0,
           "features": ["Waterproof", "Insulated", "Windproof", "Breathable"],
           "condition": "New",
           "material": "Gore-Tex",
           "warranty": "2 years",
       },
   )

User Profile
~~~~~~~~~~~~

.. code-block:: python

   user_profile = """
   - Prefers business/professional style
   - Budget-conscious but values quality
   - Likes to compare prices before purchasing
   - May mention competitor prices during negotiation
   """

   observation, info = env.reset(
       user_requirement="I need a laptop for work",
       product_info={...},
       user_profile=user_profile,
   )

Multi-Product Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   observation, info = env.reset(
       products=[
           {
               "name": "Laptop",
               "price": 1200.0,
               "features": ["16GB RAM", "512GB SSD"],
           },
           {
               "name": "Mouse",
               "price": 50.0,
               "features": ["Wireless", "Ergonomic"],
           },
           {
               "name": "Keyboard",
               "price": 100.0,
               "features": ["Mechanical", "RGB"],
           },
       ]
   )

Environment Variables
---------------------

AgenticPay supports these environment variables:

.. code-block:: bash

   # OpenAI API Key
   export OPENAI_API_KEY="sk-..."

   # HuggingFace Token (for gated models)
   export HF_TOKEN="hf_..."

   # CUDA Device Selection
   export CUDA_VISIBLE_DEVICES="0,1,2,3"

Logging Configuration
---------------------

Configure logging for debugging:

.. code-block:: python

   import logging

   # Set logging level
   logging.basicConfig(level=logging.INFO)

   # Enable detailed negotiation logging
   logging.getLogger("agenticpay").setLevel(logging.DEBUG)

Best Practices
--------------

1. **Price Ranges**: Ensure ``buyer_max_price > seller_min_price`` for achievable deals

2. **Round Limits**: Set ``max_rounds`` based on complexity (5-10 for simple, 20-30 for complex)

3. **Memory Management**: For long sessions, consider clearing conversation history periodically

4. **GPU Memory**: Monitor GPU usage with vLLM's ``gpu_memory_utilization`` parameter

5. **Temperature**: Lower temperature (0.3-0.5) for more consistent negotiations, higher (0.7-0.9) for varied responses
