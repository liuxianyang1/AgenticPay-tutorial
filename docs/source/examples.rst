Examples
========

This section provides complete working examples for various negotiation scenarios.

Basic Single-Product Negotiation
--------------------------------

The simplest negotiation scenario with one buyer, one product, and one seller.

.. code-block:: python

   from agenticpay import make
   from agenticpay.agents.buyer_agent import BuyerAgent
   from agenticpay.agents.seller_agent import SellerAgent
   from agenticpay.models.sglang_lm import SGLangLM

   # Initialize model
   model_path = "agenticpay/models/download_models/Qwen3-8B-Instruct"
   model = SGLangLM(model_path=model_path)

   # Create agents with confidential price limits
   buyer_max_price = 120.0
   seller_min_price = 80.0

   buyer = BuyerAgent(model=model, buyer_max_price=buyer_max_price)
   seller = SellerAgent(model=model, seller_min_price=seller_min_price)

   # Create environment
   env = make(
       "Task1_basic_price_negotiation-v0",
       buyer_agent=buyer,
       seller_agent=seller,
       max_rounds=20,
       initial_seller_price=150.0,
       buyer_max_price=buyer_max_price,
       seller_min_price=seller_min_price,
       price_tolerance=0.0,
   )

   # Reset with product info
   observation, info = env.reset(
       user_requirement="I need a high-quality winter jacket",
       product_info={
           "name": "Premium Winter Jacket",
           "brand": "Mountain Gear",
           "price": 180.0,
           "features": ["Waterproof", "Insulated", "Windproof"],
           "condition": "New",
       },
   )

   # Run negotiation
   done = False
   while not done:
       buyer_action = buyer.respond(
           conversation_history=observation["conversation_history"],
           current_state=observation
       )

       updated_history = observation["conversation_history"].copy()
       if buyer_action:
           updated_history.append({
               "role": "buyer",
               "content": buyer_action,
               "round": observation.get("current_round", 0)
           })

       seller_action = seller.respond(
           conversation_history=updated_history,
           current_state=observation
       )

       observation, reward, terminated, truncated, info = env.step(
           buyer_action=buyer_action,
           seller_action=seller_action
       )
       done = terminated or truncated
       env.render()

   print(f"Result: {info['status']}")
   print(f"Final Price: ${info.get('seller_price', 'N/A')}")
   env.close()

Multi-Product Negotiation
-------------------------

Negotiating multiple products in sequence.

.. code-block:: python

   from agenticpay import make
   from agenticpay.agents.buyer_agent import BuyerAgent
   from agenticpay.agents.seller_agent import SellerAgent
   from agenticpay.models.sglang_lm import SGLangLM

   model = SGLangLM(model_path="path/to/model")

   buyer = BuyerAgent(model=model, buyer_max_price=500.0)
   seller = SellerAgent(model=model, seller_min_price=300.0)

   env = make(
       "Task1_multi_product_negotiation-v0",
       buyer_agent=buyer,
       seller_agent=seller,
       max_rounds_per_product=15,
   )

   # Multiple products to negotiate
   products = [
       {"name": "Laptop Stand", "price": 80.0, "features": ["Adjustable", "Aluminum"]},
       {"name": "Wireless Mouse", "price": 45.0, "features": ["Ergonomic", "USB-C"]},
       {"name": "USB Hub", "price": 35.0, "features": ["4-port", "USB 3.0"]},
   ]

   observation, info = env.reset(
       user_requirement="I'm setting up my home office",
       products=products,
   )

   # Negotiation loop for all products
   done = False
   while not done:
       buyer_action = buyer.respond(
           observation["conversation_history"],
           observation
       )
       seller_action = seller.respond(
           observation["conversation_history"],
           observation
       )

       observation, reward, terminated, truncated, info = env.step(
           buyer_action=buyer_action,
           seller_action=seller_action
       )
       done = terminated or truncated
       env.render()

   print(f"Total spent: ${info.get('total_price', 'N/A')}")
   env.close()

Parallel Multi-Seller Negotiation
---------------------------------

A buyer negotiating with multiple sellers simultaneously to find the best deal.

