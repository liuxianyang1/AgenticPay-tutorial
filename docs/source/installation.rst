Installation
============

This guide will help you install AgenticPay and its dependencies.

Requirements
------------

- Python 3.10 or higher
- Conda (recommended) or pip
- GPU with CUDA support (recommended for local model inference)

Basic Installation
------------------

Using Conda (Recommended)
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: console

   # Create conda environment
   conda create -n agenticpay python=3.10 -y
   conda activate agenticpay

   # Clone the repository
   git clone https://github.com/SafeRL-Lab/AgenticPay.git
   cd AgenticPay

   # Install dependencies
   pip install -r requirements.txt

   # Install package in editable mode
   pip install -e .

Using pip
~~~~~~~~~

.. code-block:: console

   # Create virtual environment
   python -m venv agenticpay-env
   source agenticpay-env/bin/activate  # On Windows: agenticpay-env\Scripts\activate

   # Clone and install
   git clone https://github.com/SafeRL-Lab/AgenticPay.git
   cd AgenticPay
   pip install -r requirements.txt
   pip install -e .

Model Setup
-----------

Local Models
~~~~~~~~~~~~

For local model inference, download models from Hugging Face and save them to the 
``agenticpay/models/download_models`` directory.

.. code-block:: console

   # Example: Download Qwen3-8B-Instruct
   mkdir -p agenticpay/models/download_models
   # Use huggingface-cli or git lfs to download the model

Supported local inference backends:

- **SGLang**: High-performance serving framework
- **vLLM**: Fast LLM inference with PagedAttention

OpenAI API
~~~~~~~~~~

To use OpenAI models, set your API key:

.. code-block:: console

   export OPENAI_API_KEY="your-api-key-here"

Or in Python:

.. code-block:: python

   import os
   os.environ["OPENAI_API_KEY"] = "your-api-key-here"

Verification
------------

Verify your installation by running:

.. code-block:: python

   import agenticpay
   from agenticpay import make
   from agenticpay.agents.buyer_agent import BuyerAgent
   from agenticpay.agents.seller_agent import SellerAgent

   print("AgenticPay installed successfully!")

Dependencies
------------

Core dependencies include:

- ``torch`` - PyTorch for model inference
- ``transformers`` - Hugging Face transformers
- ``gymnasium`` - OpenAI Gymnasium for environment API
- ``openai`` - OpenAI API client (optional)
- ``sglang`` - SGLang inference backend (optional)
- ``vllm`` - vLLM inference backend (optional)

For a complete list, see ``requirements.txt`` in the repository.
