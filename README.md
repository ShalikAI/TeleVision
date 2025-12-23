<h1 align="center"><img src="img/logo.png" width="40"> Open-TeleVision: Teleoperation with

Immersive Active Visual Feedback</h1>

<p align="center">
<img src="./img/main.webp" width="80%"/>
</p>

## Introduction
This code contains implementation for teleoperation and imitation learning of Open-TeleVision.

## Clone the repo
```
git clone https://github.com/ShalikAI/TeleVision.git
cd TeleVision
```

## Installation
Activate conda environment and install packages:
```bash
source ~/miniconda3/bin/activate
conda create -n tv python=3.8
conda activate tv
conda install -c conda-forge av ffmpeg
# pip3 install -r requirements.txt
pip3 install -r requirements.txt --no-deps
cd act/detr 
pip3 install -e .
```

Install ZED sdk: https://www.stereolabs.com/developers/release/

Install ZED Python API:
```bash
cd /usr/local/zed/ 
python3 get_python_api.py
```

Install Isaac Gym for Teleoperation in Simulated Environment: 
Download the Isaac Gym Preview 4 release from the [website](https://developer.nvidia.com/isaac-gym). Extract the zip file and copy the folder `isaacgym` inside `TeleVision`. Go inside `TeleVision`:
```bash
cd ~/TeleVision/isaacgym/python/
pip3 install -e .
```

## Teleoperation Guide

### Local streaming
For **Quest** local streaming, we will use adb (Android Debug Bridge):
```
sudo apt install -y adb
adb version
sudo apt install -y android-sdk-platform-tools-common
sudo usermod -aG plugdev $USER
```
Use the adb reverse tool to set tcp port:
```
adb reverse tcp:8012 tcp:8012
```
Then, do a `adb reverse --list` to verify:
```
adb reverse --list
```
See the hand in action in vuer site (https://vuer.ai?grid=False):

<div align="center">
    <a href="https://www.youtube.com/shorts/SqhlbrGsM8M">
        <img src="img/vuer_vr.gif" width="700">
    </a>  
</div>

To test the application locally, we need to create a self-signed certificate and install it on the client. You need a ubuntu machine and a router. Connect the Quest3 and the ubuntu machine to the same router through WiFi. 
1. install [mkcert](https://github.com/FiloSottile/mkcert):
```
wget https://github.com/FiloSottile/mkcert/releases/download/v1.4.4/mkcert-v1.4.4-linux-amd64
chmod +x mkcert-v1.4.4-linux-amd64
sudo mv mkcert-v1.4.4-linux-amd64 /usr/local/bin/mkcert
mkcert -version
```
2. Check local ip address: 

```
ifconfig | grep inet
```
Suppose the local ip address of the ubuntu machine is `192.168.8.102`.

3. create certificate: 

```
cd TeleVision/teleop
mkcert -install && mkcert -cert-file cert.pem -key-file key.pem 192.168.8.102 localhost 127.0.0.1
```
ps. `cert.pem` and `key.pem` files should be generated/moved in `Television/teleop`.

4. open firewall on server
```
sudo iptables -A INPUT -p tcp --dport 8012 -j ACCEPT
sudo iptables-save
sudo iptables -L
```
or can be done with `ufw`:
```
sudo ufw allow 8012
```
5.
```
tv = OpenTeleVision(self.resolution_cropped, shm.name, image_queue, toggle_streaming, ngrok=False)
```

Settings > General > About > Certificate Trust Settings. Under "Enable full trust for root certificates", turn on trust for the certificate.

7. Open the browser on Quest 3 and go to `https://192.168.8.102:8012?ws=wss://192.168.8.102:8012`

8. Click `Enter VR` and ``Allow`` to start the VR session.


### Network Streaming
For Meta Quest3, installation of the certificate is not trivial. We need to use a network streaming solution. We use `ngrok` to create a secure tunnel to the server.

1. Install [ngrok](https://ngrok.com/download):
```
curl -sSL https://ngrok-agent.s3.amazonaws.com/ngrok.asc \
  | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null \
  && echo "deb https://ngrok-agent.s3.amazonaws.com bookworm main" \
  | sudo tee /etc/apt/sources.list.d/ngrok.list \
  && sudo apt update \
  && sudo apt install ngrok
```
2. Add the authentication token:
```
ngrok config add-authtoken <authentication_token>
```
For first time, you have to generate the authentication token by opening an account in ngrok and signing up for 2FA. From next time, you will find the authentication token inside `/home/arghya/.config/ngrok/ngrok.yml`. 
3. Run ngrok:
```
ngrok http 8012
```
4. Copy the https address (https://unenthralling-nonunited-thomas.ngrok-free.dev) and open the browser on Meta Quest3 and go to the address. You can also open `http://127.0.0.1:4040` or `http://localhost:8012` to see the same thing in your ubuntu pc. 

ps. When using ngrok for network streaming, remember to call `OpenTeleVision` with:
```
self.tv = OpenTeleVision(self.resolution_cropped, self.shm.name, image_queue, toggle_streaming, ngrok=True)
```

### Copy Files 
You can record the VR Streaming and copy files from Occulus Quest. List the adb devices:
```
adb devices
```
List the files inside the SD card of Occulus Quest 3:
```
adb shell ls /sdcard/Oculus/Screenshots
adb shell ls /sdcard/Oculus/VideoShots
```
Copy the necessary files:
```
adb pull /sdcard/Oculus/VideoShots/<file_name>.mp4 .
```

### Simulation Teleoperation Example
1. After setup up streaming with either local or network streaming following the above instructions, you can try teleoperating two robot hands in Issac Gym:
```
cd teleop && python teleop_hand.py
```
2. Confirm your Vuer server is actually on 8012:
```
sudo lsof -iTCP:8012 -sTCP:LISTEN -n -P
```
You should see which process is listening on 8012.
3. Go to the local site (https://localhost:8012/) and see the hands in action:

<div align="center">
    <a href="https://www.youtube.com/shorts/ZQiJDCjlNwc">
        <img src="img/localhost_vr.gif" width="700">
    </a>  
</div>

4. Go to vuer site (https://localhost:8012/?ws=wss://localhost:8012) on Quest 3 for local streaming or ngrok site (https://unenthralling-nonunited-thomas.ngrok-free.dev) for network streaming, click `Enter VR` and ``Allow`` to enter immersive environment.

5. See hands in 3D inside isaac gym:

<div align="center">
    <a href="https://www.youtube.com/watch?v=Dj490QL5inE">
        <img src="img/vr.gif" width="700">
    </a>  
</div>

## Training Guide
1. Download dataset from https://drive.google.com/drive/folders/11WO96mUMjmxRo9Hpvm4ADz7THuuGNEMY?usp=sharing.

2. Place the downloaded dataset in ``data/recordings/``.

3. Process the specified dataset for training using ``scripts/post_process.py``.

4. You can verify the image and action sequences of a specific episode in the dataset using ``scripts/replay_demo.py``.

5. To train ACT, run:
```
python imitate_episodes.py --policy_class ACT --kl_weight 10 --chunk_size 60 --hidden_dim 512 --batch_size 45 --dim_feedforward 3200 --num_epochs 50000 --lr 5e-5 --seed 0 --taskid 00 --exptid 01-sample-expt
```

6. After training, save jit for the desired checkpoint:
```
python imitate_episodes.py --policy_class ACT --kl_weight 10 --chunk_size 60 --hidden_dim 512 --batch_size 45 --dim_feedforward 3200 --num_epochs 50000 --lr 5e-5 --seed 0 --taskid 00 --exptid 01-sample-expt\
                               --save_jit --resume_ckpt 25000
```

7. You can visualize the trained policy with inputs from dataset using ``scripts/deploy_sim.py``, example usage:
```
python deploy_sim.py --taskid 00 --exptid 01 --resume_ckpt 25000
```
