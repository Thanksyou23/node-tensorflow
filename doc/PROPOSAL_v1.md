# PROPOSAL

Hello World! We are the guys embracing Node.js and Tensorflow.
This is the proposal and guideline to build Node addon for Tensorflow.
Do not be hesitated to create an issue if you encounter problems.

## I. INTRODUCTION

### Tensorflow is an open source library for numerical computation using data flow graphs

Tensorflow was released in autumn 2015 with Apache 2.0 open source license. The big picture of Tensorflow is composed by 1) C++ core library 2) SWIG interface for Python 3) Python binding generated by SWIG 4) Python API for building graph. The workflow is clear as follow:

C++ core library <-> SWIG <-> Python binding <-> Python API <-> End User

### Node-tensorflow is a NodeJS API for utilizing Google's machine learning library TensorFlow

We are going to make a Node.js wrapper instead of reinventing the wheel. The proposed approach is very similar to what Tensorflow does. Therefore the end users play around Tensorflow using Node.js.

C++ core library <-> node-gyp <-> Node binding <-> Node API <-> End User

We currently try to define some guidelines, so please be patience to read through this white paper. Don't be hesitated to create an issue if you have any question/suggestion.

## II. INSTALLATION & REPOSITORY STRUCTURE

You can now install the development environment using NPM directly.

`npm install -g tensorflowjs`

Note that we execute a bash script under `tools/install.sh`, that **should** install all dependencies required. Be aware that this hasn't been tested on multiple enviroments. The tested enviroments are:

- Mac OSX [10.11.2]
- Docker with Ubuntu 14.04
- *Have you got problems/success? Let us know!*

It will take 10-20 minutes if it's the first time installation, because we do compile TensorFlow from Source. The following is the structure of this repository:

```
src/              
tools/	           
.dockerignore	    
.gitignore	      
Dockerfile        Docker file for installation
LICENSE	          License statement
README.md	        
binding.gyp	      Module configuration in JSON format
index.js	        Entry file for Node API
package.json      
```

## III. NODE ADDON

C++ core library <-> node-gyp <-> Node binding

`node-gyp` is a tool to build native Node.js addon. We use it to build the Node.js addon from the Tensorflow core library which is written in C++. A binding.gyp file is created in module's root directory which is JSON format, describes the configuration to build our module. Once `install.sh` is finished execution, the files of Tensorflow core library are located in `src/lib`. We also suggest to use `nan` here, which is a Native abstractions for Node.js.

```
{                                                                    
  'targets': [{
      'target_name': 'node-tensorflow',
      'sources': ['src/api.cc'],

      'libraries' : [
          "<!@(pkg-config --libs protobuf)",
          "<!@(find $(pwd)/src/lib -iname \*.o)",
      ],

      'include_dirs' : [
          # All c++ dependencies goes here
          "src/includes",

          # Third party must be included as root as well
          "src/includes/third_party/eigen3",
          "src/includes/third_party/gpus",

          # Include NAN
          "<!(node -e \"require('nan')\")"
      ],

      "conditions": [
          [ "OS==\"mac\"", {
              "xcode_settings": {
                  "OTHER_CFLAGS": [
                      "-mmacosx-version-min=10.7",
                      "-std=c++",
                      "-stdlib=libc++",
                      "<!@(pkg-config --cflags protobuf)",
                  ],
                  "OTHER_LDFLAGS": [
                      '-stdlib=libc++',
                  ],
                  "OTHER_CPLUSPLUSFLAGS": [
                      '-std=c++11',
                      '-stdlib=libc++',
                      '-v'
                  ],
                  "GCC_ENABLE_CPP_RTTI": "YES",
                  "GCC_ENABLE_CPP_EXCEPTIONS": "YES",
                  'MACOSX_DEPLOYMENT_TARGET': '10.9',
              },
          }]
      ],
      'cflags': [
          "<!@(pkg-config --cflags protobuf)",
      ],
      "cflags!": [
          "-fno-exceptions"
      ],
      'cflags_cc!': [
          "-fno-rtti", "-fno-exceptions"
      ]
  }]
}
```

For the first commit, we have created three methods in `src/api.cc`. They are respectively:

+ print
+ version
+ test

`print` is a function printing a string "This is a sample Node.js addon" on your screen.<br/>
`version` is a function printing the current version of Tensorflow.<br/>
`test` is a function to do the Image Recognition Demo (label_image) in [here](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/label_image) <br/>

Run the following commands to generate the build files for current platform of your local machine, and build the addon.

```
$ node-gyp configure
$ node-gyp build
```

We can now invoke these sample functions using Node.js. `index.js` is the entry file of our module. 

```
var bindings = require('bindings');
var addon = bindings('node-tensorflow');
addon.print();
addon.version();                               
addon.test();
```

The results should be as follow:

```
This is a sample Node.js addon
0.5.0

Running Test
I tensorflow/examples/label_image/main.cc:200] military uniform (866): 0.902268
I tensorflow/examples/label_image/main.cc:200] bow tie (817): 0.05407
I tensorflow/examples/label_image/main.cc:200] suit (794): 0.0113195
I tensorflow/examples/label_image/main.cc:200] bulletproof vest (833): 0.0100269
I tensorflow/examples/label_image/main.cc:200] bearskin (849): 0.00649746
```

See [Section VI](#v-next-step) for the next step we are looking for.

## IV. NODE API

Node binding <-> Node API <-> End User

In Tensorflow, there is a Python API to build the computation graph. See [this](https://www.tensorflow.org/versions/master/api_docs/cc/index.html). Python API also provides Optimizers, Tensor Transformations, Type Casting, and some other useful features (see [doc](https://www.tensorflow.org/versions/master/api_docs/python/array_ops.html)). 

## V. NEXT STEP

Build a Node API that wraps [C++ API](https://tensorflow.googlesource.com/tensorflow/+/master/tensorflow/g3doc/api_docs/cc/index.md) defined in official Tensorflow. The next step is to wrap the C++ header files in the folder `/src/includes/tensorflow/core/public/`, or you can search those files under `tensorflow/core/public/`. Then we wrap the C++ API `/src/includes/tensorflow/core/client/tensor_c_api.cc`, or you can watch `tensorflow/core/client/tensor_c_api.cc` in official Tensorflow repository. @jagandecapri have already wrapped the `TF_NewsStatus` in `tensor_c_api.h`. You can take a look in his [tensorflow-test](https://github.com/jagandecapri/tensorflow-test).

Feel free to create a pull request =]

## VI. REFERENCE

#### Inspiration and Ideas

+ Installation process was inspired by `node-canvas` and `node-opencv`, both using native `.gyp` bindings from
  existent c++ code. Part of their work was used in the `bindings.gyp` and `install.sh`
