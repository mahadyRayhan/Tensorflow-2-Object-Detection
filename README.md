# Training Tensorflow Objecte Detection

The most convenient way to train a TensorFlow object detection model is to use verified Tensorflow models architectures provided by TensorFlow. you can find the GitHub repo at this link [TensorFlow official](https://github.com/tensorflow/models).

In this section, I train an object detection model (EfficientDet D3) in a virtual environment. The reason for using a virtual environment is to make the whole process separate so that it won't affect other model implements or environment setups.

I use an anaconda virtual environment to train this model. You can use any kind of virtual environment setup to train the model. the benefit of using an anaconda environment is, when I install TensorFlow GPU using conda commands, it automatically install compatible Cuda and Cudnn libraries. if you use other environments, you may or may not have to set up these two necessary libraries.

### Install anaconda virtual environment

installing anaconda is fairly easy. Open a terminal copy-paste the following commands.

<pre>
sudo apt install libgl1-mesa-glx libegl1-mesa libxrandr2 libxrandr2 libxss1 libxcursor1 libxcomposite1 libasound2 libxi6 libxtst6

wget -P /tmp https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh

bash /tmp/Anaconda3-2020.02-Linux-x86_64.sh

source ~/.bashrc
</pre>

This process will ask for license approval. just approve the license and you are ready to go.

After installing an anaconda-navigator, close the terminal and open a new one. Now create a new anaconda virtual environment. Simple type -
<pre>conda create -n tensorflow pip python=3.6</pre>
where "tensorflow" is the name of your environment. You can choose any python version. I use python 3.6 as I know this version is one of the most stable versions.

After creating the virtual environment activate the environment. Simply type **source activate tensorflow**. The terminal will show you this command once the environment installation is successfully finished.

Next, install TensorFlow-GPU (as we will use GPU to train) on our system. simply type -

<pre>conda install -c anaconda tensorflow-gpu</pre>

This will install the TensorFlow-GPU version to our system as well as compatible Cuda and Cudnn in this virtual environment.

That's all for an anaconda virtual environment setup.<br>
**Note: all these commands should be run in a terminal, not in the Jupyter notebook.**

## Install TensorFlow Object Detection API

Now open a jupyter notebook for the next installation process that is TensorFlow Object Detection API. Simply type -
<pre> jupyter notebook</pre> to open a jupyter notebook.

Now you may face some errors as jupyter notebook is not always installed with the new environment setup. If this happens for you simply type - <pre> pip install notebook </pre> This will install a new jupyter notebook for this virtual environment.<br>

Now open jupyter notebook.

First check the installed version of Tensorflow. I installed Tensorflow 2.4.1. Your version might be different.


```python
import tensorflow as tf
tf.__version__
```

    2022-01-26 13:49:37.407858: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library libcudart.so.10.1





    '2.4.1'



### Downloading the TensorFlow Model Garden

First create a folder where you like to store all your files(pre-trained model, trained model, data, etc) related to object detection. then go to that directory and execute the following command.


```python
pwd
```




    '/home/ubuntu-20/Desktop/TensorFlow'




```python
!git clone https://github.com/tensorflow/models.git
```

    Cloning into 'models'...
    remote: Enumerating objects: 68544, done.[K
    remote: Counting objects: 100% (10/10), done.[K
    remote: Compressing objects: 100% (8/8), done.[K
    remote: Total 68544 (delta 2), reused 9 (delta 2), pack-reused 68534[K
    Receiving objects: 100% (68544/68544), 576.95 MiB | 14.26 MiB/s, done.
    Resolving deltas: 100% (48277/48277), done.


This will download Tensorflow model garden in your folder. I named my model as **"TensorFlow"**. You can name anything you want.

### Installat Protobuf 

Tensorflow Object Detection API uses Protobufs to configure model and training parameters. This library must be installed inside the **"research"** folder, which is inside the **"models"** folder. BTW, the **"models"** folder is the Tensorflow model garden repo which we pulled earlier.


```python
cd models/research/
```

    /home/ubuntu-20/Desktop/TensorFlow/models/research



```python
pwd
```




    '/home/ubuntu-20/Desktop/TensorFlow/models/research'



Now execute the Protobuf installation command here.


```python
!protoc object_detection/protos/*.proto --python_out=.
```

### Install COCO API

"pycocotools" is another dependency needed for Tensorflow object detection API. You can simply pull the library from GitHub and install/make it. But before doing that, you need "cython" library to be installed. Install "cython" as follows -


```python
!pip install cython
```

    Collecting cython
      Downloading Cython-0.29.26-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_24_x86_64.whl (1.9 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 1.9 MB 7.6 MB/s eta 0:00:01
    [?25hInstalling collected packages: cython
    Successfully installed cython-0.29.26


Next clone "pycocotools" repo from GitHub and execute the following commands one by one.


```python
!git clone https://github.com/cocodataset/cocoapi.git
```

    Cloning into 'cocoapi'...
    remote: Enumerating objects: 975, done.[K
    remote: Total 975 (delta 0), reused 0 (delta 0), pack-reused 975[K
    Receiving objects: 100% (975/975), 11.72 MiB | 13.58 MiB/s, done.
    Resolving deltas: 100% (576/576), done.



```python
cd cocoapi/PythonAPI
```

    /home/ubuntu-20/Desktop/TensorFlow/models/research/cocoapi/PythonAPI



```python
!make
```

    python setup.py build_ext --inplace
    running build_ext
    cythoning pycocotools/_mask.pyx to pycocotools/_mask.c
    /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/Cython/Compiler/Main.py:369: FutureWarning: Cython directive 'language_level' not set, using 2 for now (Py2). This will change in a later release! File: /home/ubuntu-20/Desktop/TensorFlow/models/research/cocoapi/PythonAPI/pycocotools/_mask.pyx
      tree = Parsing.p_module(s, pxd, full_module_name)
    building 'pycocotools._mask' extension
    gcc -pthread -B /home/ubuntu-20/anaconda3/envs/tensorflow/compiler_compat -Wno-unused-result -Wsign-compare -DNDEBUG -O2 -Wall -fPIC -O2 -isystem /home/ubuntu-20/anaconda3/envs/tensorflow/include -I/home/ubuntu-20/anaconda3/envs/tensorflow/include -fPIC -O2 -isystem /home/ubuntu-20/anaconda3/envs/tensorflow/include -fPIC -I/home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/numpy/core/include -I../common -I/home/ubuntu-20/anaconda3/envs/tensorflow/include/python3.9 -c ../common/maskApi.c -o build/temp.linux-x86_64-3.9/../common/maskApi.o -Wno-cpp -Wno-unused-function -std=c99
    [01m[K../common/maskApi.c:[m[K In function ???[01m[KrleDecode[m[K???:
    [01m[K../common/maskApi.c:46:7:[m[K [01;35m[Kwarning: [m[Kthis ???[01m[Kfor[m[K??? clause does not guard... [[01;35m[K-Wmisleading-indentation[m[K]
       46 |       [01;35m[Kfor[m[K( k=0; k<R[i].cnts[j]; k++ ) *(M++)=v; v=!v; }}
          |       [01;35m[K^~~[m[K
    [01m[K../common/maskApi.c:46:49:[m[K [01;36m[Knote: [m[K...this statement, but the latter is misleadingly indented as if it were guarded by the ???[01m[Kfor[m[K???
       46 |       for( k=0; k<R[i].cnts[j]; k++ ) *(M++)=v; [01;36m[Kv[m[K=!v; }}
          |                                                 [01;36m[K^[m[K
    [01m[K../common/maskApi.c:[m[K In function ???[01m[KrleFrPoly[m[K???:
    [01m[K../common/maskApi.c:166:3:[m[K [01;35m[Kwarning: [m[Kthis ???[01m[Kfor[m[K??? clause does not guard... [[01;35m[K-Wmisleading-indentation[m[K]
      166 |   [01;35m[Kfor[m[K(j=0; j<k; j++) x[j]=(int)(scale*xy[j*2+0]+.5); x[k]=x[0];
          |   [01;35m[K^~~[m[K
    [01m[K../common/maskApi.c:166:54:[m[K [01;36m[Knote: [m[K...this statement, but the latter is misleadingly indented as if it were guarded by the ???[01m[Kfor[m[K???
      166 |   for(j=0; j<k; j++) x[j]=(int)(scale*xy[j*2+0]+.5); [01;36m[Kx[m[K[k]=x[0];
          |                                                      [01;36m[K^[m[K
    [01m[K../common/maskApi.c:167:3:[m[K [01;35m[Kwarning: [m[Kthis ???[01m[Kfor[m[K??? clause does not guard... [[01;35m[K-Wmisleading-indentation[m[K]
      167 |   [01;35m[Kfor[m[K(j=0; j<k; j++) y[j]=(int)(scale*xy[j*2+1]+.5); y[k]=y[0];
          |   [01;35m[K^~~[m[K
    [01m[K../common/maskApi.c:167:54:[m[K [01;36m[Knote: [m[K...this statement, but the latter is misleadingly indented as if it were guarded by the ???[01m[Kfor[m[K???
      167 |   for(j=0; j<k; j++) y[j]=(int)(scale*xy[j*2+1]+.5); [01;36m[Ky[m[K[k]=y[0];
          |                                                      [01;36m[K^[m[K
    [01m[K../common/maskApi.c:[m[K In function ???[01m[KrleToString[m[K???:
    [01m[K../common/maskApi.c:212:7:[m[K [01;35m[Kwarning: [m[Kthis ???[01m[Kif[m[K??? clause does not guard... [[01;35m[K-Wmisleading-indentation[m[K]
      212 |       [01;35m[Kif[m[K(more) c |= 0x20; c+=48; s[p++]=c;
          |       [01;35m[K^~[m[K
    [01m[K../common/maskApi.c:212:27:[m[K [01;36m[Knote: [m[K...this statement, but the latter is misleadingly indented as if it were guarded by the ???[01m[Kif[m[K???
      212 |       if(more) c |= 0x20; [01;36m[Kc[m[K+=48; s[p++]=c;
          |                           [01;36m[K^[m[K
    [01m[K../common/maskApi.c:[m[K In function ???[01m[KrleFrString[m[K???:
    [01m[K../common/maskApi.c:220:3:[m[K [01;35m[Kwarning: [m[Kthis ???[01m[Kwhile[m[K??? clause does not guard... [[01;35m[K-Wmisleading-indentation[m[K]
      220 |   [01;35m[Kwhile[m[K( s[m] ) m++; cnts=malloc(sizeof(uint)*m); m=0;
          |   [01;35m[K^~~~~[m[K
    [01m[K../common/maskApi.c:220:22:[m[K [01;36m[Knote: [m[K...this statement, but the latter is misleadingly indented as if it were guarded by the ???[01m[Kwhile[m[K???
      220 |   while( s[m] ) m++; [01;36m[Kcnts[m[K=malloc(sizeof(uint)*m); m=0;
          |                      [01;36m[K^~~~[m[K
    [01m[K../common/maskApi.c:228:5:[m[K [01;35m[Kwarning: [m[Kthis ???[01m[Kif[m[K??? clause does not guard... [[01;35m[K-Wmisleading-indentation[m[K]
      228 |     [01;35m[Kif[m[K(m>2) x+=(long) cnts[m-2]; cnts[m++]=(uint) x;
          |     [01;35m[K^~[m[K
    [01m[K../common/maskApi.c:228:34:[m[K [01;36m[Knote: [m[K...this statement, but the latter is misleadingly indented as if it were guarded by the ???[01m[Kif[m[K???
      228 |     if(m>2) x+=(long) cnts[m-2]; [01;36m[Kcnts[m[K[m++]=(uint) x;
          |                                  [01;36m[K^~~~[m[K
    gcc -pthread -B /home/ubuntu-20/anaconda3/envs/tensorflow/compiler_compat -Wno-unused-result -Wsign-compare -DNDEBUG -O2 -Wall -fPIC -O2 -isystem /home/ubuntu-20/anaconda3/envs/tensorflow/include -I/home/ubuntu-20/anaconda3/envs/tensorflow/include -fPIC -O2 -isystem /home/ubuntu-20/anaconda3/envs/tensorflow/include -fPIC -I/home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/numpy/core/include -I../common -I/home/ubuntu-20/anaconda3/envs/tensorflow/include/python3.9 -c pycocotools/_mask.c -o build/temp.linux-x86_64-3.9/pycocotools/_mask.o -Wno-cpp -Wno-unused-function -std=c99
    creating build/lib.linux-x86_64-3.9
    creating build/lib.linux-x86_64-3.9/pycocotools
    gcc -pthread -B /home/ubuntu-20/anaconda3/envs/tensorflow/compiler_compat -shared -Wl,-rpath,/home/ubuntu-20/anaconda3/envs/tensorflow/lib -Wl,-rpath-link,/home/ubuntu-20/anaconda3/envs/tensorflow/lib -L/home/ubuntu-20/anaconda3/envs/tensorflow/lib -L/home/ubuntu-20/anaconda3/envs/tensorflow/lib -Wl,-rpath,/home/ubuntu-20/anaconda3/envs/tensorflow/lib -Wl,-rpath-link,/home/ubuntu-20/anaconda3/envs/tensorflow/lib -L/home/ubuntu-20/anaconda3/envs/tensorflow/lib build/temp.linux-x86_64-3.9/../common/maskApi.o build/temp.linux-x86_64-3.9/pycocotools/_mask.o -o build/lib.linux-x86_64-3.9/pycocotools/_mask.cpython-39-x86_64-linux-gnu.so
    copying build/lib.linux-x86_64-3.9/pycocotools/_mask.cpython-39-x86_64-linux-gnu.so -> pycocotools
    rm -rf build



```python
cp -r pycocotools /home/ubuntu-20/Desktop/TensorFlow/models/research
```

So,"pycocotools" library is installed successfully.

### Install the Object Detection API

Now all the dependencies necessary for installing Object Detection API are done. So, we can install Object Detection API now. Object Detection API should be installed inside the **"research"** directory. Check the present current directory and go to the **"research"** directory


```python
pwd
```




    '/home/ubuntu-20/Desktop/TensorFlow/models/research/cocoapi/PythonAPI'




```python
cd ..
```

    /home/ubuntu-20/Desktop/TensorFlow/models/research/cocoapi



```python
cd ..
```

    /home/ubuntu-20/Desktop/TensorFlow/models/research



```python
pwd
```




    '/home/ubuntu-20/Desktop/TensorFlow/models/research'



Next run the following two commands to install **"Object Detection API"**


```python
cp object_detection/packages/tf2/setup.py .
```


```python
!python -m pip install .
```

    Processing /home/ubuntu-20/Desktop/TensorFlow/models/research
    [33m  DEPRECATION: A future pip version will change local packages to be built in-place without first copying to a temporary directory. We recommend you use --use-feature=in-tree-build to test your packages with this new behavior before it becomes the default.
       pip 21.3 will remove support for this functionality. You can find discussion regarding this at https://github.com/pypa/pip/issues/7555.[0m
    Collecting avro-python3
      Downloading avro-python3-1.10.2.tar.gz (38 kB)
    Collecting apache-beam
      Downloading apache-beam-2.35.0.zip (2.6 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 2.6 MB 10.0 MB/s eta 0:00:01
    [?25hCollecting pillow
      Downloading Pillow-9.0.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (4.3 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 4.3 MB 27.3 MB/s eta 0:00:01
    [?25hCollecting lxml
      Downloading lxml-4.7.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_24_x86_64.whl (6.9 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 6.9 MB 20.1 MB/s eta 0:00:01
    [?25hCollecting matplotlib
      Downloading matplotlib-3.5.1-cp39-cp39-manylinux_2_5_x86_64.manylinux1_x86_64.whl (11.2 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 11.2 MB 31.2 MB/s eta 0:00:01
    [?25hRequirement already satisfied: Cython in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from object-detection==0.1) (0.29.26)
    Collecting contextlib2
      Downloading contextlib2-21.6.0-py2.py3-none-any.whl (13 kB)
    Collecting tf-slim
      Downloading tf_slim-1.1.0-py2.py3-none-any.whl (352 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 352 kB 31.6 MB/s eta 0:00:01
    [?25hRequirement already satisfied: six in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from object-detection==0.1) (1.15.0)
    Collecting pycocotools
      Downloading pycocotools-2.0.4.tar.gz (106 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 106 kB 54.9 MB/s eta 0:00:01
    [?25h  Installing build dependencies ... [?25ldone
    [?25h  Getting requirements to build wheel ... [?25ldone
    [?25h    Preparing wheel metadata ... [?25ldone
    [?25hCollecting lvis
      Downloading lvis-0.5.3-py3-none-any.whl (14 kB)
    Requirement already satisfied: scipy in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from object-detection==0.1) (1.6.2)
    Collecting pandas
      Downloading pandas-1.4.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (11.7 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 11.7 MB 16.0 MB/s eta 0:00:01
    [?25hCollecting tf-models-official>=2.5.1
      Downloading tf_models_official-2.7.0-py2.py3-none-any.whl (1.8 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 1.8 MB 31.7 MB/s eta 0:00:01
    [?25hCollecting tensorflow_io
      Downloading tensorflow_io-0.23.1-cp39-cp39-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (23.1 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 23.1 MB 24.7 MB/s eta 0:00:01
    [?25hCollecting keras
      Downloading keras-2.7.0-py2.py3-none-any.whl (1.3 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 1.3 MB 25.4 MB/s eta 0:00:01
    [?25hCollecting tensorflow-hub>=0.6.0
      Downloading tensorflow_hub-0.12.0-py2.py3-none-any.whl (108 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 108 kB 41.8 MB/s eta 0:00:01
    [?25hCollecting oauth2client
      Downloading oauth2client-4.1.3-py2.py3-none-any.whl (98 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 98 kB 5.0 MB/s  eta 0:00:01
    [?25hCollecting seqeval
      Downloading seqeval-1.2.2.tar.gz (43 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 43 kB 1.7 MB/s  eta 0:00:01
    [?25hCollecting sentencepiece
      Downloading sentencepiece-0.1.96-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (1.2 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 1.2 MB 13.2 MB/s eta 0:00:01
    [?25hCollecting google-api-python-client>=1.6.7
      Downloading google_api_python_client-2.36.0-py2.py3-none-any.whl (8.0 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 8.0 MB 27.4 MB/s eta 0:00:01
    [?25hCollecting psutil>=5.4.3
      Downloading psutil-5.9.0-cp39-cp39-manylinux_2_12_x86_64.manylinux2010_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl (280 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 280 kB 24.5 MB/s eta 0:00:01
    [?25hCollecting gin-config
      Downloading gin_config-0.5.0-py3-none-any.whl (61 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 61 kB 1.3 MB/s eta 0:00:011
    [?25hCollecting tensorflow-datasets
      Downloading tensorflow_datasets-4.4.0-py3-none-any.whl (4.0 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 4.0 MB 32.3 MB/s eta 0:00:01
    [?25hCollecting py-cpuinfo>=3.3.0
      Downloading py-cpuinfo-8.0.0.tar.gz (99 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 99 kB 8.8 MB/s  eta 0:00:01
    [?25hCollecting tensorflow-addons
      Downloading tensorflow_addons-0.15.0-cp39-cp39-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (1.1 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 1.1 MB 28.4 MB/s eta 0:00:01
    [?25hCollecting tensorflow-model-optimization>=0.4.1
      Downloading tensorflow_model_optimization-0.7.0-py2.py3-none-any.whl (213 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 213 kB 38.5 MB/s eta 0:00:01
    [?25hCollecting kaggle>=1.3.9
      Downloading kaggle-1.5.12.tar.gz (58 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 58 kB 6.1 MB/s  eta 0:00:01
    [?25hCollecting sacrebleu
      Downloading sacrebleu-2.0.0-py3-none-any.whl (90 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 90 kB 9.6 MB/s  eta 0:00:01
    [?25hCollecting opencv-python-headless
      Downloading opencv_python_headless-4.5.5.62-cp36-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (47.7 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 47.7 MB 25.7 MB/s eta 0:00:01
    [?25hCollecting tensorflow-text>=2.7.0
      Downloading tensorflow_text-2.7.3-cp39-cp39-manylinux2010_x86_64.whl (4.9 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 4.9 MB 32.5 MB/s eta 0:00:01
    [?25hCollecting tensorflow>=2.7.0
      Downloading tensorflow-2.7.0-cp39-cp39-manylinux2010_x86_64.whl (489.7 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 489.7 MB 36 kB/s s eta 0:00:011
    [?25hRequirement already satisfied: numpy>=1.15.4 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tf-models-official>=2.5.1->object-detection==0.1) (1.19.2)
    Collecting pyyaml>=5.1
      Downloading PyYAML-6.0-cp39-cp39-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (661 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 661 kB 525 kB/s eta 0:00:01
    [?25hCollecting uritemplate<5,>=3.0.1
      Downloading uritemplate-4.1.1-py2.py3-none-any.whl (10 kB)
    Collecting httplib2<1dev,>=0.15.0
      Downloading httplib2-0.20.2-py3-none-any.whl (96 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 96 kB 5.0 MB/s  eta 0:00:01
    [?25hRequirement already satisfied: google-auth<3.0.0dev,>=1.16.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (1.21.3)
    Collecting google-api-core<3.0.0dev,>=1.21.0
      Downloading google_api_core-2.4.0-py2.py3-none-any.whl (111 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 111 kB 31.6 MB/s eta 0:00:01
    [?25hCollecting google-auth-httplib2>=0.1.0
      Downloading google_auth_httplib2-0.1.0-py2.py3-none-any.whl (9.3 kB)
    Collecting google-auth<3.0.0dev,>=1.16.0
      Using cached google_auth-2.5.0-py2.py3-none-any.whl (157 kB)
    Requirement already satisfied: requests<3.0.0dev,>=2.18.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from google-api-core<3.0.0dev,>=1.21.0->google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (2.27.1)
    Collecting googleapis-common-protos<2.0dev,>=1.52.0
      Downloading googleapis_common_protos-1.54.0-py2.py3-none-any.whl (207 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 207 kB 46.3 MB/s eta 0:00:01
    [?25hRequirement already satisfied: protobuf>=3.12.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from google-api-core<3.0.0dev,>=1.21.0->google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (3.19.1)
    Requirement already satisfied: setuptools>=40.3.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from google-api-core<3.0.0dev,>=1.21.0->google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (58.0.4)
    Requirement already satisfied: pyasn1-modules>=0.2.1 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from google-auth<3.0.0dev,>=1.16.0->google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (0.2.8)
    Requirement already satisfied: cachetools<6.0,>=2.0.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from google-auth<3.0.0dev,>=1.16.0->google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (4.1.1)
    Requirement already satisfied: rsa<5,>=3.1.4 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from google-auth<3.0.0dev,>=1.16.0->google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (4.6)
    Requirement already satisfied: pyparsing!=3.0.0,!=3.0.1,!=3.0.2,!=3.0.3,<4,>=2.4.2 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from httplib2<1dev,>=0.15.0->google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (3.0.7)
    Requirement already satisfied: certifi in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from kaggle>=1.3.9->tf-models-official>=2.5.1->object-detection==0.1) (2021.10.8)
    Requirement already satisfied: python-dateutil in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from kaggle>=1.3.9->tf-models-official>=2.5.1->object-detection==0.1) (2.8.2)
    Collecting tqdm
      Downloading tqdm-4.62.3-py2.py3-none-any.whl (76 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 76 kB 3.8 MB/s  eta 0:00:01
    [?25hCollecting python-slugify
      Downloading python_slugify-5.0.2-py2.py3-none-any.whl (6.7 kB)
    Requirement already satisfied: urllib3 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from kaggle>=1.3.9->tf-models-official>=2.5.1->object-detection==0.1) (1.25.11)
    Collecting pytz>=2020.1
      Downloading pytz-2021.3-py2.py3-none-any.whl (503 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 503 kB 10.2 MB/s eta 0:00:01
    [?25hRequirement already satisfied: pyasn1<0.5.0,>=0.4.6 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from pyasn1-modules>=0.2.1->google-auth<3.0.0dev,>=1.16.0->google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (0.4.8)
    Requirement already satisfied: idna<4,>=2.5 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from requests<3.0.0dev,>=2.18.0->google-api-core<3.0.0dev,>=1.21.0->google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (2.10)
    Requirement already satisfied: charset-normalizer~=2.0.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from requests<3.0.0dev,>=2.18.0->google-api-core<3.0.0dev,>=1.21.0->google-api-python-client>=1.6.7->tf-models-official>=2.5.1->object-detection==0.1) (2.0.4)
    Requirement already satisfied: absl-py>=0.4.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (0.15.0)
    Requirement already satisfied: google-pasta>=0.1.1 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (0.2.0)
    Requirement already satisfied: keras-preprocessing>=1.1.1 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (1.1.2)
    Requirement already satisfied: h5py>=2.9.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (2.10.0)
    Collecting libclang>=9.0.1
      Downloading libclang-12.0.0-2-py2.py3-none-manylinux1_x86_64.whl (13.3 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 13.3 MB 37.6 MB/s eta 0:00:01
    [?25hRequirement already satisfied: tensorboard~=2.6 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (2.6.0)
    Requirement already satisfied: termcolor>=1.1.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (1.1.0)
    Collecting tensorflow-estimator<2.8,~=2.7.0rc0
      Downloading tensorflow_estimator-2.7.0-py2.py3-none-any.whl (463 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 463 kB 36.9 MB/s eta 0:00:01
    [?25hRequirement already satisfied: wrapt>=1.11.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (1.13.3)
    Requirement already satisfied: opt-einsum>=2.3.2 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (3.1.0)
    Requirement already satisfied: wheel<1.0,>=0.32.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (0.37.1)
    Requirement already satisfied: gast<0.5.0,>=0.2.1 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (0.4.0)
    Requirement already satisfied: grpcio<2.0,>=1.24.3 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (1.42.0)
    Collecting tensorflow-io-gcs-filesystem>=0.21.0
      Downloading tensorflow_io_gcs_filesystem-0.23.1-cp39-cp39-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (2.1 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 2.1 MB 31.8 MB/s eta 0:00:01
    [?25hRequirement already satisfied: astunparse>=1.6.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (1.6.3)
    Requirement already satisfied: typing-extensions>=3.6.6 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (4.0.1)
    Requirement already satisfied: flatbuffers<3.0,>=1.12 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (2.0)
    Requirement already satisfied: markdown>=2.6.8 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorboard~=2.6->tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (3.3.4)
    Collecting google-auth<3.0.0dev,>=1.16.0
      Downloading google_auth-1.35.0-py2.py3-none-any.whl (152 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 152 kB 29.7 MB/s eta 0:00:01
    [?25hRequirement already satisfied: werkzeug>=0.11.15 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorboard~=2.6->tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (1.0.1)
    Requirement already satisfied: google-auth-oauthlib<0.5,>=0.4.1 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorboard~=2.6->tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (0.4.1)
    Requirement already satisfied: tensorboard-plugin-wit>=1.6.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorboard~=2.6->tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (1.6.0)
    Requirement already satisfied: tensorboard-data-server<0.7.0,>=0.6.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorboard~=2.6->tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (0.6.0)
    Requirement already satisfied: requests-oauthlib>=0.7.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from google-auth-oauthlib<0.5,>=0.4.1->tensorboard~=2.6->tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (1.3.0)
    Requirement already satisfied: oauthlib>=3.0.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from requests-oauthlib>=0.7.0->google-auth-oauthlib<0.5,>=0.4.1->tensorboard~=2.6->tensorflow>=2.7.0->tf-models-official>=2.5.1->object-detection==0.1) (3.1.0)
    Collecting dm-tree~=0.1.1
      Downloading dm_tree-0.1.6-cp39-cp39-manylinux_2_24_x86_64.whl (94 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 94 kB 3.1 MB/s  eta 0:00:01
    [?25hCollecting crcmod<2.0,>=1.7
      Downloading crcmod-1.7.tar.gz (89 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 89 kB 7.4 MB/s  eta 0:00:01
    [?25hCollecting orjson<4.0
      Downloading orjson-3.6.6-cp39-cp39-manylinux_2_24_x86_64.whl (245 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 245 kB 69.9 MB/s eta 0:00:01
    [?25hCollecting dill<0.3.2,>=0.3.1.1
      Downloading dill-0.3.1.1.tar.gz (151 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 151 kB 33.1 MB/s eta 0:00:01
    [?25hCollecting fastavro<2,>=0.21.4
      Downloading fastavro-1.4.9-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (2.5 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 2.5 MB 22.3 MB/s eta 0:00:01
    [?25hCollecting hdfs<3.0.0,>=2.1.0
      Downloading hdfs-2.6.0-py3-none-any.whl (33 kB)
    Collecting httplib2<1dev,>=0.15.0
      Downloading httplib2-0.19.1-py3-none-any.whl (95 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 95 kB 4.1 MB/s  eta 0:00:01
    [?25hCollecting pymongo<4.0.0,>=3.8.0
      Downloading pymongo-3.12.3-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (516 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 516 kB 40.7 MB/s eta 0:00:01
    [?25hCollecting proto-plus<2,>=1.7.1
      Downloading proto_plus-1.19.9-py3-none-any.whl (45 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 45 kB 3.0 MB/s  eta 0:00:01
    [?25hCollecting pyarrow<7.0.0,>=0.15.1
      Downloading pyarrow-6.0.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25.6 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 25.6 MB 23.7 MB/s eta 0:00:01
    [?25hCollecting pydot<2,>=1.2.0
      Downloading pydot-1.4.2-py2.py3-none-any.whl (21 kB)
    Collecting typing-extensions>=3.6.6
      Downloading typing_extensions-3.10.0.2-py3-none-any.whl (26 kB)
    Collecting docopt
      Downloading docopt-0.6.2.tar.gz (25 kB)
    Collecting pyparsing!=3.0.0,!=3.0.1,!=3.0.2,!=3.0.3,<4,>=2.4.2
      Downloading pyparsing-2.4.7-py2.py3-none-any.whl (67 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 67 kB 4.9 MB/s  eta 0:00:01
    [?25hCollecting kiwisolver>=1.1.0
      Downloading kiwisolver-1.3.2-cp39-cp39-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (1.6 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 1.6 MB 48.6 MB/s eta 0:00:01
    [?25hCollecting cycler>=0.10.0
      Downloading cycler-0.11.0-py3-none-any.whl (6.4 kB)
    Collecting opencv-python>=4.1.0.25
      Using cached opencv_python-4.5.5.62-cp36-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (60.4 MB)
    Requirement already satisfied: packaging>=20.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from matplotlib->object-detection==0.1) (21.3)
    Collecting fonttools>=4.22.0
      Downloading fonttools-4.29.0-py3-none-any.whl (895 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 895 kB 18.4 MB/s eta 0:00:01
    [?25hCollecting numpy>=1.15.4
      Downloading numpy-1.20.3-cp39-cp39-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (15.4 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 15.4 MB 22.5 MB/s eta 0:00:01
    [?25hCollecting text-unidecode>=1.3
      Downloading text_unidecode-1.3-py2.py3-none-any.whl (78 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 78 kB 6.7 MB/s  eta 0:00:011
    [?25hCollecting portalocker
      Downloading portalocker-2.3.2-py2.py3-none-any.whl (15 kB)
    Collecting regex
      Downloading regex-2022.1.18-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (763 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 763 kB 17.0 MB/s eta 0:00:01
    [?25hCollecting tabulate>=0.8.9
      Downloading tabulate-0.8.9-py3-none-any.whl (25 kB)
    Collecting colorama
      Downloading colorama-0.4.4-py2.py3-none-any.whl (16 kB)
    Collecting scikit-learn>=0.21.3
      Downloading scikit_learn-1.0.2-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (26.4 MB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 26.4 MB 29.3 MB/s eta 0:00:01
    [?25hCollecting joblib>=0.11
      Downloading joblib-1.1.0-py2.py3-none-any.whl (306 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 306 kB 31.9 MB/s eta 0:00:01
    [?25hCollecting threadpoolctl>=2.0.0
      Downloading threadpoolctl-3.0.0-py3-none-any.whl (14 kB)
    Collecting typeguard>=2.7
      Downloading typeguard-2.13.3-py3-none-any.whl (17 kB)
    Collecting promise
      Downloading promise-2.3.tar.gz (19 kB)
    Requirement already satisfied: attrs>=18.1.0 in /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages (from tensorflow-datasets->tf-models-official>=2.5.1->object-detection==0.1) (21.4.0)
    Collecting tensorflow-metadata
      Downloading tensorflow_metadata-1.6.0-py3-none-any.whl (48 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 48 kB 4.9 MB/s  eta 0:00:01
    [?25hCollecting future
      Downloading future-0.18.2.tar.gz (829 kB)
    [K     |????????????????????????????????????????????????????????????????????????????????????????????????| 829 kB 56.3 MB/s eta 0:00:01
    [?25hBuilding wheels for collected packages: object-detection, kaggle, py-cpuinfo, apache-beam, crcmod, dill, avro-python3, docopt, pycocotools, seqeval, future, promise
      Building wheel for object-detection (setup.py) ... [?25ldone
    [?25h  Created wheel for object-detection: filename=object_detection-0.1-py3-none-any.whl size=1651960 sha256=b4a770825456ecb980e9c1194be56efd5c7b82398e1ee141758090889036b201
      Stored in directory: /tmp/pip-ephem-wheel-cache-_lazcpli/wheels/39/13/de/7907e0ebd1103d93d3fbbf5800d2127ab2314a9468e208dd7d
      Building wheel for kaggle (setup.py) ... [?25ldone
    [?25h  Created wheel for kaggle: filename=kaggle-1.5.12-py3-none-any.whl size=73051 sha256=11a9da565a8f68f81379a0ebf7562e418661845909e9b01ee3323f6cb4e36ddd
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/ac/b2/c3/fa4706d469b5879105991d1c8be9a3c2ef329ba9fe2ce5085e
      Building wheel for py-cpuinfo (setup.py) ... [?25ldone
    [?25h  Created wheel for py-cpuinfo: filename=py_cpuinfo-8.0.0-py3-none-any.whl size=22257 sha256=480b6b4d0882ec3d7c8792187a30589cdddc9c2f33798bf4df11bda240dca83c
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/a9/33/c2/bcf6550ff9c95f699d7b2f261c8520b42b7f7c33b6e6920e29
      Building wheel for apache-beam (setup.py) ... [?25ldone
    [?25h  Created wheel for apache-beam: filename=apache_beam-2.35.0-cp39-cp39-linux_x86_64.whl size=4559874 sha256=785b201cbc0b0006b772fb07a0f4704b9a479d0f24fc7c3362227910dfbf02a8
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/bd/f1/0a/5766032006be3a4bb638cabefcf84bd192a747f3ef2a48adbb
      Building wheel for crcmod (setup.py) ... [?25ldone
    [?25h  Created wheel for crcmod: filename=crcmod-1.7-cp39-cp39-linux_x86_64.whl size=23681 sha256=a74f294bda1fac48d202425d751e863a3f667e4510f60ad5124a5868cbb4f71c
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/4a/6c/a6/ffdd136310039bf226f2707a9a8e6857be7d70a3fc061f6b36
      Building wheel for dill (setup.py) ... [?25ldone
    [?25h  Created wheel for dill: filename=dill-0.3.1.1-py3-none-any.whl size=78544 sha256=25bb4c4730b62c27f2d0aacc663fffe214fb03bc0ac7e24b69b8dc65de7c67c3
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/4f/0b/ce/75d96dd714b15e51cb66db631183ea3844e0c4a6d19741a149
      Building wheel for avro-python3 (setup.py) ... [?25ldone
    [?25h  Created wheel for avro-python3: filename=avro_python3-1.10.2-py3-none-any.whl size=44010 sha256=4239a0a07dc1701f14570d4f9d481bc2baf29fa4fc70eb64f9edb18fa6a7dda8
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/5a/29/4d/510c0e098c49c5e49519f430481a5425e60b8752682d7b1e55
      Building wheel for docopt (setup.py) ... [?25ldone
    [?25h  Created wheel for docopt: filename=docopt-0.6.2-py2.py3-none-any.whl size=13723 sha256=cb7c111b7d9cabb89c247e06924c67af85cf484fea43dae60086e1c65e8be41d
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/70/4a/46/1309fc853b8d395e60bafaf1b6df7845bdd82c95fd59dd8d2b
      Building wheel for pycocotools (PEP 517) ... [?25ldone
    [?25h  Created wheel for pycocotools: filename=pycocotools-2.0.4-cp39-cp39-linux_x86_64.whl size=103609 sha256=128ef223cd1f8dd7b2fff6b1e1b4cd54b1a99ddf5201c08a93a32bb8fac06f2d
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/7e/b0/8e/f2c3593944ead79f5146d057d1310ee6d7b60d30b826779846
      Building wheel for seqeval (setup.py) ... [?25ldone
    [?25h  Created wheel for seqeval: filename=seqeval-1.2.2-py3-none-any.whl size=16180 sha256=e018962b1ec240867d1922631f6ee71b91f50cc4f183d0e8fc3621333de08545
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/e2/a5/92/2c80d1928733611c2747a9820e1324a6835524d9411510c142
      Building wheel for future (setup.py) ... [?25ldone
    [?25h  Created wheel for future: filename=future-0.18.2-py3-none-any.whl size=491070 sha256=bfa636879c4c7ace4e73f9503f31a39fcbd5b6187317090d8579f13b0ee0c0ac
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/2f/a0/d3/4030d9f80e6b3be787f19fc911b8e7aa462986a40ab1e4bb94
      Building wheel for promise (setup.py) ... [?25ldone
    [?25h  Created wheel for promise: filename=promise-2.3-py3-none-any.whl size=21503 sha256=a0b5ba58931c82628cff08098d31ee1fa7560da0a435ace40e10cd307e66942e
      Stored in directory: /home/ubuntu-20/.cache/pip/wheels/e1/e8/83/ddea66100678d139b14bc87692ece57c6a2a937956d2532608
    Successfully built object-detection kaggle py-cpuinfo apache-beam crcmod dill avro-python3 docopt pycocotools seqeval future promise
    Installing collected packages: google-auth, pyparsing, numpy, typing-extensions, threadpoolctl, text-unidecode, tensorflow-io-gcs-filesystem, tensorflow-estimator, pillow, libclang, kiwisolver, keras, joblib, httplib2, googleapis-common-protos, fonttools, cycler, uritemplate, typeguard, tqdm, tensorflow-metadata, tensorflow-hub, tensorflow, tabulate, scikit-learn, regex, pytz, python-slugify, promise, portalocker, matplotlib, google-auth-httplib2, google-api-core, future, docopt, dm-tree, dill, colorama, tf-slim, tensorflow-text, tensorflow-model-optimization, tensorflow-datasets, tensorflow-addons, seqeval, sentencepiece, sacrebleu, pyyaml, pymongo, pydot, pycocotools, pyarrow, py-cpuinfo, psutil, proto-plus, pandas, orjson, opencv-python-headless, opencv-python, oauth2client, kaggle, hdfs, google-api-python-client, gin-config, fastavro, crcmod, tf-models-official, tensorflow-io, lxml, lvis, contextlib2, avro-python3, apache-beam, object-detection
      Attempting uninstall: google-auth
        Found existing installation: google-auth 1.21.3
        Uninstalling google-auth-1.21.3:
          Successfully uninstalled google-auth-1.21.3
      Attempting uninstall: pyparsing
        Found existing installation: pyparsing 3.0.7
        Uninstalling pyparsing-3.0.7:
          Successfully uninstalled pyparsing-3.0.7
      Attempting uninstall: numpy
        Found existing installation: numpy 1.19.2
        Uninstalling numpy-1.19.2:
          Successfully uninstalled numpy-1.19.2
      Attempting uninstall: typing-extensions
        Found existing installation: typing-extensions 4.0.1
        Uninstalling typing-extensions-4.0.1:
          Successfully uninstalled typing-extensions-4.0.1
      Attempting uninstall: tensorflow-estimator
        Found existing installation: tensorflow-estimator 2.4.0
        Uninstalling tensorflow-estimator-2.4.0:
          Successfully uninstalled tensorflow-estimator-2.4.0
      Attempting uninstall: tensorflow
        Found existing installation: tensorflow 2.4.1
        Uninstalling tensorflow-2.4.1:
          Successfully uninstalled tensorflow-2.4.1
    Successfully installed apache-beam-2.35.0 avro-python3-1.10.2 colorama-0.4.4 contextlib2-21.6.0 crcmod-1.7 cycler-0.11.0 dill-0.3.1.1 dm-tree-0.1.6 docopt-0.6.2 fastavro-1.4.9 fonttools-4.29.0 future-0.18.2 gin-config-0.5.0 google-api-core-2.4.0 google-api-python-client-2.36.0 google-auth-1.35.0 google-auth-httplib2-0.1.0 googleapis-common-protos-1.54.0 hdfs-2.6.0 httplib2-0.19.1 joblib-1.1.0 kaggle-1.5.12 keras-2.7.0 kiwisolver-1.3.2 libclang-12.0.0 lvis-0.5.3 lxml-4.7.1 matplotlib-3.5.1 numpy-1.20.3 oauth2client-4.1.3 object-detection-0.1 opencv-python-4.5.5.62 opencv-python-headless-4.5.5.62 orjson-3.6.6 pandas-1.4.0 pillow-9.0.0 portalocker-2.3.2 promise-2.3 proto-plus-1.19.9 psutil-5.9.0 py-cpuinfo-8.0.0 pyarrow-6.0.1 pycocotools-2.0.4 pydot-1.4.2 pymongo-3.12.3 pyparsing-2.4.7 python-slugify-5.0.2 pytz-2021.3 pyyaml-6.0 regex-2022.1.18 sacrebleu-2.0.0 scikit-learn-1.0.2 sentencepiece-0.1.96 seqeval-1.2.2 tabulate-0.8.9 tensorflow-2.7.0 tensorflow-addons-0.15.0 tensorflow-datasets-4.4.0 tensorflow-estimator-2.7.0 tensorflow-hub-0.12.0 tensorflow-io-0.23.1 tensorflow-io-gcs-filesystem-0.23.1 tensorflow-metadata-1.6.0 tensorflow-model-optimization-0.7.0 tensorflow-text-2.7.3 text-unidecode-1.3 tf-models-official-2.7.0 tf-slim-1.1.0 threadpoolctl-3.0.0 tqdm-4.62.3 typeguard-2.13.3 typing-extensions-3.10.0.2 uritemplate-4.1.1


Now object detection setup is done. You can verify the installation by executing the following command.


```python
!python object_detection/builders/model_builder_tf2_test.py
```

    2022-01-26 14:06:57.624502: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:939] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
    2022-01-26 14:06:57.627064: W tensorflow/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libcusolver.so.11'; dlerror: libcusolver.so.11: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: /usr/local/cuda/lib64
    2022-01-26 14:06:57.627577: W tensorflow/core/common_runtime/gpu/gpu_device.cc:1850] Cannot dlopen some GPU libraries. Please make sure the missing libraries mentioned above are installed properly if you would like to use GPU. Follow the guide at https://www.tensorflow.org/install/gpu for how to download and setup the required libraries for your platform.
    Skipping registering GPU devices...
    Running tests under Python 3.9.7: /home/ubuntu-20/anaconda3/envs/tensorflow/bin/python
    [ RUN      ] ModelBuilderTF2Test.test_create_center_net_deepmac
    2022-01-26 14:06:57.633834: I tensorflow/core/platform/cpu_feature_guard.cc:151] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
    To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
    /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/builders/model_builder.py:1100: DeprecationWarning: The 'warn' function is deprecated, use 'warning' instead
      logging.warn(('Building experimental DeepMAC meta-arch.'
    W0126 14:06:57.887338 139775303639424 model_builder.py:1100] Building experimental DeepMAC meta-arch. Some features may be omitted.
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_center_net_deepmac): 0.43s
    I0126 14:06:58.056958 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_center_net_deepmac): 0.43s
    [       OK ] ModelBuilderTF2Test.test_create_center_net_deepmac
    [ RUN      ] ModelBuilderTF2Test.test_create_center_net_model0 (customize_head_params=True)
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_center_net_model0 (customize_head_params=True)): 0.32s
    I0126 14:06:58.382251 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_center_net_model0 (customize_head_params=True)): 0.32s
    [       OK ] ModelBuilderTF2Test.test_create_center_net_model0 (customize_head_params=True)
    [ RUN      ] ModelBuilderTF2Test.test_create_center_net_model1 (customize_head_params=False)
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_center_net_model1 (customize_head_params=False)): 0.18s
    I0126 14:06:58.560569 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_center_net_model1 (customize_head_params=False)): 0.18s
    [       OK ] ModelBuilderTF2Test.test_create_center_net_model1 (customize_head_params=False)
    [ RUN      ] ModelBuilderTF2Test.test_create_center_net_model_from_keypoints
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_center_net_model_from_keypoints): 0.27s
    I0126 14:06:58.836110 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_center_net_model_from_keypoints): 0.27s
    [       OK ] ModelBuilderTF2Test.test_create_center_net_model_from_keypoints
    [ RUN      ] ModelBuilderTF2Test.test_create_center_net_model_mobilenet
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_center_net_model_mobilenet): 1.94s
    I0126 14:07:00.775264 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_center_net_model_mobilenet): 1.94s
    [       OK ] ModelBuilderTF2Test.test_create_center_net_model_mobilenet
    [ RUN      ] ModelBuilderTF2Test.test_create_experimental_model
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_experimental_model): 0.0s
    I0126 14:07:00.776373 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_experimental_model): 0.0s
    [       OK ] ModelBuilderTF2Test.test_create_experimental_model
    [ RUN      ] ModelBuilderTF2Test.test_create_faster_rcnn_from_config_with_crop_feature0 (True)
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_from_config_with_crop_feature0 (True)): 0.02s
    I0126 14:07:00.794725 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_from_config_with_crop_feature0 (True)): 0.02s
    [       OK ] ModelBuilderTF2Test.test_create_faster_rcnn_from_config_with_crop_feature0 (True)
    [ RUN      ] ModelBuilderTF2Test.test_create_faster_rcnn_from_config_with_crop_feature1 (False)
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_from_config_with_crop_feature1 (False)): 0.01s
    I0126 14:07:00.804883 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_from_config_with_crop_feature1 (False)): 0.01s
    [       OK ] ModelBuilderTF2Test.test_create_faster_rcnn_from_config_with_crop_feature1 (False)
    [ RUN      ] ModelBuilderTF2Test.test_create_faster_rcnn_model_from_config_with_example_miner
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_model_from_config_with_example_miner): 0.01s
    I0126 14:07:00.815815 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_model_from_config_with_example_miner): 0.01s
    [       OK ] ModelBuilderTF2Test.test_create_faster_rcnn_model_from_config_with_example_miner
    [ RUN      ] ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_faster_rcnn_with_matmul
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_faster_rcnn_with_matmul): 0.08s
    I0126 14:07:00.892999 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_faster_rcnn_with_matmul): 0.08s
    [       OK ] ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_faster_rcnn_with_matmul
    [ RUN      ] ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_faster_rcnn_without_matmul
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_faster_rcnn_without_matmul): 0.08s
    I0126 14:07:00.975105 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_faster_rcnn_without_matmul): 0.08s
    [       OK ] ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_faster_rcnn_without_matmul
    [ RUN      ] ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_mask_rcnn_with_matmul
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_mask_rcnn_with_matmul): 0.08s
    I0126 14:07:01.056505 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_mask_rcnn_with_matmul): 0.08s
    [       OK ] ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_mask_rcnn_with_matmul
    [ RUN      ] ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_mask_rcnn_without_matmul
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_mask_rcnn_without_matmul): 0.07s
    I0126 14:07:01.129811 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_mask_rcnn_without_matmul): 0.07s
    [       OK ] ModelBuilderTF2Test.test_create_faster_rcnn_models_from_config_mask_rcnn_without_matmul
    [ RUN      ] ModelBuilderTF2Test.test_create_rfcn_model_from_config
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_rfcn_model_from_config): 0.22s
    I0126 14:07:01.345800 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_rfcn_model_from_config): 0.22s
    [       OK ] ModelBuilderTF2Test.test_create_rfcn_model_from_config
    [ RUN      ] ModelBuilderTF2Test.test_create_ssd_fpn_model_from_config
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_ssd_fpn_model_from_config): 0.02s
    I0126 14:07:01.367722 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_ssd_fpn_model_from_config): 0.02s
    [       OK ] ModelBuilderTF2Test.test_create_ssd_fpn_model_from_config
    [ RUN      ] ModelBuilderTF2Test.test_create_ssd_models_from_config
    I0126 14:07:01.496387 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b0
    I0126 14:07:01.496513 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 64
    I0126 14:07:01.496586 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 3
    I0126 14:07:01.498541 139775303639424 efficientnet_model.py:147] round_filter input=32 output=32
    I0126 14:07:01.511177 139775303639424 efficientnet_model.py:147] round_filter input=32 output=32
    I0126 14:07:01.511280 139775303639424 efficientnet_model.py:147] round_filter input=16 output=16
    I0126 14:07:01.567244 139775303639424 efficientnet_model.py:147] round_filter input=16 output=16
    I0126 14:07:01.567365 139775303639424 efficientnet_model.py:147] round_filter input=24 output=24
    I0126 14:07:01.709391 139775303639424 efficientnet_model.py:147] round_filter input=24 output=24
    I0126 14:07:01.709509 139775303639424 efficientnet_model.py:147] round_filter input=40 output=40
    I0126 14:07:01.866763 139775303639424 efficientnet_model.py:147] round_filter input=40 output=40
    I0126 14:07:01.866876 139775303639424 efficientnet_model.py:147] round_filter input=80 output=80
    I0126 14:07:02.097244 139775303639424 efficientnet_model.py:147] round_filter input=80 output=80
    I0126 14:07:02.097411 139775303639424 efficientnet_model.py:147] round_filter input=112 output=112
    I0126 14:07:02.300666 139775303639424 efficientnet_model.py:147] round_filter input=112 output=112
    I0126 14:07:02.300801 139775303639424 efficientnet_model.py:147] round_filter input=192 output=192
    I0126 14:07:02.603019 139775303639424 efficientnet_model.py:147] round_filter input=192 output=192
    I0126 14:07:02.603131 139775303639424 efficientnet_model.py:147] round_filter input=320 output=320
    I0126 14:07:02.672072 139775303639424 efficientnet_model.py:147] round_filter input=1280 output=1280
    I0126 14:07:02.704981 139775303639424 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=1.0, depth_coefficient=1.0, resolution=224, dropout_rate=0.2, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    I0126 14:07:02.740081 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b1
    I0126 14:07:02.740202 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 88
    I0126 14:07:02.740279 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 4
    I0126 14:07:02.741616 139775303639424 efficientnet_model.py:147] round_filter input=32 output=32
    I0126 14:07:02.757367 139775303639424 efficientnet_model.py:147] round_filter input=32 output=32
    I0126 14:07:02.757482 139775303639424 efficientnet_model.py:147] round_filter input=16 output=16
    I0126 14:07:02.892325 139775303639424 efficientnet_model.py:147] round_filter input=16 output=16
    I0126 14:07:02.892455 139775303639424 efficientnet_model.py:147] round_filter input=24 output=24
    I0126 14:07:03.140067 139775303639424 efficientnet_model.py:147] round_filter input=24 output=24
    I0126 14:07:03.140193 139775303639424 efficientnet_model.py:147] round_filter input=40 output=40
    I0126 14:07:03.355316 139775303639424 efficientnet_model.py:147] round_filter input=40 output=40
    I0126 14:07:03.355602 139775303639424 efficientnet_model.py:147] round_filter input=80 output=80
    I0126 14:07:03.704387 139775303639424 efficientnet_model.py:147] round_filter input=80 output=80
    I0126 14:07:03.704514 139775303639424 efficientnet_model.py:147] round_filter input=112 output=112
    I0126 14:07:04.048340 139775303639424 efficientnet_model.py:147] round_filter input=112 output=112
    I0126 14:07:04.048468 139775303639424 efficientnet_model.py:147] round_filter input=192 output=192
    I0126 14:07:04.445351 139775303639424 efficientnet_model.py:147] round_filter input=192 output=192
    I0126 14:07:04.445484 139775303639424 efficientnet_model.py:147] round_filter input=320 output=320
    I0126 14:07:04.601372 139775303639424 efficientnet_model.py:147] round_filter input=1280 output=1280
    I0126 14:07:04.637036 139775303639424 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=1.0, depth_coefficient=1.1, resolution=240, dropout_rate=0.2, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    I0126 14:07:04.813985 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b2
    I0126 14:07:04.814128 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 112
    I0126 14:07:04.814162 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 5
    I0126 14:07:04.815535 139775303639424 efficientnet_model.py:147] round_filter input=32 output=32
    I0126 14:07:04.835434 139775303639424 efficientnet_model.py:147] round_filter input=32 output=32
    I0126 14:07:04.835582 139775303639424 efficientnet_model.py:147] round_filter input=16 output=16
    I0126 14:07:04.986997 139775303639424 efficientnet_model.py:147] round_filter input=16 output=16
    I0126 14:07:04.987106 139775303639424 efficientnet_model.py:147] round_filter input=24 output=24
    I0126 14:07:05.199536 139775303639424 efficientnet_model.py:147] round_filter input=24 output=24
    I0126 14:07:05.199649 139775303639424 efficientnet_model.py:147] round_filter input=40 output=48
    I0126 14:07:05.415032 139775303639424 efficientnet_model.py:147] round_filter input=40 output=48
    I0126 14:07:05.415156 139775303639424 efficientnet_model.py:147] round_filter input=80 output=88
    I0126 14:07:05.802588 139775303639424 efficientnet_model.py:147] round_filter input=80 output=88
    I0126 14:07:05.802737 139775303639424 efficientnet_model.py:147] round_filter input=112 output=120
    I0126 14:07:06.088043 139775303639424 efficientnet_model.py:147] round_filter input=112 output=120
    I0126 14:07:06.088167 139775303639424 efficientnet_model.py:147] round_filter input=192 output=208
    I0126 14:07:06.498754 139775303639424 efficientnet_model.py:147] round_filter input=192 output=208
    I0126 14:07:06.498878 139775303639424 efficientnet_model.py:147] round_filter input=320 output=352
    I0126 14:07:06.645416 139775303639424 efficientnet_model.py:147] round_filter input=1280 output=1408
    I0126 14:07:06.686201 139775303639424 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=1.1, depth_coefficient=1.2, resolution=260, dropout_rate=0.3, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    I0126 14:07:06.730251 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b3
    I0126 14:07:06.730380 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 160
    I0126 14:07:06.730456 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 6
    I0126 14:07:06.731891 139775303639424 efficientnet_model.py:147] round_filter input=32 output=40
    I0126 14:07:06.744326 139775303639424 efficientnet_model.py:147] round_filter input=32 output=40
    I0126 14:07:06.744479 139775303639424 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 14:07:06.844000 139775303639424 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 14:07:06.844130 139775303639424 efficientnet_model.py:147] round_filter input=24 output=32
    I0126 14:07:07.060233 139775303639424 efficientnet_model.py:147] round_filter input=24 output=32
    I0126 14:07:07.060352 139775303639424 efficientnet_model.py:147] round_filter input=40 output=48
    I0126 14:07:07.298660 139775303639424 efficientnet_model.py:147] round_filter input=40 output=48
    I0126 14:07:07.298794 139775303639424 efficientnet_model.py:147] round_filter input=80 output=96
    I0126 14:07:07.688406 139775303639424 efficientnet_model.py:147] round_filter input=80 output=96
    I0126 14:07:07.688540 139775303639424 efficientnet_model.py:147] round_filter input=112 output=136
    I0126 14:07:08.108583 139775303639424 efficientnet_model.py:147] round_filter input=112 output=136
    I0126 14:07:08.108700 139775303639424 efficientnet_model.py:147] round_filter input=192 output=232
    I0126 14:07:08.575955 139775303639424 efficientnet_model.py:147] round_filter input=192 output=232
    I0126 14:07:08.576074 139775303639424 efficientnet_model.py:147] round_filter input=320 output=384
    I0126 14:07:08.739605 139775303639424 efficientnet_model.py:147] round_filter input=1280 output=1536
    I0126 14:07:08.774342 139775303639424 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=1.2, depth_coefficient=1.4, resolution=300, dropout_rate=0.3, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    I0126 14:07:08.827544 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b4
    I0126 14:07:08.827677 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 224
    I0126 14:07:08.827760 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 7
    I0126 14:07:08.829215 139775303639424 efficientnet_model.py:147] round_filter input=32 output=48
    I0126 14:07:08.845425 139775303639424 efficientnet_model.py:147] round_filter input=32 output=48
    I0126 14:07:08.845582 139775303639424 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 14:07:08.958070 139775303639424 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 14:07:08.958185 139775303639424 efficientnet_model.py:147] round_filter input=24 output=32
    I0126 14:07:09.225388 139775303639424 efficientnet_model.py:147] round_filter input=24 output=32
    I0126 14:07:09.225503 139775303639424 efficientnet_model.py:147] round_filter input=40 output=56
    I0126 14:07:09.997115 139775303639424 efficientnet_model.py:147] round_filter input=40 output=56
    I0126 14:07:09.997481 139775303639424 efficientnet_model.py:147] round_filter input=80 output=112
    I0126 14:07:10.618946 139775303639424 efficientnet_model.py:147] round_filter input=80 output=112
    I0126 14:07:10.619070 139775303639424 efficientnet_model.py:147] round_filter input=112 output=160
    I0126 14:07:11.172519 139775303639424 efficientnet_model.py:147] round_filter input=112 output=160
    I0126 14:07:11.172716 139775303639424 efficientnet_model.py:147] round_filter input=192 output=272
    I0126 14:07:11.924983 139775303639424 efficientnet_model.py:147] round_filter input=192 output=272
    I0126 14:07:11.925154 139775303639424 efficientnet_model.py:147] round_filter input=320 output=448
    I0126 14:07:12.110312 139775303639424 efficientnet_model.py:147] round_filter input=1280 output=1792
    I0126 14:07:12.156013 139775303639424 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=1.4, depth_coefficient=1.8, resolution=380, dropout_rate=0.4, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    I0126 14:07:12.216317 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b5
    I0126 14:07:12.216438 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 288
    I0126 14:07:12.216550 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 7
    I0126 14:07:12.217988 139775303639424 efficientnet_model.py:147] round_filter input=32 output=48
    I0126 14:07:12.233840 139775303639424 efficientnet_model.py:147] round_filter input=32 output=48
    I0126 14:07:12.233994 139775303639424 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 14:07:12.442352 139775303639424 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 14:07:12.442608 139775303639424 efficientnet_model.py:147] round_filter input=24 output=40
    I0126 14:07:12.901839 139775303639424 efficientnet_model.py:147] round_filter input=24 output=40
    I0126 14:07:12.901962 139775303639424 efficientnet_model.py:147] round_filter input=40 output=64
    I0126 14:07:13.358628 139775303639424 efficientnet_model.py:147] round_filter input=40 output=64
    I0126 14:07:13.358763 139775303639424 efficientnet_model.py:147] round_filter input=80 output=128
    I0126 14:07:13.977698 139775303639424 efficientnet_model.py:147] round_filter input=80 output=128
    I0126 14:07:13.977850 139775303639424 efficientnet_model.py:147] round_filter input=112 output=176
    I0126 14:07:14.595020 139775303639424 efficientnet_model.py:147] round_filter input=112 output=176
    I0126 14:07:14.595132 139775303639424 efficientnet_model.py:147] round_filter input=192 output=304
    I0126 14:07:15.408861 139775303639424 efficientnet_model.py:147] round_filter input=192 output=304
    I0126 14:07:15.408992 139775303639424 efficientnet_model.py:147] round_filter input=320 output=512
    I0126 14:07:15.911651 139775303639424 efficientnet_model.py:147] round_filter input=1280 output=2048
    I0126 14:07:15.955137 139775303639424 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=1.6, depth_coefficient=2.2, resolution=456, dropout_rate=0.4, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    I0126 14:07:16.027103 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b6
    I0126 14:07:16.027235 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 384
    I0126 14:07:16.027269 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 8
    I0126 14:07:16.028584 139775303639424 efficientnet_model.py:147] round_filter input=32 output=56
    I0126 14:07:16.044325 139775303639424 efficientnet_model.py:147] round_filter input=32 output=56
    I0126 14:07:16.044440 139775303639424 efficientnet_model.py:147] round_filter input=16 output=32
    I0126 14:07:16.250075 139775303639424 efficientnet_model.py:147] round_filter input=16 output=32
    I0126 14:07:16.250248 139775303639424 efficientnet_model.py:147] round_filter input=24 output=40
    I0126 14:07:16.787415 139775303639424 efficientnet_model.py:147] round_filter input=24 output=40
    I0126 14:07:16.787561 139775303639424 efficientnet_model.py:147] round_filter input=40 output=72
    I0126 14:07:17.292409 139775303639424 efficientnet_model.py:147] round_filter input=40 output=72
    I0126 14:07:17.292533 139775303639424 efficientnet_model.py:147] round_filter input=80 output=144
    I0126 14:07:18.011171 139775303639424 efficientnet_model.py:147] round_filter input=80 output=144
    I0126 14:07:18.011286 139775303639424 efficientnet_model.py:147] round_filter input=112 output=200
    I0126 14:07:18.725660 139775303639424 efficientnet_model.py:147] round_filter input=112 output=200
    I0126 14:07:18.725782 139775303639424 efficientnet_model.py:147] round_filter input=192 output=344
    I0126 14:07:19.761791 139775303639424 efficientnet_model.py:147] round_filter input=192 output=344
    I0126 14:07:19.761954 139775303639424 efficientnet_model.py:147] round_filter input=320 output=576
    I0126 14:07:20.048552 139775303639424 efficientnet_model.py:147] round_filter input=1280 output=2304
    I0126 14:07:20.091845 139775303639424 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=1.8, depth_coefficient=2.6, resolution=528, dropout_rate=0.5, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    I0126 14:07:20.166972 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b7
    I0126 14:07:20.167091 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 384
    I0126 14:07:20.167124 139775303639424 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 8
    I0126 14:07:20.168422 139775303639424 efficientnet_model.py:147] round_filter input=32 output=64
    I0126 14:07:20.185153 139775303639424 efficientnet_model.py:147] round_filter input=32 output=64
    I0126 14:07:20.185356 139775303639424 efficientnet_model.py:147] round_filter input=16 output=32
    I0126 14:07:20.498679 139775303639424 efficientnet_model.py:147] round_filter input=16 output=32
    I0126 14:07:20.498799 139775303639424 efficientnet_model.py:147] round_filter input=24 output=48
    I0126 14:07:21.150535 139775303639424 efficientnet_model.py:147] round_filter input=24 output=48
    I0126 14:07:21.150656 139775303639424 efficientnet_model.py:147] round_filter input=40 output=80
    I0126 14:07:22.106084 139775303639424 efficientnet_model.py:147] round_filter input=40 output=80
    I0126 14:07:22.106252 139775303639424 efficientnet_model.py:147] round_filter input=80 output=160
    I0126 14:07:23.065239 139775303639424 efficientnet_model.py:147] round_filter input=80 output=160
    I0126 14:07:23.065360 139775303639424 efficientnet_model.py:147] round_filter input=112 output=224
    I0126 14:07:23.981920 139775303639424 efficientnet_model.py:147] round_filter input=112 output=224
    I0126 14:07:23.982052 139775303639424 efficientnet_model.py:147] round_filter input=192 output=384
    I0126 14:07:25.154724 139775303639424 efficientnet_model.py:147] round_filter input=192 output=384
    I0126 14:07:25.154839 139775303639424 efficientnet_model.py:147] round_filter input=320 output=640
    I0126 14:07:25.557402 139775303639424 efficientnet_model.py:147] round_filter input=1280 output=2560
    I0126 14:07:25.597800 139775303639424 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=2.0, depth_coefficient=3.1, resolution=600, dropout_rate=0.5, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_create_ssd_models_from_config): 24.32s
    I0126 14:07:25.692402 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_create_ssd_models_from_config): 24.32s
    [       OK ] ModelBuilderTF2Test.test_create_ssd_models_from_config
    [ RUN      ] ModelBuilderTF2Test.test_invalid_faster_rcnn_batchnorm_update
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_invalid_faster_rcnn_batchnorm_update): 0.0s
    I0126 14:07:25.702633 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_invalid_faster_rcnn_batchnorm_update): 0.0s
    [       OK ] ModelBuilderTF2Test.test_invalid_faster_rcnn_batchnorm_update
    [ RUN      ] ModelBuilderTF2Test.test_invalid_first_stage_nms_iou_threshold
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_invalid_first_stage_nms_iou_threshold): 0.0s
    I0126 14:07:25.704203 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_invalid_first_stage_nms_iou_threshold): 0.0s
    [       OK ] ModelBuilderTF2Test.test_invalid_first_stage_nms_iou_threshold
    [ RUN      ] ModelBuilderTF2Test.test_invalid_model_config_proto
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_invalid_model_config_proto): 0.0s
    I0126 14:07:25.704608 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_invalid_model_config_proto): 0.0s
    [       OK ] ModelBuilderTF2Test.test_invalid_model_config_proto
    [ RUN      ] ModelBuilderTF2Test.test_invalid_second_stage_batch_size
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_invalid_second_stage_batch_size): 0.0s
    I0126 14:07:25.705996 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_invalid_second_stage_batch_size): 0.0s
    [       OK ] ModelBuilderTF2Test.test_invalid_second_stage_batch_size
    [ RUN      ] ModelBuilderTF2Test.test_session
    [  SKIPPED ] ModelBuilderTF2Test.test_session
    [ RUN      ] ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor): 0.0s
    I0126 14:07:25.707099 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor): 0.0s
    [       OK ] ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor
    [ RUN      ] ModelBuilderTF2Test.test_unknown_meta_architecture
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_unknown_meta_architecture): 0.0s
    I0126 14:07:25.707320 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_unknown_meta_architecture): 0.0s
    [       OK ] ModelBuilderTF2Test.test_unknown_meta_architecture
    [ RUN      ] ModelBuilderTF2Test.test_unknown_ssd_feature_extractor
    INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_unknown_ssd_feature_extractor): 0.0s
    I0126 14:07:25.708297 139775303639424 test_util.py:2308] time(__main__.ModelBuilderTF2Test.test_unknown_ssd_feature_extractor): 0.0s
    [       OK ] ModelBuilderTF2Test.test_unknown_ssd_feature_extractor
    ----------------------------------------------------------------------
    Ran 24 tests in 28.079s
    
    OK (skipped=1)


## Train Object Detection

To train an Object Detection model, we need some assets like some images for both train and test, image annotations, pre-trained object detection model, trained resources, and exported model to use for prediction. To keep things clear I create some directories. All the new folders are created inside the named **"training_demo"**.


```python
pwd
```




    '/home/ubuntu-20/Desktop/TensorFlow/models/research'



**"training_demo"** for organizing all the resources


```python
!mkdir ../../training_demo
```

**"annotations"** for ".record" and ".pbtxt" files. ".record" files are used as annotations and ".pbtxt" file for class names.


```python
!mkdir ../../training_demo/annotations
```

**"exported-models"** for final exported models. This model (.pb file and .config file is used for final prediction)


```python
!mkdir ../../training_demo/exported-models
```

**"images"** folder for store train and validation images


```python
!mkdir ../../training_demo/images
```


```python
!mkdir ../../training_demo/images/train
```


```python
!mkdir ../../training_demo/images/test
```

**"models"** directory to store models and training informations. 


```python
!mkdir ../../training_demo/models
```

**"pre-trained-models"** for storing "pre-trained-models" models. in our case "efficientdetD3"


```python
!mkdir ../../training_demo/pre-trained-models
```

### Download pre-trained model


```python
pwd
```




    '/home/ubuntu-20/Desktop/TensorFlow/models/research'



change directory to "pre-trained-models" (which I create earlier). Then downlaod pre-train model from tensorflow model zoo. To download any model go to this [GitHub repo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/tf2_detection_zoo.md). Copy any model's link address and download it using wget


```python
cd ../../training_demo/pre-trained-models
```

    /home/ubuntu-20/Desktop/TensorFlow/training_demo/pre-trained-models



```python
pwd
```




    '/home/ubuntu-20/Desktop/TensorFlow/training_demo/pre-trained-models'




```python
!wget http://download.tensorflow.org/models/object_detection/tf2/20200711/efficientdet_d3_coco17_tpu-32.tar.gz
```

    --2022-01-26 16:12:25--  http://download.tensorflow.org/models/object_detection/tf2/20200711/efficientdet_d3_coco17_tpu-32.tar.gz
    Resolving download.tensorflow.org (download.tensorflow.org)... 2404:6800:4004:810::2010, 172.217.174.112
    Connecting to download.tensorflow.org (download.tensorflow.org)|2404:6800:4004:810::2010|:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 92858658 (89M) [application/x-tar]
    Saving to: ???efficientdet_d3_coco17_tpu-32.tar.gz???
    
    efficientdet_d3_coc 100%[===================>]  88.56M  27.6MB/s    in 3.5s    
    
    2022-01-26 16:12:29 (25.6 MB/s) - ???efficientdet_d3_coco17_tpu-32.tar.gz??? saved [92858658/92858658]
    


unpack downloaded model inside "pre-trained-models"


```python
!tar -xvf efficientdet_d3_coco17_tpu-32.tar.gz
```

    efficientdet_d3_coco17_tpu-32/
    efficientdet_d3_coco17_tpu-32/checkpoint/
    efficientdet_d3_coco17_tpu-32/checkpoint/ckpt-0.data-00000-of-00001
    efficientdet_d3_coco17_tpu-32/checkpoint/checkpoint
    efficientdet_d3_coco17_tpu-32/checkpoint/ckpt-0.index
    efficientdet_d3_coco17_tpu-32/pipeline.config
    efficientdet_d3_coco17_tpu-32/saved_model/
    efficientdet_d3_coco17_tpu-32/saved_model/saved_model.pb
    efficientdet_d3_coco17_tpu-32/saved_model/assets/
    efficientdet_d3_coco17_tpu-32/saved_model/variables/
    efficientdet_d3_coco17_tpu-32/saved_model/variables/variables.data-00000-of-00001
    efficientdet_d3_coco17_tpu-32/saved_model/variables/variables.index



```python
pwd
```




    '/home/ubuntu-20/Desktop/TensorFlow/training_demo/pre-trained-models'



### Generate .record file

Now we have a pre-trained model. Let's create training and test files. We have some training images along with annotation XML files both in the train and test folder. There is a **"generate_tfrecord.py"** file inside the "training_demo" folder. This file can be used to generate train and test the ".record" file.<br>

simply call "generate_tfrecord.py" and pass train images folder path, label_map.pbtxt file path, and directory path to save train.record file path. Paths can be anywhere. it completely depends on you.


```python
cd ..
```

    /home/ubuntu-20/Desktop/TensorFlow/training_demo



```python
# Create train data:
!python generate_tfrecord.py -x images/train -l annotations/label_map.pbtxt -o annotations/train.record

#Create test data:
!python generate_tfrecord.py -x images/test -l annotations/label_map.pbtxt -o annotations/test.record
```

    Successfully created the TFRecord file: annotations/train.record
    Successfully created the TFRecord file: annotations/test.record


Now we have necessary training files (train.record, test.record and label_map.pbtxt)and necessary pre-traind models. One last thing is needed is to configer **"pipeline.config"** file. Inside this file we have to configer

- train.record file path
- test.record file path
- label_map.pbtxt file path
- batch_size
- number of classes
- fine_tune_checkpoint_type
- use_bfloat16
- fine_tune_checkpoint_type

You can find "pipeline.config" file inside the downloaded pre-trained model folder

To make everything clean and reusable, I coppied the "pipeline.config" file from "pre-trained-models/efficientdet_d3_coco17_tpu-32" to "exported_models/efficientdet" folder and edit it. Open "pipeline.config" and searche for 
1. num_classes
<pre>As we are focusing just on finding the object, our number of classes is 1. Now change num_classes to 1.</pre>
2. batch_size
<pre>Depending of the GPU, we can use any batch size. So use any number suitable for the training</pre>
3. total_steps/num_steps
<pre>Defines how long we like to train the model. Note that total_steps & num_steps are two different parameters. You have to assign the same value for both of them. Another important point is "warmup_steps". Both total_steps & num_steps must be higher than warmup_steps. Otherwise, you will get some errors.</pre>
4. fine_tune_checkpoint
<pre>You may find it just before the "num_steps" parameter. Basically, you have to specify the downloaded pre-trained model's checkpoint path to start training using that pre-trained model. Currently, we are using efficientdetD3. So, go to the pre-trained-models -> efficientdet_d3_coco17_tpu-32 -> checkpoint folder  and copy the relative path of "ckpt-0.index" and paste it here. As I am currently inside "training_demo" folder, so relative path from "training_demo" would be "./pre-trained-models/efficientdet_d3_coco17_tpu-32/checkpoint/ckpt-0". <strong>Note that, you have to remove the extension from "ckpt-0.index" file</strong>. The model already knows the file extension. <strong>BTW, all folder names are changeable. If you want to change the folder name or path, you can do that. Just make sure to specify the file path correctly</strong></pre>
5. fine_tune_checkpoint_type
<pre>Change fine_tune_checkpoint_type to <strong>"detection"</strong> as we are trying to detect objects.</pre>
6. use_bfloat16
<pre>Change "use_bfloat16" to <strong>flase</strong> as we are not using TPU's</pre>
7. train_input_reader
<pre>Inside "train_input_reader" section, you have to modify 2 parameters, "label_map_path" & "input_path".  The "label_map.pbtxt" can be found inside "annotations" folder (as i put it there. You can put it anywhere you like). As we are still inside "training_demo" folder, so relative path would be "./annotations/label_map.pbtxt".<br>
Next "input_path". As this "input_path" is inside "train_input_reader" section, so we have to specify traing input file which is <strong>train.record</strong> file. The "train.record" file can also be found inside "annotations" folder. As we are still inside "training_demo" folder, so relative path would be "./annotations/train.record".</pre>

8. eval_input_reader
<pre>Inside "eval_input_reader" section, you also have to modify 2 parameters, "label_map_path" & "input_path".  The "label_map.pbtxt" can be found inside "annotations" folder (as i put it there. You can put it anywhere you like). As we are still inside "training_demo" folder, so relative path would be "./annotations/label_map.pbtxt".<br>
Next "input_path". As this "input_path" is inside the "eval_input_reader" section, so we have to specify the training input file which is <strong>test.record</strong> file (I named validation set as test. You can use val or validation. Just make sure to change the path accordingly). The "test.record" file can also be found inside the "annotations" folder. As we are still inside the "training_demo" folder, so the relative path would be "./annotations/test.record".</pre>

### Training

Now we are done with all installation and configurations. Let's train the model. Currently we are inside **"training_demo"** folder. If you check, you will find a file named **"model_main_tf2.py"**. This is the file we are going to use to train the object detection model. This file is also provided by Tensorflow. Check out this file. There are also some configurations inside this file. You can change/pass the argument depending on your objective.


```python
pwd
```




    '/home/ubuntu-20/Desktop/TensorFlow/training_demo'




```python
ls
```

    [0m[01;34mannotations[0m/         export_tflite_graph_tf2.py  model_main_tf2.py
    [01;34mexported-models[0m/     generate_tfrecord.py        [01;34mmodels[0m/
    exporter_main_v2.py  [01;34mimages[0m/                     [01;34mpre-trained-models[0m/


To train the model, you have to pass two parameters, **"model_dir"** & **"pipeline_config_path"**. "model_dir" is the folder where you want to save your model training progress (this folder will be needed later for visualization of the training graphs) and "pipeline_config_path" is the folder where we want to save our trained model configuration. (Basically, the configuration file which will be saved in this training process is the same one we configured earlier. Just to make this separate, I saved this config file in the same folder where the trained models are going to save). 


```python
!python model_main_tf2.py --model_dir=models/EfficientDet-D3-custom-trained --pipeline_config_path=models/EfficientDet-D3-custom-trained/pipeline.config
```

    2022-01-26 19:18:16.924165: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:939] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
    2022-01-26 19:18:16.927108: W tensorflow/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libcusolver.so.11'; dlerror: libcusolver.so.11: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/cv2/../../lib64:/usr/local/cuda/lib64
    2022-01-26 19:18:16.927694: W tensorflow/core/common_runtime/gpu/gpu_device.cc:1850] Cannot dlopen some GPU libraries. Please make sure the missing libraries mentioned above are installed properly if you would like to use GPU. Follow the guide at https://www.tensorflow.org/install/gpu for how to download and setup the required libraries for your platform.
    Skipping registering GPU devices...
    2022-01-26 19:18:16.928442: I tensorflow/core/platform/cpu_feature_guard.cc:151] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
    To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
    WARNING:tensorflow:There are non-GPU devices in `tf.distribute.Strategy`, not using nccl allreduce.
    W0126 19:18:16.930691 139684560966016 cross_device_ops.py:1387] There are non-GPU devices in `tf.distribute.Strategy`, not using nccl allreduce.
    INFO:tensorflow:Using MirroredStrategy with devices ('/job:localhost/replica:0/task:0/device:CPU:0',)
    I0126 19:18:16.931914 139684560966016 mirrored_strategy.py:376] Using MirroredStrategy with devices ('/job:localhost/replica:0/task:0/device:CPU:0',)
    INFO:tensorflow:Maybe overwriting train_steps: None
    I0126 19:18:16.935007 139684560966016 config_util.py:552] Maybe overwriting train_steps: None
    INFO:tensorflow:Maybe overwriting use_bfloat16: False
    I0126 19:18:16.935164 139684560966016 config_util.py:552] Maybe overwriting use_bfloat16: False
    I0126 19:18:16.944221 139684560966016 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b3
    I0126 19:18:16.944335 139684560966016 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 160
    I0126 19:18:16.944365 139684560966016 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 6
    I0126 19:18:16.947594 139684560966016 efficientnet_model.py:147] round_filter input=32 output=40
    I0126 19:18:16.979430 139684560966016 efficientnet_model.py:147] round_filter input=32 output=40
    I0126 19:18:16.979545 139684560966016 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 19:18:17.123809 139684560966016 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 19:18:17.123928 139684560966016 efficientnet_model.py:147] round_filter input=24 output=32
    I0126 19:18:17.395811 139684560966016 efficientnet_model.py:147] round_filter input=24 output=32
    I0126 19:18:17.395925 139684560966016 efficientnet_model.py:147] round_filter input=40 output=48
    I0126 19:18:17.663366 139684560966016 efficientnet_model.py:147] round_filter input=40 output=48
    I0126 19:18:17.663570 139684560966016 efficientnet_model.py:147] round_filter input=80 output=96
    I0126 19:18:18.200055 139684560966016 efficientnet_model.py:147] round_filter input=80 output=96
    I0126 19:18:18.200171 139684560966016 efficientnet_model.py:147] round_filter input=112 output=136
    I0126 19:18:18.719847 139684560966016 efficientnet_model.py:147] round_filter input=112 output=136
    I0126 19:18:18.719960 139684560966016 efficientnet_model.py:147] round_filter input=192 output=232
    I0126 19:18:19.291749 139684560966016 efficientnet_model.py:147] round_filter input=192 output=232
    I0126 19:18:19.291912 139684560966016 efficientnet_model.py:147] round_filter input=320 output=384
    I0126 19:18:19.552504 139684560966016 efficientnet_model.py:147] round_filter input=1280 output=1536
    I0126 19:18:19.620378 139684560966016 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=1.2, depth_coefficient=1.4, resolution=300, dropout_rate=0.3, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/model_lib_v2.py:563: StrategyBase.experimental_distribute_datasets_from_function (from tensorflow.python.distribute.distribute_lib) is deprecated and will be removed in a future version.
    Instructions for updating:
    rename to distribute_datasets_from_function
    W0126 19:18:19.679734 139684560966016 deprecation.py:341] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/model_lib_v2.py:563: StrategyBase.experimental_distribute_datasets_from_function (from tensorflow.python.distribute.distribute_lib) is deprecated and will be removed in a future version.
    Instructions for updating:
    rename to distribute_datasets_from_function
    INFO:tensorflow:Reading unweighted datasets: ['./annotations/train.record']
    I0126 19:18:19.692135 139684560966016 dataset_builder.py:163] Reading unweighted datasets: ['./annotations/train.record']
    INFO:tensorflow:Reading record datasets for input file: ['./annotations/train.record']
    I0126 19:18:19.692332 139684560966016 dataset_builder.py:80] Reading record datasets for input file: ['./annotations/train.record']
    INFO:tensorflow:Number of filenames to read: 1
    I0126 19:18:19.692425 139684560966016 dataset_builder.py:81] Number of filenames to read: 1
    WARNING:tensorflow:num_readers has been reduced to 1 to match input file shards.
    W0126 19:18:19.692487 139684560966016 dataset_builder.py:87] num_readers has been reduced to 1 to match input file shards.
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/builders/dataset_builder.py:101: parallel_interleave (from tensorflow.python.data.experimental.ops.interleave_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.data.Dataset.interleave(map_func, cycle_length, block_length, num_parallel_calls=tf.data.AUTOTUNE)` instead. If sloppy execution is desired, use `tf.data.Options.deterministic`.
    W0126 19:18:19.694516 139684560966016 deprecation.py:341] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/builders/dataset_builder.py:101: parallel_interleave (from tensorflow.python.data.experimental.ops.interleave_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.data.Dataset.interleave(map_func, cycle_length, block_length, num_parallel_calls=tf.data.AUTOTUNE)` instead. If sloppy execution is desired, use `tf.data.Options.deterministic`.
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/builders/dataset_builder.py:236: DatasetV1.map_with_legacy_function (from tensorflow.python.data.ops.dataset_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.data.Dataset.map()
    W0126 19:18:19.851195 139684560966016 deprecation.py:341] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/builders/dataset_builder.py:236: DatasetV1.map_with_legacy_function (from tensorflow.python.data.ops.dataset_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.data.Dataset.map()
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/util/dispatch.py:1096: sparse_to_dense (from tensorflow.python.ops.sparse_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Create a `tf.sparse.SparseTensor` and use `tf.sparse.to_dense` instead.
    W0126 19:18:24.580934 139684560966016 deprecation.py:341] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/util/dispatch.py:1096: sparse_to_dense (from tensorflow.python.ops.sparse_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Create a `tf.sparse.SparseTensor` and use `tf.sparse.to_dense` instead.
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/autograph/impl/api.py:465: to_float (from tensorflow.python.ops.math_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.cast` instead.
    W0126 19:18:27.232981 139684560966016 deprecation.py:341] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/autograph/impl/api.py:465: to_float (from tensorflow.python.ops.math_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.cast` instead.
    2022-01-26 19:18:29.275193: W tensorflow/core/framework/dataset.cc:744] Input of GeneratorDatasetOp::Dataset will not be optimized because the dataset does not implement the AsGraphDefInternal() method needed to apply optimizations.
    2022-01-26 19:18:29.802818: W tensorflow/core/framework/op_kernel.cc:1745] OP_REQUIRES failed at multi_device_iterator_ops.cc:789 : NOT_FOUND: Resource AnonymousMultiDeviceIterator/AnonymousMultiDeviceIterator0/N10tensorflow4data12_GLOBAL__N_119MultiDeviceIteratorE does not exist.
    /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/keras/backend.py:414: UserWarning: `tf.keras.backend.set_learning_phase` is deprecated and will be removed after 2020-10-11. To update it, simply pass a True/False value to the `training` argument of the `__call__` method of your layer or model.
      warnings.warn('`tf.keras.backend.set_learning_phase` is deprecated and '
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/util/deprecation.py:620: calling map_fn_v2 (from tensorflow.python.ops.map_fn) with dtype is deprecated and will be removed in a future version.
    Instructions for updating:
    Use fn_output_signature instead
    W0126 19:18:58.426820 139676748863232 deprecation.py:545] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/util/deprecation.py:620: calling map_fn_v2 (from tensorflow.python.ops.map_fn) with dtype is deprecated and will be removed in a future version.
    Instructions for updating:
    Use fn_output_signature instead
    WARNING:tensorflow:Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    W0126 19:19:06.249039 139676748863232 utils.py:76] Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    WARNING:tensorflow:Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    W0126 19:19:17.842705 139676748863232 utils.py:76] Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    WARNING:tensorflow:Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    W0126 19:19:27.477190 139676748863232 utils.py:76] Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    WARNING:tensorflow:Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    W0126 19:19:38.556267 139676748863232 utils.py:76] Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    INFO:tensorflow:Step 100 per-step time 6.912s
    I0126 19:30:29.503887 139684560966016 model_lib_v2.py:705] Step 100 per-step time 6.912s
    INFO:tensorflow:{'Loss/classification_loss': 2.3749464,
     'Loss/localization_loss': 0.0,
     'Loss/regularization_loss': 0.038565468,
     'Loss/total_loss': 2.4135118,
     'learning_rate': 0.0}
    I0126 19:30:29.526928 139684560966016 model_lib_v2.py:708] {'Loss/classification_loss': 2.3749464,
     'Loss/localization_loss': 0.0,
     'Loss/regularization_loss': 0.038565468,
     'Loss/total_loss': 2.4135118,
     'learning_rate': 0.0}


If you to train on multiple GPU, then you have to add **"num_clones"**. "num_clones" defines how many GPUs you like to use. I have only 1 GPU. So i assign **--num_clones=1**


```python
!python model_main_tf2.py  --model_dir=models/EfficientDet-D3-custom-trained --pipeline_config_path=models/EfficientDet-D3-custom-trained/pipeline.config --num_clones=1 --ps_tasks=1
```

    2022-01-26 19:51:11.329579: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:939] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
    2022-01-26 19:51:11.332355: W tensorflow/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libcusolver.so.11'; dlerror: libcusolver.so.11: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/cv2/../../lib64:/usr/local/cuda/lib64
    2022-01-26 19:51:11.332701: W tensorflow/core/common_runtime/gpu/gpu_device.cc:1850] Cannot dlopen some GPU libraries. Please make sure the missing libraries mentioned above are installed properly if you would like to use GPU. Follow the guide at https://www.tensorflow.org/install/gpu for how to download and setup the required libraries for your platform.
    Skipping registering GPU devices...
    2022-01-26 19:51:11.333297: I tensorflow/core/platform/cpu_feature_guard.cc:151] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
    To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
    WARNING:tensorflow:There are non-GPU devices in `tf.distribute.Strategy`, not using nccl allreduce.
    W0126 19:51:11.335279 140671546012032 cross_device_ops.py:1387] There are non-GPU devices in `tf.distribute.Strategy`, not using nccl allreduce.
    INFO:tensorflow:Using MirroredStrategy with devices ('/job:localhost/replica:0/task:0/device:CPU:0',)
    I0126 19:51:11.336194 140671546012032 mirrored_strategy.py:376] Using MirroredStrategy with devices ('/job:localhost/replica:0/task:0/device:CPU:0',)
    INFO:tensorflow:Maybe overwriting train_steps: None
    I0126 19:51:11.338904 140671546012032 config_util.py:552] Maybe overwriting train_steps: None
    INFO:tensorflow:Maybe overwriting use_bfloat16: False
    I0126 19:51:11.339073 140671546012032 config_util.py:552] Maybe overwriting use_bfloat16: False
    I0126 19:51:11.347533 140671546012032 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b3
    I0126 19:51:11.347657 140671546012032 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 160
    I0126 19:51:11.347728 140671546012032 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 6
    I0126 19:51:11.350413 140671546012032 efficientnet_model.py:147] round_filter input=32 output=40
    I0126 19:51:11.381894 140671546012032 efficientnet_model.py:147] round_filter input=32 output=40
    I0126 19:51:11.382004 140671546012032 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 19:51:11.517031 140671546012032 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 19:51:11.517138 140671546012032 efficientnet_model.py:147] round_filter input=24 output=32
    I0126 19:51:11.772451 140671546012032 efficientnet_model.py:147] round_filter input=24 output=32
    I0126 19:51:11.772562 140671546012032 efficientnet_model.py:147] round_filter input=40 output=48
    I0126 19:51:12.061040 140671546012032 efficientnet_model.py:147] round_filter input=40 output=48
    I0126 19:51:12.061159 140671546012032 efficientnet_model.py:147] round_filter input=80 output=96
    I0126 19:51:12.604596 140671546012032 efficientnet_model.py:147] round_filter input=80 output=96
    I0126 19:51:12.604715 140671546012032 efficientnet_model.py:147] round_filter input=112 output=136
    I0126 19:51:13.175338 140671546012032 efficientnet_model.py:147] round_filter input=112 output=136
    I0126 19:51:13.175463 140671546012032 efficientnet_model.py:147] round_filter input=192 output=232
    I0126 19:51:13.943156 140671546012032 efficientnet_model.py:147] round_filter input=192 output=232
    I0126 19:51:13.943384 140671546012032 efficientnet_model.py:147] round_filter input=320 output=384
    I0126 19:51:14.294837 140671546012032 efficientnet_model.py:147] round_filter input=1280 output=1536
    I0126 19:51:14.345150 140671546012032 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=1.2, depth_coefficient=1.4, resolution=300, dropout_rate=0.3, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/model_lib_v2.py:563: StrategyBase.experimental_distribute_datasets_from_function (from tensorflow.python.distribute.distribute_lib) is deprecated and will be removed in a future version.
    Instructions for updating:
    rename to distribute_datasets_from_function
    W0126 19:51:14.389445 140671546012032 deprecation.py:341] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/model_lib_v2.py:563: StrategyBase.experimental_distribute_datasets_from_function (from tensorflow.python.distribute.distribute_lib) is deprecated and will be removed in a future version.
    Instructions for updating:
    rename to distribute_datasets_from_function
    INFO:tensorflow:Reading unweighted datasets: ['./annotations/train.record']
    I0126 19:51:14.401006 140671546012032 dataset_builder.py:163] Reading unweighted datasets: ['./annotations/train.record']
    INFO:tensorflow:Reading record datasets for input file: ['./annotations/train.record']
    I0126 19:51:14.401212 140671546012032 dataset_builder.py:80] Reading record datasets for input file: ['./annotations/train.record']
    INFO:tensorflow:Number of filenames to read: 1
    I0126 19:51:14.401286 140671546012032 dataset_builder.py:81] Number of filenames to read: 1
    WARNING:tensorflow:num_readers has been reduced to 1 to match input file shards.
    W0126 19:51:14.401366 140671546012032 dataset_builder.py:87] num_readers has been reduced to 1 to match input file shards.
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/builders/dataset_builder.py:101: parallel_interleave (from tensorflow.python.data.experimental.ops.interleave_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.data.Dataset.interleave(map_func, cycle_length, block_length, num_parallel_calls=tf.data.AUTOTUNE)` instead. If sloppy execution is desired, use `tf.data.Options.deterministic`.
    W0126 19:51:14.403273 140671546012032 deprecation.py:341] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/builders/dataset_builder.py:101: parallel_interleave (from tensorflow.python.data.experimental.ops.interleave_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.data.Dataset.interleave(map_func, cycle_length, block_length, num_parallel_calls=tf.data.AUTOTUNE)` instead. If sloppy execution is desired, use `tf.data.Options.deterministic`.
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/builders/dataset_builder.py:236: DatasetV1.map_with_legacy_function (from tensorflow.python.data.ops.dataset_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.data.Dataset.map()
    W0126 19:51:14.562267 140671546012032 deprecation.py:341] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/object_detection/builders/dataset_builder.py:236: DatasetV1.map_with_legacy_function (from tensorflow.python.data.ops.dataset_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.data.Dataset.map()
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/util/dispatch.py:1096: sparse_to_dense (from tensorflow.python.ops.sparse_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Create a `tf.sparse.SparseTensor` and use `tf.sparse.to_dense` instead.
    W0126 19:51:19.616015 140671546012032 deprecation.py:341] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/util/dispatch.py:1096: sparse_to_dense (from tensorflow.python.ops.sparse_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Create a `tf.sparse.SparseTensor` and use `tf.sparse.to_dense` instead.
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/autograph/impl/api.py:465: to_float (from tensorflow.python.ops.math_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.cast` instead.
    W0126 19:51:22.496867 140671546012032 deprecation.py:341] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/autograph/impl/api.py:465: to_float (from tensorflow.python.ops.math_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Use `tf.cast` instead.
    2022-01-26 19:51:24.571085: W tensorflow/core/framework/dataset.cc:744] Input of GeneratorDatasetOp::Dataset will not be optimized because the dataset does not implement the AsGraphDefInternal() method needed to apply optimizations.
    /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/keras/backend.py:414: UserWarning: `tf.keras.backend.set_learning_phase` is deprecated and will be removed after 2020-10-11. To update it, simply pass a True/False value to the `training` argument of the `__call__` method of your layer or model.
      warnings.warn('`tf.keras.backend.set_learning_phase` is deprecated and '
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/util/deprecation.py:620: calling map_fn_v2 (from tensorflow.python.ops.map_fn) with dtype is deprecated and will be removed in a future version.
    Instructions for updating:
    Use fn_output_signature instead
    W0126 19:51:55.055413 140663467276032 deprecation.py:545] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/util/deprecation.py:620: calling map_fn_v2 (from tensorflow.python.ops.map_fn) with dtype is deprecated and will be removed in a future version.
    Instructions for updating:
    Use fn_output_signature instead
    WARNING:tensorflow:Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    W0126 19:52:03.805298 140663467276032 utils.py:76] Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    WARNING:tensorflow:Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    W0126 19:52:17.698312 140663467276032 utils.py:76] Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    WARNING:tensorflow:Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    W0126 19:52:28.856673 140663467276032 utils.py:76] Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    WARNING:tensorflow:Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    W0126 19:52:40.214063 140663467276032 utils.py:76] Gradients do not exist for variables ['stack_6/block_1/expand_bn/gamma:0', 'stack_6/block_1/expand_bn/beta:0', 'stack_6/block_1/depthwise_conv2d/depthwise_kernel:0', 'stack_6/block_1/depthwise_bn/gamma:0', 'stack_6/block_1/depthwise_bn/beta:0', 'stack_6/block_1/project_bn/gamma:0', 'stack_6/block_1/project_bn/beta:0', 'top_bn/gamma:0', 'top_bn/beta:0'] when minimizing the loss. If you're using `model.compile()`, did you forget to provide a `loss`argument?
    INFO:tensorflow:Step 100 per-step time 6.888s
    I0126 20:03:23.654749 140671546012032 model_lib_v2.py:705] Step 100 per-step time 6.888s
    INFO:tensorflow:{'Loss/classification_loss': 1.0755744,
     'Loss/localization_loss': 0.024037467,
     'Loss/regularization_loss': 0.038565367,
     'Loss/total_loss': 1.1381773,
     'learning_rate': 0.0}
    I0126 20:03:23.673841 140671546012032 model_lib_v2.py:708] {'Loss/classification_loss': 1.0755744,
     'Loss/localization_loss': 0.024037467,
     'Loss/regularization_loss': 0.038565367,
     'Loss/total_loss': 1.1381773,
     'learning_rate': 0.0}



```python
pwd
```




    '/home/ubuntu-20/Desktop/TensorFlow/training_demo'



# Visualizing the training process

Tensorflow has a nice tool to visulize the praining progress named "Tensorbord". This tool is automatically installed when you install tensorflow using the conda comand. To visulize training progress: <br>

- Open a new terminal.
- activate the virtual environment by trping **source activate tensorflow** (which we have done before in a different terminal)
- navigate to **"training_demo"** folder
- now type **tensorboard --logdir=models/EfficientDet-D3-custom-trained** as we saved our trained network in "EfficientDet-D3-custom-trained" folder.

Once this is done, terminal will show you something like this **"TensorBoard 2.6.0 at http://localhost:6006/ (Press CTRL+C to quit)"**<br>

Go to **http://localhost:6006/** and you can see different graphs related to your training progress.(Port number might be different)

# Export the trained model

We already trained our model. But Tensorflow Object Detection API didn't save the final model. Instead, it saves the checkpoints after every 1000 steps(you can change this step number from model_main_tf2.py). Now we have to export the final model from these checkpoints. <br> <br>
**Note:** the benefit of having these checkpoint files is, you can restart your training process from any of these checkpoint files later if you want.<br><br>
Now to export the final model, we will use "exporter_main_v2.py". This file is also provided by Tensorflow.<br><br>

Execute the following command with appropriate parameters. We already discussed all the folders necessary for this command.


```python
!python exporter_main_v2.py --input_type image_tensor --pipeline_config_path models/EfficientDet-D3-custom-trained/pipeline.config --trained_checkpoint_dir models/EfficientDet-D3-custom-trained --output_directory exported_models/efficientdet
```

    2022-01-26 19:38:08.131745: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:939] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
    2022-01-26 19:38:08.134366: W tensorflow/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libcusolver.so.11'; dlerror: libcusolver.so.11: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: /usr/local/cuda/lib64
    2022-01-26 19:38:08.134966: W tensorflow/core/common_runtime/gpu/gpu_device.cc:1850] Cannot dlopen some GPU libraries. Please make sure the missing libraries mentioned above are installed properly if you would like to use GPU. Follow the guide at https://www.tensorflow.org/install/gpu for how to download and setup the required libraries for your platform.
    Skipping registering GPU devices...
    2022-01-26 19:38:08.140429: I tensorflow/core/platform/cpu_feature_guard.cc:151] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
    To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
    I0126 19:38:08.147075 140401355592064 ssd_efficientnet_bifpn_feature_extractor.py:145] EfficientDet EfficientNet backbone version: efficientnet-b3
    I0126 19:38:08.147200 140401355592064 ssd_efficientnet_bifpn_feature_extractor.py:147] EfficientDet BiFPN num filters: 160
    I0126 19:38:08.147293 140401355592064 ssd_efficientnet_bifpn_feature_extractor.py:148] EfficientDet BiFPN num iterations: 6
    I0126 19:38:08.150027 140401355592064 efficientnet_model.py:147] round_filter input=32 output=40
    I0126 19:38:08.166386 140401355592064 efficientnet_model.py:147] round_filter input=32 output=40
    I0126 19:38:08.166506 140401355592064 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 19:38:08.273832 140401355592064 efficientnet_model.py:147] round_filter input=16 output=24
    I0126 19:38:08.273949 140401355592064 efficientnet_model.py:147] round_filter input=24 output=32
    I0126 19:38:08.459990 140401355592064 efficientnet_model.py:147] round_filter input=24 output=32
    I0126 19:38:08.460103 140401355592064 efficientnet_model.py:147] round_filter input=40 output=48
    I0126 19:38:08.650922 140401355592064 efficientnet_model.py:147] round_filter input=40 output=48
    I0126 19:38:08.651039 140401355592064 efficientnet_model.py:147] round_filter input=80 output=96
    I0126 19:38:08.969503 140401355592064 efficientnet_model.py:147] round_filter input=80 output=96
    I0126 19:38:08.969624 140401355592064 efficientnet_model.py:147] round_filter input=112 output=136
    I0126 19:38:09.477416 140401355592064 efficientnet_model.py:147] round_filter input=112 output=136
    I0126 19:38:09.477530 140401355592064 efficientnet_model.py:147] round_filter input=192 output=232
    I0126 19:38:09.952585 140401355592064 efficientnet_model.py:147] round_filter input=192 output=232
    I0126 19:38:09.952699 140401355592064 efficientnet_model.py:147] round_filter input=320 output=384
    I0126 19:38:10.110199 140401355592064 efficientnet_model.py:147] round_filter input=1280 output=1536
    I0126 19:38:10.148629 140401355592064 efficientnet_model.py:457] Building model efficientnet with params ModelConfig(width_coefficient=1.2, depth_coefficient=1.4, resolution=300, dropout_rate=0.3, blocks=(BlockConfig(input_filters=32, output_filters=16, kernel_size=3, num_repeat=1, expand_ratio=1, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=16, output_filters=24, kernel_size=3, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=24, output_filters=40, kernel_size=5, num_repeat=2, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=40, output_filters=80, kernel_size=3, num_repeat=3, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=80, output_filters=112, kernel_size=5, num_repeat=3, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=112, output_filters=192, kernel_size=5, num_repeat=4, expand_ratio=6, strides=(2, 2), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise'), BlockConfig(input_filters=192, output_filters=320, kernel_size=3, num_repeat=1, expand_ratio=6, strides=(1, 1), se_ratio=0.25, id_skip=True, fused_conv=False, conv_type='depthwise')), stem_base_filters=32, top_base_filters=1280, activation='simple_swish', batch_norm='default', bn_momentum=0.99, bn_epsilon=0.001, weight_decay=5e-06, drop_connect_rate=0.2, depth_divisor=8, min_depth=None, use_se=True, input_channels=3, num_classes=1000, model_name='efficientnet', rescale_input=False, data_format='channels_last', dtype='float32')
    WARNING:tensorflow:From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/autograph/impl/api.py:464: calling map_fn_v2 (from tensorflow.python.ops.map_fn) with back_prop=False is deprecated and will be removed in a future version.
    Instructions for updating:
    back_prop=False is deprecated. Consider using tf.stop_gradient instead.
    Instead of:
    results = tf.map_fn(fn, elems, back_prop=False)
    Use:
    results = tf.nest.map_structure(tf.stop_gradient, tf.map_fn(fn, elems))
    W0126 19:38:13.148314 140401355592064 deprecation.py:614] From /home/ubuntu-20/anaconda3/envs/tensorflow/lib/python3.9/site-packages/tensorflow/python/autograph/impl/api.py:464: calling map_fn_v2 (from tensorflow.python.ops.map_fn) with back_prop=False is deprecated and will be removed in a future version.
    Instructions for updating:
    back_prop=False is deprecated. Consider using tf.stop_gradient instead.
    Instead of:
    results = tf.map_fn(fn, elems, back_prop=False)
    Use:
    results = tf.nest.map_structure(tf.stop_gradient, tf.map_fn(fn, elems))
    WARNING:tensorflow:Skipping full serialization of Keras layer <object_detection.meta_architectures.ssd_meta_arch.SSDMetaArch object at 0x7fb13074bca0>, because it is not built.
    W0126 19:38:28.202083 140401355592064 save_impl.py:71] Skipping full serialization of Keras layer <object_detection.meta_architectures.ssd_meta_arch.SSDMetaArch object at 0x7fb13074bca0>, because it is not built.
    2022-01-26 19:38:59.669341: W tensorflow/python/util/util.cc:368] Sets are not currently considered sequences, but this may change in the future, so consider avoiding using them.
    W0126 19:39:36.125263 140401355592064 save.py:263] Found untraced functions such as WeightSharedConvolutionalBoxPredictor_layer_call_fn, WeightSharedConvolutionalBoxPredictor_layer_call_and_return_conditional_losses, WeightSharedConvolutionalBoxHead_layer_call_fn, WeightSharedConvolutionalBoxHead_layer_call_and_return_conditional_losses, WeightSharedConvolutionalBoxPredictor_layer_call_fn while saving (showing 5 of 1315). These functions will not be directly callable after loading.
    INFO:tensorflow:Assets written to: exported_models/efficientdet/saved_model/assets
    I0126 19:39:58.307252 140401355592064 builder_impl.py:783] Assets written to: exported_models/efficientdet/saved_model/assets
    INFO:tensorflow:Writing pipeline config file to exported_models/efficientdet/pipeline.config
    I0126 19:39:59.810654 140401355592064 config_util.py:253] Writing pipeline config file to exported_models/efficientdet/pipeline.config



```python

```
