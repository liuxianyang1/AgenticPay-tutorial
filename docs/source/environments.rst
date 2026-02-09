Environments
============

This section provides detailed documentation for all available negotiation environments.

Environment API
---------------

All environments follow the Gymnasium-like API:

.. code-block:: python

   # Create environment
   env = make("Environment-ID-v0", **kwargs)

   # Reset and get initial observation
   observation, info = env.reset(**reset_kwargs)

   # Run negotiation step
   observation, reward, terminated, truncated, info = env.step(
       buyer_action=buyer_action,
       seller_action=seller_action
   )

   # Display current state
   env.render()

   # Cleanup
   env.close()

Common Parameters
-----------------

All environments share these configuration parameters:

.. list-table::
   :header-rows: 1
   :widths: 25 15 60

   * - Parameter
     - Type
     - Description
   * - ``buyer_agent``
     - BuyerAgent
     - The buyer agent instance
   * - ``seller_agent``
     - SellerAgent
     - The seller agent instance
   * - ``max_rounds``
     - int
     - Maximum negotiation rounds
   * - ``initial_seller_price``
     - float
     - Starting price from seller
   * - ``buyer_max_price``
     - float
     - Maximum acceptable price for buyer
   * - ``seller_min_price``
     - float
     - Minimum acceptable price for seller
   * - ``price_tolerance``
     - float
     - Price difference threshold for agreement
   * - ``environment_info``
     - dict
     - Contextual information (weather, season, etc.)
   * - ``reward_weights``
     - dict
     - Weights for reward components

Single Buyer + Product + Seller
-------------------------------

Basic negotiation scenarios with one buyer, one product, and one seller.

Task1: Basic Price Negotiation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Environment ID:** ``Task1_basic_price_negotiation-v0``

Standard price negotiation between buyer and seller.

.. code-block:: python

   env = make(
       "Task1_basic_price_negotiation-v0",
       buyer_agent=buyer,
       seller_agent=seller,
       max_rounds=20,
       initial_seller_price=150.0,
       buyer_max_price=120.0,
       seller_min_price=80.0,
   )

Task2: Close Price Negotiation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Environment ID:** ``Task2_close_price_negotiation-v0``

Tests edge cases with narrow price ranges where buyer max and seller min are close.

Task3: Close to Market Price Negotiation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Environment ID:** ``Task3_close_to_market_price_negotiation-v0``

Tests scenarios where initial price is near the market/fair price.

Multi-Product Environments
--------------------------

Environments for negotiating multiple products.

Task1: Multi-Product Negotiation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Environment ID:** ``Task1_multi_product_negotiation-v0``

General multi-product negotiation.

.. code-block:: python

   env = make(
       "Task1_multi_product_negotiation-v0",
       buyer_agent=buyer,
       seller_agent=seller,
       max_rounds_per_product=20,
   )

   observation, info = env.reset(
       products=[
           {"name": "Laptop", "price": 1000.0},
           {"name": "Mouse", "price": 50.0},
           {"name": "Keyboard", "price": 80.0},
       ]
   )

Task2-4: Specific Product Counts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **Task2**: Two product negotiation
- **Task3**: Five product negotiation
- **Task4**: Select three from five products

Multi-Seller Environments
-------------------------

Environments with multiple sellers competing for a single buyer.

Parallel Multi-Seller
~~~~~~~~~~~~~~~~~~~~~

**Environment IDs:**

- ``Task1_parallel_two_seller_negotiation-v0``
- ``Task2_parallel_three_seller_negotiation-v0``

Buyer negotiates with multiple sellers simultaneously.

.. code-block:: python

   from agenticpay.agents.seller_agent import SellerAgent

   seller1 = SellerAgent(model=model, seller_min_price=80.0)
   seller2 = SellerAgent(model=model, seller_min_price=85.0)

   env = make(
       "Task1_parallel_two_seller_negotiation-v0",
       buyer_agent=buyer,
       seller_agents=[seller1, seller2],
       max_rounds=20,
   )

