# codegen-quantization-M1
A tutorial to convert to ggml, quantize and run the Salesforce's [CodeGen](https://github.com/salesforce/CodeGen) mono models (Python code generation only) on Mac OS Silicon CPU. All the steps described in this tutorial have been tested on M1 only, but everything should work on M2 too.  
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
Porting of the CodeGen mono models to GPT-J are already available from *moyix* in the Hugging Face Hub. So, we are not reinventing the wheel and going to use these as starting point.  
Enable Git LFS first:  
  
````git lfs install````
  
Then, with reference to the 2B mono, download it from HF using git:  
  
````
GIT_LFS_SKIP_SMUDGE=1 git clone https://huggingface.co/moyix/codegen-2B-mono-gptj
cd ./codegen-2B-mono-gptj
git config lfs.fetchexclude "*.zst"
git lfs fetch
git lfs checkout *.bin
````
  
You can notice from the list of commands above that we are filtering to pull only the files needed (the .bin PyTorch model weights and the related JSON configuration file) and excluding two unnecessary .tar.zst large files from download. Download time depends on the network: please be patient as the 2B model weights file size is 5.69 GB.    
We ar not done yet, as we need to add the *vocab* and *added_token* files from the original CodeGen model. They can be downloaded from the *Salesforce*'s Hugging Face space and placed in the same directory as for the ported model (using wget in this tutorial, but you can use a different tool or download them manually):  
  
````
wget https://huggingface.co/Salesforce/codegen-2B-mono/raw/main/vocab.json
wget https://huggingface.co/Salesforce/codegen-2B-mono/raw/main/added_tokens.json
````
### Model conversion
A Python script to convert the GPT-J CodeGen model to ggml is provided as part of the turbopilot repo. Move back to the parent directory of the one where the model has been downloaded and then execute the script:  
  
````
cd ..
python ./convert-codegen-to-ggml.py ./codegen-2B-mono-gptj 0
````
  
The scripts expects 2 mandatory arguments: the first one is the path of the directory where the original model has been downloaded, the second one indicates the floating point type (0 = float32, 1 = float16). The converted model is saved in the same directory as for the original model and would be named as *ggml-model-f32.bin* (or *ggml-model-f16.bin*, depending on the selected floating point type).  
### Model quantization
Quantization of the model can be done through the *codegen-quantize* tool that has already been built from C++ source code in one of the previous steps. Here is how you can use it:  
  
````./ggml/build/bin/codegen-quantize ./codegen-2B-mono-gptj/ggml-model-f32.bin ./codegen-2B-mono-gptj/ggml-model-quant.bin 2````
  
This tool expects three arguments: relative path and name of the ggml converted model, destination path and name of the quantized model, the 4-bit quantization strategy (leave it to 2).  
### Code generation
To start using the model to generate Python code from natural language prompts you can use the *codegen* tool that has already been built from C++ source code in one of the previous steps. Here is how you can do it:     
````
./ggml/build/bin/codegen -t 10 -m ./codegen-2B-mono-gptj/ggml-model-quant.bin -p 'import os

                                          import json
                                          def main():

                                             """this is the main function that opens the file and loads the json data"""'
````
    
where:  
* -t is the temperature;
* -m is the path of the quantized model;
* -p is the text prompt.  
## Models
CodeGen mono models quantized through this procedure will be made available publicy in the Hugging Face Hub. Here are their links:  
  
:hugs: [CodeGen 350M mono](https://huggingface.co/Guglielmo/CodeGen-350M-mono-ggml-quant)  
:hugs: [CodeGen 2B mono](https://huggingface.co/Guglielmo/CodeGen-2B-mono-ggml-quant)  
:hugs: [CodeGen 6B mono](https://huggingface.co/Guglielmo/CodeGen-6B-mono-ggml-quant)  