.. code-block:: python

   from agenticpay import make
   from agenticpay.agents.buyer_agent import BuyerAgent
   from agenticpay.agents.seller_agent import SellerAgent
   from agenticpay.models.sglang_lm import SGLangLM

   model = SGLangLM(model_path="path/to/model")

   # One buyer
   buyer = BuyerAgent(model=model, buyer_max_price=100.0)

   # Multiple sellers with different minimum prices
   seller1 = SellerAgent(model=model, seller_min_price=70.0)
   seller2 = SellerAgent(model=model, seller_min_price=75.0)
   seller3 = SellerAgent(model=model, seller_min_price=65.0)

   env = make(
       "Task2_parallel_three_seller_negotiation-v0",
       buyer_agent=buyer,
       seller_agents=[seller1, seller2, seller3],
       max_rounds=20,
   )

   observation, info = env.reset(
       user_requirement="Looking for the best deal on headphones",
       product_info={
           "name": "Wireless Headphones",
           "price": 120.0,
       },
   )

   done = False
   while not done:
       # Buyer negotiates with all sellers
       buyer_action = buyer.respond(
           observation["conversation_history"],
           observation
       )

       # All sellers respond
       seller_actions = []
       for i, seller in enumerate([seller1, seller2, seller3]):
           action = seller.respond(
               observation["conversation_history"],
               observation
           )
           seller_actions.append(action)

       observation, reward, terminated, truncated, info = env.step(
           buyer_action=buyer_action,
           seller_actions=seller_actions
       )
       done = terminated or truncated

   print(f"Best deal from: Seller {info.get('winning_seller', 'N/A')}")
   print(f"Final Price: ${info.get('seller_price', 'N/A')}")
   env.close()

Sequential Multi-Buyer Negotiation
----------------------------------

Multiple buyers negotiating with a seller one at a time.

.. code-block:: python

   from agenticpay import make
   from agenticpay.agents.buyer_agent import BuyerAgent
   from agenticpay.agents.seller_agent import SellerAgent
   from agenticpay.models.sglang_lm import SGLangLM

   model = SGLangLM(model_path="path/to/model")

   # Multiple buyers with different budgets
   buyer1 = BuyerAgent(model=model, buyer_max_price=120.0)
   buyer2 = BuyerAgent(model=model, buyer_max_price=100.0)

   # One seller
   seller = SellerAgent(model=model, seller_min_price=80.0)

   env = make(
       "Task3_sequential_two_buyer_negotiation-v0",
       buyer_agents=[buyer1, buyer2],
       seller_agent=seller,
       max_rounds=15,
   )

   observation, info = env.reset(
       product_info={
           "name": "Vintage Watch",
           "price": 150.0,
           "condition": "Excellent",
       },
   )

   done = False
   while not done:
       # Current buyer negotiates
       current_buyer = info.get("current_buyer", buyer1)
       buyer_action = current_buyer.respond(
           observation["conversation_history"],
           observation
       )

       seller_action = seller.respond(
           observation["conversation_history"],
           observation
       )

       observation, reward, terminated, truncated, info = env.step(
           buyer_action=buyer_action,
           seller_action=seller_action
       )
       done = terminated or truncated

   print(f"Winning buyer: {info.get('winning_buyer', 'N/A')}")
   env.close()

Using OpenAI API
----------------

Example using OpenAI's GPT models instead of local models.

.. code-block:: python

   import os
   from agenticpay import make
   from agenticpay.agents.buyer_agent import BuyerAgent
   from agenticpay.agents.seller_agent import SellerAgent
   from agenticpay.models.openai_lm import OpenAILLM

   # Set API key
   os.environ["OPENAI_API_KEY"] = "your-api-key"

   # Use GPT-4
   model = OpenAILLM(model_name="gpt-4")

   buyer = BuyerAgent(model=model, buyer_max_price=200.0)
   seller = SellerAgent(model=model, seller_min_price=150.0)

   env = make(
       "Task1_basic_price_negotiation-v0",
       buyer_agent=buyer,
       seller_agent=seller,
       max_rounds=10,
       initial_seller_price=250.0,
       buyer_max_price=200.0,
       seller_min_price=150.0,
   )

   observation, info = env.reset(
       user_requirement="I want to buy a designer bag",
       product_info={
           "name": "Designer Handbag",
           "brand": "Luxury Brand",
           "price": 300.0,
       },
   )

   # Run negotiation...
   done = False
   while not done:
       buyer_action = buyer.respond(
           observation["conversation_history"],
           observation
       )
       seller_action = seller.respond(
           observation["conversation_history"],
           observation
       )

       observation, reward, terminated, truncated, info = env.step(
           buyer_action=buyer_action,
           seller_action=seller_action
       )
       done = terminated or truncated
       env.render()

   env.close()

Custom User Profile
-------------------

Using detailed user profiles to influence negotiation behavior.

