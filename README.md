# Setup:

To have the taste of both local system and dockers, the whole setup is divided into 2 parts.

## Section: 1. Virtual env. 
A virtual Env for running the jupyter notebook locally

## Section: 2. Use Dockers as microservice used for:

a. Running the tensorboard to see the training metrics.

b. Running TF serving for serving the model as endpoint.

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

### 1. Virtual Env.

a. Create virtual env with below command and activate it:

`virtualenv challenge-fashion`

`source challenge-fashion/bin/activate`

b.Install all the dependencies 

`pip install -r requirements.txt`

c.Run jupyter notebook server

`jupyter notebook`

d. Open the notebook 'rest_simple.ipynb' from the server url http://localhost:8888/tree

e. Please follow the guidelines to reproduce the results in the notebook.


+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ 
  

### 2. Docker environment for tensorboard and TF serving.

#### A. For Tensorboard:

a. Install docker env in your local machine.please follow steps here https://docs.docker.com/get-docker/ (I'm using Ubuntu 18.04)

b. RUN below command to download tensorflow container Image:

`sudo docker pull tensorflow/tensorflow`

c. RUN the below command where it maps the 'challenge-fashion' folder on your local machine to the docker volume. 
In our case assuming we're in 'challenge-fashion' folder (virtual env we created intially) which looks like below:

```
srauniy@sonu:~/challenge-fashion$ echo $PWD
~/challenge-fashion
```
```
srauniy@sonu:~/challenge-fashion$ ls -l
total 160
drwxrwxr-x 6 sonu sonu   4096 Dec 16 13:51 fashion-classification
-rw-rw-r-- 1 sonu sonu   9586 Dec 20 12:03 image.json
drwxr-xr-x 6 sonu sonu   4096 Dec 20 12:42 logs
-rw-rw-r-- 1 sonu sonu      0 Dec 20 12:31 README.md
-rw-rw-r-- 1 sonu sonu 136625 Dec 20 12:54 rest_simple.ipynb
drwxr-xr-x 3 sonu sonu   4096 Dec 19 22:45 saved_models
```

`sudo docker run -it -p 6006:6006 -v "$PWD:/tensorboard" --entrypoint /bin/bash tensorflow/tensorflow`

Now inside the container goto tensorboard folder

`cd tensorboard/` 

`tensorboard --logdir logs --host 0.0.0.0`

This will start tensorboard for monitoring metrics on the url `0.0.0.0:6006` accessible on your local system browser

#### B. For serving our Best model (model-2), please use below steps:
  
a. Pull serving image from dockerhub: 

`sudo docker pull tensorflow/serving`


b. RUN the below command where it maps the model saved path on your local machine to the docker volume. 
In our case assuming your're in challenge-fashion folder (virtual env we created intially).

`sudo docker run -it -p 8605:8605 -v "$PWD:/mnist-fashion" --entrypoint /bin/bash tensorflow/serving`

Here we are mapping host machine port 8605 to container service port 8605 and using the entry point '/bin/bash' to get into container tensorflow/serving.

c. Once inside the container RUN below command for serving the model .Let's call the model mnist_model

`tensorflow_model_server --rest_api_port=8605 --model_base_path=/mnist-fashion/saved_models/ --model_name=mnist_model`

d. Now the model is being served on your local machine at : http://localhost:8605/v1/models/mnist_model
```

{
 "model_version_status": [
  {
   "version": "1",
   "state": "AVAILABLE",
   "status": {
    "error_code": "OK",
    "error_message": ""
   }
  }
 ]
}
```

e. Use the curl command to make a request for predictions:
```
srauniy@sonu:~/challenge-fashion$ curl -X POST -H 'content-type: application/json' -d "@image.json" http://localhost:8605/v1/models/mnist_model:predict
{
    "predictions": [[9.35298e-10, 1.94899985e-10, 2.85322765e-09, 7.65899622e-10, 8.67189653e-10, 3.46909e-07, 1.84958282e-09, 3.66329623e-05, 1.10681775e-08, 0.999963045]
    ]
}
```
The actual and predicted label is Ankle boot (class 9)

e. Please follow notebook for more predictions examples.


