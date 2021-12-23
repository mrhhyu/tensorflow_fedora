# Install TensorFlow on Fedora

TensorFlow is a well-known and open source machine learning framework in industry as well as academia that supports both CPU and GPU based computations. 

In the case of only CPU support, all is straightforward; you can easily install TensorFlow by using pip, anaconda, and docker; however, when it comes to GPU support, the story is a bit complicated. 

In the TensorFlow official website, using docker image is suggested as the best way to employ TensorFlow GPU. However, I believe that building it from source is the best choice since the docker image could be failed due to some minor versions unmatched in any possible packages.  

I have struggled fo
r a week to install TensorFlow GPU on Fedora 33; I have tried different possible methods including various versions of nvidia drivers, CUDA installers, docker images, and even [nvidia TensorFlow images](https://docs.nvidia.com/deeplearning/frameworks/tensorflow-release-notes/overview.html#overview).

Finally, I could successfully build and install **TensorFlow 2.5** (i.e., both GPU and CPU) on **Fedora 33** with **kernel 5.9.16** as follows. Hope, this post could be helpful for you as well. 

**Instructions on TensorFlow 2.7 (TensorRT support) and Fedora 35 is located under Release 1.**

### Important notes to install TensorFlow-GPU: ###

1. The most important steps are 1, 2, and 3; after each step, follow the instructions to assure that packages are installed in a proper way.
2. ***The following steps can be applied when the versions of your kernel, cuda, and the nvidia driver are different with the ones used here***; however, **make sure** that both the *nvidia driver* and *cuDDN package* support the **actuall version of your CUDA package**.

## Install TensorFlow-GPU on Fedora 33

You need *root privilege* only for steps 1 to 8.

1. Install compatible **nvidia driver supported CUDA 11.1** by following [here](https://www.if-not-true-then-false.com/2015/fedora-nvidia-guide/)
       
2. Install **CUDA Toolkit version 11.1** by following [here](https://www.if-not-true-then-false.com/2018/install-nvidia-cuda-toolkit-on-fedora/)
       
3. Download **cuDDN version 8.0.05** (*libcudnn8, libcudnn8-devel, libcudnn8-samples*) that is compatible with CUDA-11.1 from [here](https://developer.nvidia.com/rdp/cudnn-download) (*you need to make a free account*) and install them by `rpm -Uvh libcudnn8*.rpm` command. Then, verify the installation as follows (note that "freeimage" and "freeimage-dev" pachakges must be installed to make the cuDNN samples):
   ```       
   cp -r /usr/src/cudnn_samples_v8/ $HOME
   cd  $HOME/cudnn_samples_v8/mnistCUDNN
   make clean && make
   ./mnistCUDNN
   ```
   If cuDNN is properly installed and running on your Linux system, you will see a long message with **Test passed!** at the end.
           
4. Install python 3.8 (`sudo dnf install python38`)
       
5. Make a **virtual environment** to build and install TensorFlow and active it (Note that you can just use python instead of python3 if your system recognizes python as the default Python3.8 interpreter; you can also make the virtual environment under user not root)
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
      
      Note: Do you wish to build TensorFlow with ROCm support? [y/N]: n
      
      Note: Do you wish to build TensorFlow with CUDA support? [y/N]: y
      
      Note: Do you want to use clang as CUDA compiler? [y/N]: n
      
      Note: Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -Wno-sign-compare]:-march=native
            
12. Start the build process  (it takes few hours to be completed) {--config=v2 to build tensorflow 2.x}
    ``` 
     bazel build --config=cuda --config=v2 --config=mkl --config=monolithic //tensorflow/tools/pip_package:build_pip_package
    ```
13. Make the **whl** file, which will be used to install TensorFlow 2.5
    ``` 
     ./bazel-bin/tensorflow/tools/pip_package/build_pip_package 'your_desire_dir'/tensorflow_pkg
    ```
14. Install TensorFlow 2.5
    ``` 
     pip install 'your_desire_dir'/tensorflow_pkg/tensorflow-2.5.0-*.whl
    ```
15. Save the following python code in test.py and execute it (`python test.py`)
    ``` 
    import tensorflow as tf
    from tensorflow.python.client import device_lib
    print("TF Version: "+tf.__version__)
    print()
    print(device_lib.list_local_devices())
    ``` 
    You must see "TF Version:2.5" and a long bash output containig something like follows:
    ```
    Created TensorFlow device (/device:GPU:0 with 22434 MB memory) -> physical GPU (device: 0, name: TITAN RTX, pci bus id: 0000:01:00.0, compute capability: 7.5)
    [name: "/device:CPU:0"
    device_type: "CPU"
    memory_limit: 268435456
    locality {
    }
    incarnation: 17126562804018110364
    , name: "/device:GPU:0"
    device_type: "GPU"
    memory_limit: 23523885056
    locality {
      bus_id: 1
      links {
      }
    }
    incarnation: 17839688261725336163
    physical_device_desc: "device: 0, name: TITAN RTX, pci bus id: 0000:01:00.0, compute capability: 7.5"
    ]
    ```

## Install TensorFlow-CPU on Fedora 33

The process is very similar to the TensorFlow GPU support with few following differences:

1. Steps 1, 2, and 3 are unnecessary, of course. 

2. Follow steps 4~10.

3. Configure the build process by answering some questions through executing `python configure.py`:

      Note: do not change the default location of Python
      
      Note: Do you wish to build TensorFlow with CUDA support? [y/N]: n
      
      Note: Do you want to use clang as CUDA compiler? [y/N]: n
      
      Note: Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -Wno-sign-compare]:-march=native
  
4. Start the build process  (it takes few hours to be completed) {--config=v2 to build tensorflow 2.x}
    ``` 
     bazel build --config=v2 --config=mkl --config=monolithic //tensorflow/tools/pip_package:build_pip_package
    ```
5. Make the **whl** file, which will be used to install TensorFlow 2.5
    ``` 
     ./bazel-bin/tensorflow/tools/pip_package/build_pip_package 'your_desire_dir'/tensorflow_pkg
    ```
6. Install TensorFlow 2.5; **do not** intsall it in the same virtual environment as TensorFlow GPU.
    ``` 
     pip install 'your_desire_dir'/tensorflow_pkg/tensorflow-2.5.0-*.whl
    ```
7. Save the following python code in test.py and execute it (`python test.py`)
    ``` 
    import tensorflow as tf
    from tensorflow.python.client import device_lib
    print("TF Version: "+tf.__version__)
    print()
    print(device_lib.list_local_devices())
    ``` 
    You must see "TF Version:2.5" and a long bash output containig something like follows:
    ```
    [name: "/device:CPU:0"
    device_type: "CPU"
    memory_limit: 268435456
    locality {
    }
    incarnation: 17126562804018110364
    ]
    ```
    
## Simple check 

Save the following python code in check.py and execute it (`python check.py`)
```
    import tensorflow as tf
    import time
    
    start = time.perf_counter()
    with tf.device('/CPU:0'):
        for i in range (1,1000):
            if i%100==0:
                print (i)        
            for j  in range (1,1000):
                tf.reduce_sum(tf.random.normal([1000, 1000]))
    stop = time.perf_counter()
    time_passed = stop-start
    print("time with CPU: "+str(round((time_passed/60),3)))
    print ()
    
    start = time.perf_counter()
    with tf.device('/GPU:0'):
        for i in range (1,1000):
            if i%100==0:
                print (i)        
            for j  in range (1,1000):
                tf.reduce_sum(tf.random.normal([1000, 1000]))
    stop = time.perf_counter()
    time_passed = stop-start
    print("time with GPU: "+str(round((time_passed/60),3)))
```
In my machine with Intel i9-9900K CPU (3.60GHz), 128 GB RAM, and TITAN RTX GPU, "time with CPU: 25.827" and "time with GPU: 2.054". 

## When Kernel update affects the driver 

During installation process of a new kernel, DKMS is uninstalled; then, it is built and installed again. However, when the installed nvidia driver is **not compatible** with the new kernel, DKMS is uninstalled and will not be installed again; you cannot login via new installed kernel. 
In this case, 
1. Login via an already installed old kernel
2. Downlaod the nvidia driver compatible with your existing CUDA and the new kernel; make in exectable
3. Uninstall current nvidia driver by "nvidia-installer --uninstall"
4. reboot and login in run-level 3 (text-mode with network) via new kernel
5. Install the new downlowded nvidia driver

Now, TensorFlow-gpu is usable and no extra setting is required.  



ENJOY!

