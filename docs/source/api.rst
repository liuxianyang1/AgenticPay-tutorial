API Reference
=============

This section provides the complete API reference for AgenticPay.

Core Functions
--------------

make
~~~~

.. code-block:: python

   agenticpay.make(id, **kwargs)

Create an environment instance by its registered ID.

**Parameters:**

- ``id`` (str): Environment identifier (e.g., ``"Task1_basic_price_negotiation-v0"``)
- ``**kwargs``: Environment-specific parameters

**Returns:**

- Environment instance

**Example:**

.. code-block:: python

   from agenticpay import make

   env = make(
       "Task1_basic_price_negotiation-v0",
       buyer_agent=buyer,
       seller_agent=seller,
       max_rounds=20,
   )

register
~~~~~~~~

.. code-block:: python

   agenticpay.envs.register(id, entry_point, **kwargs)

Register a new environment.

**Parameters:**

- ``id`` (str): Unique environment identifier
- ``entry_point`` (str): Module path to the environment class
- ``max_episode_steps`` (int, optional): Maximum steps per episode

**Example:**

.. code-block:: python

   from agenticpay.envs import register

   register(
       id="MyEnv-v0",
       entry_point="my_module:MyEnvClass",
       max_episode_steps=100,
   )

pprint_registry
~~~~~~~~~~~~~~~

.. code-block:: python

   agenticpay.envs.pprint_registry()

Print all registered environments.

Environment API
---------------

BaseEnv
~~~~~~~

Base class for all environments.

**Methods:**

.. code-block:: python

   class BaseEnv:
       def reset(self, **kwargs) -> Tuple[dict, dict]:
           """Reset environment and return initial observation."""
           pass

       def step(self, **actions) -> Tuple[dict, float, bool, bool, dict]:
           """Execute one step and return (observation, reward, terminated, truncated, info)."""
           pass

       def render(self) -> None:
           """Display current state."""
           pass

       def close(self) -> None:
           """Clean up resources."""
           pass

reset()
^^^^^^^

Initialize a new negotiation episode.

**Parameters:**

- ``user_requirement`` (str): User's product requirement
- ``product_info`` (dict): Product details
- ``user_profile`` (str, optional): User personality description

**Returns:**

- ``observation`` (dict): Initial state
- ``info`` (dict): Additional information

step()
^^^^^^

Execute one negotiation round.

**Parameters:**

- ``buyer_action`` (str): Buyer's response
- ``seller_action`` (str): Seller's response

**Returns:**

- ``observation`` (dict): Updated state
- ``reward`` (float): Reward value
- ``terminated`` (bool): Episode ended naturally
- ``truncated`` (bool): Episode ended due to limit
- ``info`` (dict): Additional information

Agent API
---------

BaseAgent
~~~~~~~~~

.. code-block:: python

   class BaseAgent:
       def __init__(self, model):
           """Initialize agent with LLM model."""
           pass

       def respond(self, conversation_history, current_state) -> str:
           """Generate response based on context."""
           pass

BuyerAgent
~~~~~~~~~~

.. code-block:: python

   class BuyerAgent(BaseAgent):
       def __init__(self, model, buyer_max_price):
           """
           Initialize buyer agent.

           Args:
               model: LLM model instance
               buyer_max_price: Maximum acceptable price
           """
           pass

SellerAgent
~~~~~~~~~~~

.. code-block:: python

   class SellerAgent(BaseAgent):
       def __init__(self, model, seller_min_price):
           """
           Initialize seller agent.

           Args:
               model: LLM model instance
               seller_min_price: Minimum acceptable price
           """
           pass

Model API
---------

SGLangLM
~~~~~~~~

.. code-block:: python

   class SGLangLM:
       def __init__(self, model_path, **kwargs):
           """
           Initialize SGLang model.

           Args:
               model_path: Path to model weights
           """
           pass

       def generate(self, prompt, **kwargs) -> str:
           """Generate text from prompt."""
           pass

VLLMLM
~~~~~~

.. code-block:: python

   class VLLMLM:
       def __init__(
           self,
           model_path,
           trust_remote_code=True,
           gpu_memory_utilization=0.9,
           tensor_parallel_size=1,
           **kwargs
       ):
           """
           Initialize vLLM model.

           Args:
               model_path: Path to model weights
               trust_remote_code: Allow custom model code
               gpu_memory_utilization: GPU memory fraction (0.0-1.0)
               tensor_parallel_size: Number of GPUs for parallelism
           """
           pass

OpenAILLM
~~~~~~~~~

.. code-block:: python

   class OpenAILLM:
       def __init__(
           self,
           model_name="gpt-4",
           api_key=None,
           temperature=0.7,
           max_tokens=1024,
           **kwargs
       ):
           """
           Initialize OpenAI model.

           Args:
               model_name: OpenAI model name
               api_key: API key (or use OPENAI_API_KEY env var)
               temperature: Response randomness
               max_tokens: Maximum response length
           """
           pass

HuggingFaceLLM
~~~~~~~~~~~~~~

.. code-block:: python

   class HuggingFaceLLM:
       def __init__(self, model_name, device="cuda", **kwargs):
           """
           Initialize HuggingFace model.

           Args:
               model_name: HuggingFace model identifier
               device: Device to run on ("cuda" or "cpu")
           """
           pass

Memory API
----------

ConversationMemory
~~~~~~~~~~~~~~~~~~

.. code-block:: python

   class ConversationMemory:
       def __init__(self):
           """Initialize empty conversation memory."""
           pass

       def add_message(self, role, content, **metadata):
           """Add a message to history."""
           pass

       def get_history(self) -> list:
           """Get full conversation history."""
           pass

       def get_recent(self, n) -> list:
           """Get last n messages."""
           pass

       def clear(self):
           """Clear all messages."""
           pass

Data Structures
---------------

Observation Dictionary
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   observation = {
       "conversation_history": [
           {"role": "buyer", "content": "...", "round": 1},
           {"role": "seller", "content": "...", "round": 1},
       ],
       "current_round": 1,
       "seller_price": 150.0,
       "buyer_offer": 100.0,
       "product_info": {...},
       "environment_info": {...},
   }

Info Dictionary
~~~~~~~~~~~~~~~

.. code-block:: python

   info = {
       "status": "deal_reached",  # or "no_deal", "buyer_rejected", etc.
       "seller_price": 120.0,
       "buyer_savings": 30.0,
       "seller_profit": 40.0,
       "rounds_used": 5,
   }

Product Info Dictionary
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   product_info = {
       "name": "Product Name",
       "brand": "Brand Name",
       "price": 100.0,
       "features": ["feature1", "feature2"],
       "condition": "New",
       "material": "Material",
       "warranty": "1 year",
   }

Reward Weights Dictionary
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   reward_weights = {
       "buyer_savings": 1.0,
       "seller_profit": 1.0,
       "time_cost": 0.1,
   }
