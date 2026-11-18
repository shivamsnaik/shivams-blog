+++
date                = '2026-11-18T17:30:00+01:00'
draft               = false
title               = 'Colibri: Run LLMs on your Laptop without a GPU'
searchHidden        = false
comments            = false
TocOpen             = true
[cover]
  image = "colibri-cover.png"
  alt   = "Colibri AI: Run your favorite LLMs locally"

[ai]
  level        = 'assisted' #assisted or generated
  note         = 'Post reviewing and improvisation'
  model        = 'GLM 5.3 via Mistral AI'
  hiddenInList = true
+++

I remember 2022 — feeling smart about how neatly ChatGPT summarized my finances. I dumped every number in, without a single thought about where my data went. Today I hesitate before pasting anything confidential into a web chat, and every other week there's a story about your chats being used for training. 

{{<images 
    imagenames="chatbot-interaction.jpg"
    style="max-width: 350px;"
    captions="Interaction with a Chatbot: The concern regarding data privacy has increased over time." >}}

There was never a real alternative: even a quantized model like Qwen3.6-35B-A3B demanded a proper GPU with serious VRAM. Then an Instagram reel showed me [Colibri: Hierarchical inference engine](https://justvugg.github.io/colibri/), a 744-billion-parameter model running on a machine with 8GB RAM and no graphics card at all! Halfway through the reel, I had already opened the GitHub repo to find out if it was true.

## What's Colibri?
Colibri is an AI Engine designed to deploy [Mixture of Experts (MoE)](https://developer.nvidia.com/blog/dense-vs-moe-models-active-parameters-throughput-and-when-to-choose-each/) LLMs where the frequently used experts stay warm in RAM, while the rarely used ones stay cold on disk, streamed in only when the model asks for them. This is the trick: only the always-on dense part needs to fit in RAM, so it stays small enough for an ordinary laptop. Optionally, a GPU can also be specified to improve the experience. Let's now see how we set it up on our systems. 

{{<images 
    imagenames="qwen3.8-flash-computation.png" 
    style="max-width: 600px; border-radius: 0px;"
    captions="Computation of the current token/word on Qwen-3.8-Flash: MoE architecture utilizes only a small portion of the model, i.e., only 6 billion parameters out of 176 billion are being used for the word generation.">}}

## Prerequisites before we start
As per the official documentation, the engine can deploy LLMs on a laptop without a GPU. Nevertheless, it requires a minimum of 8GB of RAM and about 22GB of disk space for the smallest model. Honestly, the minimum requirement would suffice only if a VERY slow chatbot works for you. I tried it on my laptop and I'd say a minimum of 16GB RAM would provide a better experience.

Before I start, here are the specifications of my laptop, for anyone wanting to assess and compare their setup and the approximate result beforehand:

>[!SPECIFICATIONS]
|  **Description**  |            **Specification**           |
|:-----------------:|:--------------------------------------:|
|  **Laptop Name**  |       Lenovo Legion 5 Pro 16ACH6H      |
|      **CPU**      | AMD Ryzen 5 5600H with Radeon Graphics |
|      **RAM**      |                  16GB                  |
| **GPU Available** |                   ✅                   |
|    **GPU Name**   |   NVIDIA GeForce RTX 3060 Laptop GPU   |
|    **GPU Memory** |               6.4GB VRAM               |

## 744B parameter model possible on a 16GB RAM laptop?

In order to find out the Instagram reel's "744B on a Laptop with 8GB RAM without GPU" claim, I downloaded the engine on my system and verified the models that fit my system. Turns out, the biggest model my system currently allows is Qwen3.6-35B-A3B. The **744B** parameter models, i.e., **GLM 5.2** and **GLM 5.3**, are beyond reach due to insufficient storage space. The claim theoretically holds true here, although unverified. The following list gives a better picture:

{{< collapse summary="Every model against this machine, and why one does not fit">}}
```go
$ user@pop-os:~/Documents/Projects/colibri$ ./start-here.sh --list
colibri setup

 * qwen36-35b                    23 GB  tight      runs with a small expert cache (needs 10 GB, runs best with 20 GB)
   qwen3-coder-30b               19 GB  tight      runs with a small expert cache (needs 8 GB, runs best with 18 GB)
   deepseek-v4-flash-reap        85 GB  tight      runs with a small expert cache (needs 16 GB, runs best with 32 GB)
   deepseek-v4-flash            167 GB  tight      runs with a small expert cache (needs 16 GB, runs best with 32 GB)
   qwen-image-2.1                33 GB  tight      runs with a small expert cache (needs 12 GB, runs best with 18 GB)
   mimo-v2.6-flash              172 GB  needs-ram  needs 32 GB of RAM (dense part 24 GB plus the expert cache); this machine has 16.6 GB
   qwen38-flash-next            186 GB  needs-ram  needs 24 GB of RAM (dense part 8 GB plus the expert cache); this machine has 16.6 GB
   glm-5.3                      419 GB  needs-disk needs 430 GB free for the download; 177 GB free there
   glm-5.2                      429 GB  needs-disk needs 440 GB free for the download; 177 GB free there
   inkling                      514 GB  needs-ram  needs 120 GB of RAM (dense part 49.4 GB plus the expert cache); this machine has 16.6 GB
   mimo-v2.6-pro                564 GB  needs-ram  needs 54 GB of RAM (dense part 32.4 GB plus the expert cache); this machine has 16.6 GB
   kimi-k3                     1561 GB  needs-ram  needs 32 GB of RAM (dense part 20 GB plus the expert cache); this machine has 16.6 GB
  ```
{{< /collapse >}}

Although the claim cannot be verified, I proceeded with the installation process (more about installation guide: [here](https://github.com/JustVugg/colibri#get-started-in-one-step)). Below is the output during my setup:

{{< collapse summary="Execution output" >}}
```go 
user@pop-os:~/colibri$ ./start-here.sh --no-gpu
colibri setup
Your machine
  CPU     AMD Ryzen 5 5600H with Radeon Graphics, 6 cores (12 threads), AVX2
  RAM     16.6 GB (10.5 GB free now)
  Disk    267 GB free in ~/colibri-models
  GPU     NVIDIA GeForce RTX 3060 Laptop GPU via Vulkan 1.4.329 (discrete, 4.9 GB budget)
  GPU     NVIDIA GeForce RTX 3060 Laptop GPU (NVIDIA, 6.4 GB VRAM, compute 8.6, driver 595.104.02)
  System  Pop!_OS 24.04 LTS

Models that fit this machine
  (fits = the dense part, which always stays in RAM, plus a minimum expert cache fit in RAM, and the download fits on the disk)
   1) Qwen3.6-35B-A3B                   23 GB   runs from RAM         [recommended]
      general chat with thinking and tools; int4-gs64 container; runs with a small expert cache (needs 10 GB, runs best with 20 GB)
Choose a model [Enter = 1]:
```
>[!NOTE]
> **`--no-gpu`** flag configures the model to only use the CPU during execution.

{{< /collapse >}}

I chose the recommended option, i.e., Qwen3.6-35B-A3B (Size: 23GB). It's funny: the very model whose VRAM demands were always too much for my 6 GB GPU is now running from RAM, no GPU required.

## Results: Surprisingly good for a laptop

{{<images 
    imagenames="colibri-ui.png"
    style=""
    captions="Colibri-UI: Chat with the deployed model with a familiar Agent UI. Model information is on the top left corner, and current state of the engine on the top-right.">}}
  
The initial impression is honestly great. Although it isn't as quick as ChatGPT or Claude, the execution speed seems quite adequate for a laptop. Colibri provides an option to monitor the technical details in the **Profiling** section, which provides metrics such as Token generation speed, total time taken per execution.  

{{<images
    imagenames="token-generation-speed.png"
    style=""
    captions="Token generation speed: On CPU only mode, the model is able to generate tokens at the rate of ~3 tokens/sec.">}}

I also attempted configuring the same model with the NVIDIA GPU. The generation rate increased to `~6 tokens/sec`, which is quite impressive in itself, considering the GPU barely has 4 GB of available VRAM left for Colibri.

## Verdict: Great for some applications, good for the others!
All in all, I find 3-6 token/sec to be quite fair on a laptop, but still a bit slow for active, long interactions. However, this is still useful in applications such as scheduled workflows, where quick execution is not crucial. Additionally, it's entirely local. So, I can get going with my financial analysis now :D

If you're reading this line, I'm assuming you are interested in installing Colibri too. Installed it? Broke something? Found a model that runs even better? Tell me - I'm on [LinkedIn](https://www.linkedin.com/in/shivamsnaik/).

## References
1. [Colibri: Hierarchical inference engine](https://justvugg.github.io/colibri/)
2. [Mixture of Experts (MoE)](https://developer.nvidia.com/blog/dense-vs-moe-models-active-parameters-throughput-and-when-to-choose-each/)