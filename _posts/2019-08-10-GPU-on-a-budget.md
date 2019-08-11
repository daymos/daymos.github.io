---
layout: post
title:  "GPU for Machine Learning on a 30 dollars budget"
date:   2019-08-03 13:08:15 +0100
comments: true
categories: Machine Learning
---

![image](/assets/images/nvidia-quadro.png)

Have you ever been in a situation where you need to train a Neural Network for production and you local machine is just too slow?

I have recently, and found a really nice way to get access one of the best  GPU around, for less then 30 bucks.

Initially I thought of a couple of options: 
 - Buy a GPU, I always wanted one anyways. 
 - Use Google Colab free GPUs
 - Setup a server with a GPU


## Buy a GPU

Now, who doesn't want a GPU in the end. I looked at a NVIDIA QUADRO RTX 6000, really powerful, really fast, and really expensive, about 4k UK pounds at the time I write this. On top of that, you need time to order it receive it and set it up.

But the real deal-breaker for me was, how often am I going to use it? I don't train Neural Networks everyday for production, and by the time I'll do it again my hardware might be outdated. So let's move to the next option. 


## Google Colab free GPU/TPU

So Google Colab is really cool, and I might write a dedicated post about it soon. For the time being, suffice it to say you can use their Jupyter like interface with a GPU for free. There is no setup time, it works really well, and I was already familiar with it, since I have used it before. So I actually tried this option.

The short story is that the training was terminated by Colab. I tried few times, but it kept being terminated. I then read the user agreement, and in fact they specify you should not use it for production tasks. Sorry Google I didn't know it!

(Ah if you are wondering what a TPU is, is a dedicated GPU specifically developed by Google to train Neural Networks.)


## Dedicated Server for just the time you need

This ended up being the winner. It costed me 30 bucks, a couple of hours to set up, and if I need to re-spawn it, its going to take me about 20 minutes. This is the in-detail explanation of how to do it, and how to overcome the few gotchas. 


### What server to use

There is a lots of great companies out there and you can use any. I work everyday for my job with AWS, Heroku and Google Clouds and they are great. But for this task I went for a different company: [linode](linode.com).

They only do servers, they are specialized and the service is just simple, no magic no learning curve, just a server, as you want it, with the bare minimum.

If you are a beginner, you don't even need to ssh into the machine as they provide a console in the browser. At last, they have fair prices for the hardware, unlike AWS, which offers a similar service with AWS Sagemaker, but at a really high price.  

![image](/assets/images/linode-prices.png)

These machines are pricey, so you need to make sure you keep them only for the time you need.

Big gotcha is, they bill you not just for the time you keep the machine up, but for the time they are in existence. That is because these are dedicated machines, and even if you power them down, they are still unavailable to other customers.

 If you are think how to avoid setting up everything every time, Linode let you use docker images.

I chose the smallest as it was fine for my needs. What you need to do next is to install the driver for the GPU. [This](https://www.pugetsystems.com/labs/hpc/How-to-install-CUDA-9-2-on-Ubuntu-18-04-1184/) is an amazing post that goes in depth, but frankly is quite easy and I'll show you the short version here. There are the steps you need to do:
    * get specs on the machine
    * install CUDA ( nvidia drivers ) dependencies
    * install CUDA


#### Get machine specs

Open a terminal in the new machine
```
lscpu
```

![image](/assets/images/cpu_architecture.png)
In my case is x86_64. Go to  [the nvdia download page](https://developer.nvidia.com/cuda-downloads) and select all the options. In my case it looked like that: 

![nvidia options](/assets/images/version_select.png)

The last option is just how you are going to run it.
You can see that the file is quite big, so I suggest you to download it directly from the cli of the instance.

Right click on the download button, select copy link address, then:
```
wget https://developer.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.168_418.67_linux.run
```
Note that the above command will only work you are using my exact version of cuda. 

![nvidia options](/assets/images/download_cuda.png)

2.5 Gb in 10 seconds at 251MB/s, isn't that beautiful ?

Now before you install it, add the dependancies of
```
sudo apt-get install build-essential
```

Now run: 
```
sudo sh cuda_10.1.168_418.67_linux.run
```
The CLI program that appears is a bit slow but just be patient. Answer the questions and once it is installed verify the installation with this: 
```
nvcc -V
```

## Verify your code is running on the GPU
Now the machine is setup and you can run your program. If you want to monitor the usage of the GPU type.

```
nvidia-smi
```
NOTE:
Depending on what software you are using, you will likey need to explicity tell your code to look for a GPU. 

In my case it looked like that, so I could see there was one process using the GPU:

![nvidia options](/assets/images/gpu-usage-monitor-tool.png)

## Performance and cost
The performance was astonishing for me. My neural network was taking about one hour per EPOCH on my laptop CPU. I wanted to run about 40 of them, which was very unpractical. 

On the GPU it took just a couple of minutes per EPOCH. So I was able to do my training in one hour. After which I saved the trained algo on my machine ( the checkpoints in my case ) , and tore down the server.

The total cost of the operation was 27$. Pretty great for a GPU that cost 4k UK pounds. 

## How to make this practical for reuse

Best practice would be to create a docker image. Linode allows you to pass a link to the image from the GUI when you initialize the machine. 

I was in a hurry and went for a less optimal but faster solution, which is apt-clone. 