Sequential Multi-Seller
~~~~~~~~~~~~~~~~~~~~~~~

**Environment IDs:**

- ``Task3_sequential_two_seller_negotiation-v0``
- ``Task4_sequential_three_seller_negotiation-v0``

Buyer negotiates with sellers one at a time, using previous negotiations as context.

Multi-Buyer Environments
------------------------

Environments with multiple buyers competing for products.

Parallel Multi-Buyer
~~~~~~~~~~~~~~~~~~~~

**Environment IDs:**

- ``Task1_parallel_two_buyer_negotiation-v0``
- ``Task2_parallel_three_buyer_negotiation-v0``

Multiple buyers negotiate with the seller simultaneously.

.. code-block:: python

   buyer1 = BuyerAgent(model=model, buyer_max_price=120.0)
   buyer2 = BuyerAgent(model=model, buyer_max_price=115.0)

   env = make(
       "Task1_parallel_two_buyer_negotiation-v0",
       buyer_agents=[buyer1, buyer2],
       seller_agent=seller,
       max_rounds=20,
   )

Sequential Multi-Buyer
~~~~~~~~~~~~~~~~~~~~~~

**Environment IDs:**

- ``Task3_sequential_two_buyer_negotiation-v0``
- ``Task4_sequential_three_buyer_negotiation-v0``

Buyers negotiate with the seller one at a time.

Complex Multi-Agent Environments
--------------------------------

Multi-Buyer + Multi-Seller
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Environment IDs:**

- ``Task1_parallel_two_buyer_two_seller_negotiation-v0``
- ``Task2_parallel_three_buyer_three_seller_negotiation-v0``
- ``Task3_sequential_two_buyer_two_seller_negotiation-v0``
- ``Task4_sequential_three_buyer_three_seller_negotiation-v0``

Multi-Products + Multi-Seller
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Environment IDs:**

- ``Task1_parallel_two_seller_per_one_product_negotiation-v0``
- ``Task2_parallel_three_seller_per_one_product_negotiation-v0``
- ``Task3_sequential_two_seller_per_one_product_negotiation-v0``
- ``Task4_sequential_three_seller_per_one_product_negotiation-v0``

Multi-Buyer + Multi-Products
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Environment IDs:**

- ``Task1_parallel_two_buyer_two_product_negotiation-v0``
- ``Task2_parallel_three_buyer_two_product_negotiation-v0``
- ``Task3_sequential_two_buyer_two_product_negotiation-v0``
- ``Task4_sequential_three_buyer_two_product_negotiation-v0``

Multi-Buyer + Multi-Products + Multi-Seller
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The most complex scenarios:

**Environment IDs:**

- ``Task1_parallel_two_buyer_two_seller_two_product_negotiation-v0``
- ``Task2_parallel_three_buyer_three_seller_two_product_negotiation-v0``
- ``Task3_sequential_two_buyer_two_seller_two_product_negotiation-v0``
- ``Task4_sequential_three_buyer_three_seller_three_product_negotiation-v0``

Creating Custom Environments
----------------------------

You can create custom environments by inheriting from ``BaseEnv``:

.. code-block:: python

   from agenticpay.core import BaseEnv
   from agenticpay.envs import register

   class MyCustomEnv(BaseEnv):
       def __init__(self, buyer_agent, seller_agent, **kwargs):
           super().__init__()
           self.buyer_agent = buyer_agent
           self.seller_agent = seller_agent
           # Custom initialization

       def reset(self, **kwargs):
           # Initialize negotiation state
           observation = {...}
           info = {...}
           return observation, info

       def step(self, buyer_action, seller_action):
           # Process actions and update state
           observation = {...}
           reward = 0.0
           terminated = False
           truncated = False
           info = {...}
           return observation, reward, terminated, truncated, info

       def render(self):
           # Display current state
           pass

       def close(self):
           # Cleanup resources
           pass

   # Register the environment
   register(
       id="MyCustomEnv-v0",
       entry_point="my_module:MyCustomEnv",
       max_episode_steps=100,
   )
