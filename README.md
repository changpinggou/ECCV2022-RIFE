# RIFE Video Frame Interpolation v2.1
## [arXiv](https://arxiv.org/abs/2011.06294) | [Project Page](https://rife-vfi.github.io) | [YouTube](https://www.youtube.com/watch?v=60DX2T3zyVo&feature=youtu.be) | [Bilibili](https://www.bilibili.com/video/BV1K541157te?from=search&seid=5131698847373645765) | [Colab](https://colab.research.google.com/github/hzwer/arXiv2020-RIFE/blob/main/Colab_demo.ipynb)

Some apps has integrated RIFE. You can refer to [Waifu2x-Extension-GUI](https://github.com/AaronFeng753/Waifu2x-Extension-GUI), [Flowframes](https://nmkd.itch.io/flowframes) and [RIFE-ncnn-vulkan](https://github.com/nihui/rife-ncnn-vulkan). 中文补帧软件也已经发布，免费下载 [SVFI](https://github.com/YiWeiHuang-stack/Squirrel-Video-Frame-Interpolation)。

2021.2.9 News: We have updated the v2.0 model, faster and much better!

2021.1.4 News: We have updated the v1.8 model optimized for 2D animation. Please check our [update log](https://github.com/hzwer/arXiv2020-RIFE/issues/41#issuecomment-737651979).

There is [a tutorial of RIFE](https://www.youtube.com/watch?v=gf_on-dbwyU&feature=emb_title) on Youtube.

Our model can run 30+FPS for 2X 720p interpolation on a 2080Ti GPU. Currently, our method supports 2X,4X,8X... interpolation, and multi-frame interpolation between a pair of images. Everyone is welcome to use our alpha version and make suggestions!

16X interpolation results from two input images: 

![Demo](./demo/I0_slomo_clipped.gif)
![Demo](./demo/I2_slomo_clipped.gif)

## Usage

### Installation

```
git clone git@github.com:hzwer/arXiv2020-RIFE.git
cd arXiv2020-RIFE
pip3 install -r requirements.txt
```

* Download the pretrained **HDv2** models from [here](https://drive.google.com/file/d/1wsQIhHZ3Eg4_AfCXItFKqqyDMB4NS0Yd/view?usp=sharing). (百度网盘链接:https://pan.baidu.com/s/1qGsnXE7-Q4UPYUNFevz60w 密码:cn1f，把压缩包解开后放在 train_log/\*.pkl)

* Unzip and move the pretrained parameters to train_log/\*.pkl

**This model is designed to provide better visual effects for users and should not be used for paper benchmark.**

### Run

**Video Frame Interpolation**

You can use our [demo video](https://drive.google.com/file/d/1i3xlKb7ax7Y70khcTcuePi6E7crO_dFc/view?usp=sharing) or your own video. 
```
python3 inference_video.py --exp=1 --video=video.mp4 
```
(generate video_2X_xxfps.mp4)
```
python3 inference_video.py --exp=2 --video=video.mp4
```
(for 4X interpolation)
```
python3 inference_video.py --exp=1 --video=video.mp4 --UHD
```
(If your video has very high resolution such as 2K and 4K, we recommend to use UHD mode.)
```
python3 inference_video.py --exp=2 --img=input/
```
(to read video from pngs, like input/0.png ... input/612.png, ensure that the png names are numbers)
```
python3 inference_video.py --exp=2 --video=video.mp4 --fps=60
```
(add slomo effect, the audio will be removed)
```
python3 inference_video.py --video=video.mp4 --montage --png
```
(if you want to montage the origin video, skip static frames and save the png format output)

The warning info, 'Warning: Your video has *** static frames, it may change the duration of the generated video.' means that your video has changed the frame rate by adding static frames, it is common if you have processed 25FPS video to 30FPS.

**Image Interpolation**

```
python3 inference_img.py --img img0.png img1.png --exp=4
```
(2^4=16X interpolation results)
After that, you can use pngs to generate mp4:
```
ffmpeg -r 10 -f image2 -i output/img%d.png -s 448x256 -c:v libx264 -pix_fmt yuv420p output/slomo.mp4 -q:v 0 -q:a 0
```
You can also use pngs to generate gif:
```
ffmpeg -r 10 -f image2 -i output/img%d.png -s 448x256 -vf "split[s0][s1];[s0]palettegen=stats_mode=single[p];[s1][p]paletteuse=new=1" output/slomo.gif
```

### Run in docker
Place the pre-trained models in `train_log/\*.pkl` (as above)

Building the container:
```
docker build -t rife -f docker/Dockerfile .
```

Running the container:
```
docker run --rm -it -v $PWD:/host rife:latest inference_video --exp=1 --video=untitled.mp4 --output=untitled_rife.mp4
```
```
docker run --rm -it -v $PWD:/host rife:latest inference_img --img img0.png img1.png --exp=4
```

Using gpu acceleration (requires proper gpu drivers for docker):
```
docker run --rm -it --gpus all -v /dev/dri:/dev/dri -v $PWD:/host rife:latest inference_video --exp=1 --video=untitled.mp4 --output=untitled_rife.mp4
```

## Evaluation
**Our paper has not been officially published yet, and our method and experimental results are under improvement. Due to the incorrect data reference, the latency measurement of Sepconv and TOFlow in our arxiv paper needs to be modified.**

Download [RIFE model](https://drive.google.com/file/d/1c1R7iF-ypN6USo-D2YH_ORtaH3tukSlo/view?usp=sharing) or [RIFE2F1.5C model](https://drive.google.com/file/d/1ve9w-cRWotdvvbU1KcgtsSm12l-JUkeT/view?usp=sharing) reported by our paper.

**Vimeo90K**: Download [Vimeo90K dataset](http://toflow.csail.mit.edu/) at ./vimeo_interp_test

**MiddleBury**: Download [MiddleBury OTHER dataset](https://vision.middlebury.edu/flow/data/) at ./other-data and ./other-gt-interp
```
python3 benchmark/Vimeo90K.py
# (Final result: "Avg PSNR: 35.695 SSIM: 0.9788")
python3 benchmark/MiddelBury_Other.py
# (Final result: "2.058")
```

## Training and Reproduction
Because Vimeo90K dataset and the corresponding optical flow labels are too large, we cannot provide a complete dataset download link. We provide you with [a subset containing 100 samples](https://drive.google.com/file/d/1_MQmFWqaptBuEbsV2tmbqFsxmxMIqYDU/view?usp=sharing) for testing the pipeline. Please unzip it at ./dataset

Each sample includes images (I0 I1 Imid : 9 x 256 x 448), and optical flow (flow_t0, flow_t1: 4, 256, 448). 

For origin images, you can download them from [Vimeo90K dataset](http://toflow.csail.mit.edu/).

For generating optical flow labels, our paper use [pytorch-liteflownet](https://github.com/sniklaus/pytorch-liteflownet). We also recommend [RAFT](https://github.com/princeton-vl/RAFT) because it's easier to configure. We recommend generating optical flow labels on 2X size images for better labels. You can also generate labels during training, or finetune the optical flow network on the training set. The final impact of the above operations on Vimeo90K PSNR is expected to be within 0.3.

We use 16 CPUs, 4 GPUs and 20G memory for training: 
```
python3 -m torch.distributed.launch --nproc_per_node=4 train.py --world_size=4
```

## Artists
[Navetek](https://space.bilibili.com/477230155?spm_id_from=333.788.b_765f7570696e666f.1) | [御坂大哥想让我表白](https://space.bilibili.com/94367406?spm_id_from=333.788.b_765f7570696e666f.2) | [没有鼠鼠的雏子Official](https://space.bilibili.com/11541212?spm_id_from=333.788.b_765f7570696e666f.2) | [赫萝与罗伦斯的旅途](https://space.bilibili.com/14181638?from=search&seid=5375297779239252137) | [异度界限](https://space.bilibili.com/352761108?from=search&seid=5375297779239252137) | [今天我练出腹肌了吗](https://space.bilibili.com/402021?spm_id_from=333.788.b_765f7570696e666f.2)

## Collection
[【𝟒𝐊 𝟏𝟐𝟎𝐅𝐏𝐒】TV动画《魔女之旅》NCOP リテラチュア
](https://www.bilibili.com/video/BV1sr4y1P7Wg?from=search&seid=17356442119935352422)

[「120FPS」EVA × KATE × 米山舞「绫波丽，首次口红，在那之后」](https://www.bilibili.com/video/BV1yz4y1m7iF?from=search&seid=1496025207731732060)

[【4K・60fps】- 邓丽君，你有男朋友没有？- 我的愿望是要有四个（不是）](https://www.bilibili.com/video/BV1ZK411u7CM?from=search&seid=17760804974094984298)

[「𝟒𝑲 𝟏𝟐𝟎𝑭𝑷𝑺」最终幻想14 全版本CG合集（1.0-6.0全收录）](https://www.bilibili.com/video/BV1R541177qr?from=search&seid=3113679500491477684)
## Citation

```
@article{huang2020rife,
  title={RIFE: Real-Time Intermediate Flow Estimation for Video Frame Interpolation},
  author={Huang, Zhewei and Zhang, Tianyuan and Heng, Wen and Shi, Boxin and Zhou, Shuchang},
  journal={arXiv preprint arXiv:2011.06294},
  year={2020}
}
```

## Reference

Optical Flow:
[ARFlow](https://github.com/lliuz/ARFlow)  [pytorch-liteflownet](https://github.com/sniklaus/pytorch-liteflownet)  [RAFT](https://github.com/princeton-vl/RAFT)  [pytorch-PWCNet](https://github.com/sniklaus/pytorch-pwc)

Video Interpolation: 
[DAIN](https://github.com/baowenbo/DAIN)  [CAIN](https://github.com/myungsub/CAIN)  [TOflow](https://github.com/HyeongminLEE/AdaCoF-pytorch)  [MEMC-Net](https://github.com/baowenbo/MEMC-Net)   [SoftSplat](https://github.com/sniklaus/softmax-splatting)   [SepConv](https://github.com/sniklaus/sepconv-slomo)   [BMBC](https://github.com/JunHeum/BMBC)
