# codegen-quantization-M1
A tutorial to convert to ggml, quantize and run the Salesforce's [CodeGen](https://github.com/salesforce/CodeGen) mono models (Python code generation only) on Mac OS Silicon. All the steps described in this tutorial have been tested on M1 only, but everything should work on M2 too.  
### Prerequisites
This tutorial assumes *Homebrew*, *git*, *conda* and *make* are already installed in the destination machine. If not, please refer to the official documentation of these tools on how to proceed.  
### Environment settings
Install [Git LFS](https://docs.github.com/en/repositories/working-with-files/managing-large-files/installing-git-large-file-storage) and [Boost](https://www.boost.org) using Homebrew:  
  
````brew install git-lfs boost````  
  
Git LFS is a Git extension for versioning large files, which is needed to download the models to convert and quantize. Boost is a collection of portable C++ source libraries needed to perform the models quantization.  
  
Create a Python 3.10 conda virtual environment:  
  
````conda create -n codegenquantenv python=3.10````  
  
activate it:  
  
````conda activate codegenquantenv````  
  
and install the required Python packages (PyTorch, Hugging Face's Transformers and Accelerate):  
  
````
conda install pytorch -c pytorch  
conda install -c conda-forge transformers accelerate
````     
### Source code cloning and build
This tutorial makes use of part of the code in the ravenscroftj's *turbopilot* GitHub repository. First clone that repo and then move to the root folder for the project:  
  
````
git clone https://github.com/ravenscroftj/turbopilot.git
cd ./turbopilot
````
  
This work is built upon *ggml*, a tensor library written in C that provides support for 16-bit float, 4-bit integer quantization, is optimized for Apple Silicon, has no third-party dependencies, allocates zero memory at runtime and allows inference on the CPU.   
Clone the ggml source code, not the original repo, but the branch indicated below, as it contains the specific C++ code for the CodeGen models:  
````git clone https://github.com/ravenscroftj/ggml.git````
  
Build ggml and the codegen and codegen.quantize tools (the first one would be used for inference, the latter to quantize a CodeGen model):  
  
````
cd ./ggml
mkdir build && cd build
cmake ..
make codegen
make codegen.quantize
cd ../..
````  
### Model download
The instructions provided in this paragraph and the next three refer to the CodeGen 2B mono model, but they apply to anyone else belonging to the same family (345M, 2B, 6B, 16B).  
Porting of the CodeGen mono models to GPT-J are already available from moyix in the Hugging Face Hub. So, we are not reinventing the wheel and going to use these as starting point.  
Enable Git LFS first:  
  
````git lfs install````
  
Then, with reference to the 2B mono, downoload it from HF using git:  
  
````
GIT_LFS_SKIP_SMUDGE=1 git clone https://huggingface.co/moyix/codegen-2B-mono-gptj
cd ./codegen-2B-mono-gptj
git config lfs.fetchexclude "*.zst"
git lfs fetch
git lfs checkout *.bin
````
  
You can notice from the list of commands above that we are filtering to pull only the files needed (the .bin PyTorch model weights and the related JSON configuration file) and excluding two unnecessary .tar.zst large files from download. Download time depends on the network: please be patient as the 2B model weights file size is 5.69 GB.    
To be completed.  
### Model conversion
To be written.  
### Model quantization
To be written.  
### Code generation
To be written.  