.. code-block:: python

   from agenticpay import make
   from agenticpay.agents.buyer_agent import BuyerAgent
   from agenticpay.agents.seller_agent import SellerAgent
   from agenticpay.models.sglang_lm import SGLangLM

   model = SGLangLM(model_path="path/to/model")

   buyer = BuyerAgent(model=model, buyer_max_price=500.0)
   seller = SellerAgent(model=model, seller_min_price=350.0)

   # Detailed user profile
   user_profile = """
   - Professional software engineer, age 35
   - Values quality over price but still budget-conscious
   - Researches products extensively before purchasing
   - Prefers straightforward, no-nonsense negotiations
   - Will mention competitor prices if available
   - Prefers products with good warranty and support
   """

   env = make(
       "Task1_basic_price_negotiation-v0",
       buyer_agent=buyer,
       seller_agent=seller,
       max_rounds=15,
       initial_seller_price=600.0,
       buyer_max_price=500.0,
       seller_min_price=350.0,
   )

   observation, info = env.reset(
       user_requirement="I need a reliable laptop for software development",
       product_info={
           "name": "Developer Laptop Pro",
           "brand": "TechBrand",
           "price": 650.0,
           "features": [
               "32GB RAM",
               "1TB NVMe SSD",
               "Intel i9 Processor",
               "15.6 inch 4K Display",
           ],
           "warranty": "3 years",
       },
       user_profile=user_profile,
   )

   # Run negotiation with personalized buyer behavior...
   done = False
   while not done:
       buyer_action = buyer.respond(
           observation["conversation_history"],
           observation
       )
       seller_action = seller.respond(
           observation["conversation_history"],
           observation
       )

       observation, reward, terminated, truncated, info = env.step(
           buyer_action=buyer_action,
           seller_action=seller_action
       )
       done = terminated or truncated

   env.close()

Registering Custom Environment
------------------------------

Creating and registering a custom negotiation environment.

.. code-block:: python

   from agenticpay.core import BaseEnv
   from agenticpay.envs import register, make

   class AuctionEnv(BaseEnv):
       """Custom auction-style negotiation environment."""

       def __init__(self, buyer_agents, seller_agent, starting_bid, **kwargs):
           super().__init__()
           self.buyer_agents = buyer_agents
           self.seller_agent = seller_agent
           self.starting_bid = starting_bid
           self.current_bid = starting_bid
           self.current_bidder = None

       def reset(self, **kwargs):
           self.current_bid = self.starting_bid
           self.current_bidder = None
           self.round = 0

           observation = {
               "current_bid": self.current_bid,
               "round": self.round,
               "conversation_history": [],
           }
           info = {"status": "ongoing"}
           return observation, info

       def step(self, bids):
           # Find highest bid
           highest_bid = self.current_bid
           winning_bidder = None

           for i, bid in enumerate(bids):
               if bid and bid > highest_bid:
                   highest_bid = bid
                   winning_bidder = i

           self.current_bid = highest_bid
           self.current_bidder = winning_bidder
           self.round += 1

           # Check if auction ends
           terminated = self.round >= 10 or all(b is None for b in bids)

           observation = {
               "current_bid": self.current_bid,
               "round": self.round,
               "conversation_history": [],
           }
           reward = 0.0
           truncated = False
           info = {
               "status": "sold" if terminated else "ongoing",
               "winning_bidder": winning_bidder,
               "final_price": self.current_bid,
           }

           return observation, reward, terminated, truncated, info

       def render(self):
           print(f"Round {self.round}: Current bid ${self.current_bid}")

       def close(self):
           pass

   # Register the custom environment
   register(
       id="AuctionEnv-v0",
       entry_point="__main__:AuctionEnv",
   )

   # Use the custom environment
   env = make(
       "AuctionEnv-v0",
       buyer_agents=[buyer1, buyer2],
       seller_agent=seller,
       starting_bid=100.0,
   )

Available Example Scripts
-------------------------

The repository includes ready-to-run example scripts:

.. code-block:: text

   agenticpay/examples/
   ├── single_buyer_product_seller/
   │   ├── Task1_basic_price_negotiation.py
   │   ├── Task2_close_price_negotiation.py
   │   ├── Task3_close_to_market_price_negotiation.py
   │   └── registration_example.py
   ├── only_multi_products/
   │   └── (multi-product examples)
   ├── only_multi_seller/
   │   └── (multi-seller examples)
   ├── only_multi_buyer/
   │   └── (multi-buyer examples)
   └── multi_buyer_multi_products_multi_seller/
       └── (complex multi-agent examples)

Run any example with:

.. code-block:: console

   python agenticpay/examples/single_buyer_product_seller/Task1_basic_price_negotiation.py
