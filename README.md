# Install TensorFlow on Fedora 33

TensorFlow is a well-known machine learning framework in industry as well as academia that supports both CPU and GPU based computations. 

In the case of only CPU support, all is straightforward; you can easily install TensorFlow by using pip, anaconda, and docker; however, when it comes to GPU support, the story is a bit complicated. 

In the TensorFlow official site, using docker image is suggested as the best way to employ TensorFlow GPU. However, I believe that building it from source is the best choice since the docker image could be failed due to some minor versions unmatched in any possible packages.  

I have struggled for one week to install TensorFlow GPU on Fedora 33; I have tried different possible ways including different versions of nvidia drivers, CUDA, docker images, and even [nvidia TensorFlow images](https://docs.nvidia.com/deeplearning/frameworks/tensorflow-release-notes/overview.html#overview).

Finally, I built and installed **TensorFlow 2.5** on **Fedora 33** with **kernel 5.9.16** as follows, it works as a clock. Hope, this guide could be helpful for you as well. I have collected this  information from different sources (*“TensorFlow Install Source” page in the TensorFlow official website could be a nice source*). Note that you need *root privilege* only for steps 1 to 8.

1. Install compatible **nvidia driver supported CUDA 11.1** by following [here](https://www.if-not-true-then-false.com/2015/fedora-nvidia-guide/)
       
2. Install **CUDA version 11.1** by following [here](https://www.if-not-true-then-false.com/2018/install-nvidia-cuda-toolkit-on-fedora/)
       
3. Download **cuDDN version 8.0.05** (*libcudnn8, libcudnn8-devel, libcudnn8-samples*) that is compatible with CUDA-11.1 from [here](https://developer.nvidia.com/rdp/cudnn-download) (*you need to make a free account*) and install them by `rpm -inv libcudnn8*.rpm` command. Then, verify the installation as follows:
   ```       
   cp -r /usr/src/cudnn_samples_v8/ $HOME
   cd  $HOME/cudnn_samples_v8/mnistCUDNN
   make clean && make
   ./mnistCUDNN
   ```
   If cuDNN is properly installed and running on your Linux system, you will see a long message with **Test passed!** at the end.
           
4. Install python 3.8 (`sudo dnf install python38`)
       
5. Make a **virtual environment** to build and install TensorFlow and active it (Note that you can just use python instead of python3 if your system recognizes python as the default Python3.8 interpreter)
    ```         
    cd ‘your_desire_dir’
    python3 -m venv ‘your_venv_name’
    source ‘your_venv_name’/bin/activate
    ```
  Now, your bash looks like this:  `(your_venv_name) [root@XX XX]$`
           
6. Install *git*  (`sudo dnf install git`)
      
7. Install *perl-core* package (`sudo dnf install perl-core`) 
       
8. Install required prepackes
    ```
     pip install -U pip numpy wheel
     pip install -U keras_preprocessing --no-deps
     ```
     
9. Download *executable version of bazel-3.7.2* for Linux from [here](https://github.com/bazelbuild/bazel/releases/download/3.7.2/bazel-3.7.2-linux-x86_64), rename it to *bazel* and add it to **PATH** (export `PATH=$PATH:’your_dir’/bazel`)
       
10. Download the TensorFlow source code: 
     ```
      git clone https://github.com/tensorflow/tensorflow.git
      cd tensorflow
      ```
      This downloads the most recent version, we can choose other versions as follows:
      ```
      git checkout version_name # r2.2, r2.3, etc.
      ```
      
11. Configure the build process by answering some questions through executing `python configure.py`:

      Note: do not change the default location of Python
      
      Note: Do you wish to build TensorFlow with CUDA support? [y/N]: Y
            
12. Start the build process  (it takes few hours to be completed)
    ``` 
     bazel build --config=cuda --config=v2 --config=mkl --config=monolithic //tensorflow/tools/pip_package:build_pip_package
    ```
13. Make the **whl** file, which will be used to install TensorFlow 2.5
    ``` 
     ./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
    ```
14. Install TensorFlow 2.5
    ``` 
     pip install /tmp/tensorflow_pkg/tensorflow-2.5.0-*.whl
    ```
    
 Enjoy !!



    

