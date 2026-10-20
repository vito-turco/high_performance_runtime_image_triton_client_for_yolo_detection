# Runtime container for high performance C++ triton client for yolo detection, with NVIDIA NVDEC, NVIDIA CVCUDA and NVIDIA Triton
#### Real time object detection from RTSP streaming, NVIDIA GPU needed
# Description
This is a runtime container to perform real time detection from multiple rtsp streams using several NVIDIA technologies, such as 
* NVIDIA Video Codec SDK for gpu-accelerated stream decoding
* NVIDIA CV-CUDA for gpu-accelerated frame preprocessing 
* NVIDIA Triton as inference server for all the streams.

Since it is a runtime container, meant to be a demonstrator, there is no source code in the docker image.  

A separated thread inside the application consumes a queue of frames, saving them in /outputs folder, together with the bounding boxes of the detected objects. **This is the only output of the application**.

It is compatible with any yolo model with output shape [-1,Max_Detections,(x1,y1,x2,y2,conf,cls)], NMS-free, since no postprocessing for detections is implemented.
**All yolo26 variants are ok**. So it is any model supporting end-to-end inference, whose outputs are shaped as described above.

# Performance
I could test it with my 3 commercial ip cameras, over my local Wi-Fi network, getting real time inference on all of them, at 2K resolution concurrent streams.

The amount of resources required are approximately 1 GB VRAM (244 MB for triton, 230MB per client), 3 GB of host RAM (2 of which claimed by Triton) and an almost negligible use of CPU. The percentage of usage of my GPU was at max around 25%, the usage of the decoding unit inside it was around 6-7%.

The tests were performed on my personal computer with 
* AMD Ryzen™ 9 7940HX with Radeon™ Graphics × 32
* NVIDIA GeForce RTX 4060 8GB
* Ubuntu 26.04.1 LTS

GPU compute capability greater than 6 is required.


# Getting started
Since it is a demonstrator of no explicit practical use for other developers, I decided to host the tar of the docker image on my personal cloud, instead of pushing it on some public registry, squandering their storage.
The tar is available at [onedrive](https://1drv.ms/u/c/472a7e06093eda41/IQCE_qZGDxndTaVZNCtDfpwaASqMHymQ25PV7gnIDGLEPLk?e=AWDge7) and the image can be loaded into your local registry with
> docker load -i rtsp-cuda-triton-client-app-v0.1.tar

Once the client image is loaded, clone this repository and replicate the client service in docker-compose.yml for each stream you want to use (i.e. a client per camera). \
Each client entrypoint requires 3 arguments: rtsp connection string (e.g. rtsp://user:passwd@address:port/stream_endpoint), camera name (e.g. frontdoor), detection model name (e.g. yolo26n_onnx).

Finally, run
> docker compose up

### Note: 
If you want to use your usb webcam for stream use /dev/video0 instead of an rtsp connection string. Before doing it, assure your webcam supports 

H.264 (often listed as H264 or AVC1) — Most recommended for compatibility.

HEVC (often listed as HEVC or H265)

VP8 / VP9

If /dev/video0 only lists uncompressed raw formats (such as YUYV, MJPG, or BGR3), NVDEC hardware decoding cannot be used directly on those raw pixel frames because NVDEC only accepts compressed video bitstreams.

To check the supported webcam streams use
> v4l2-ctl --list-formats-ext -d /dev/video0

# License
Please refer to the license files from NVIDIA Video Codec SDK, NVIDIA CV-CUDA,  NVIDIA TRITON and ultralytics.

# DISCLAIMER OF WARRANTY AND LIMITATION OF LIABILITY
1. "AS IS" AND "AS AVAILABLE" BASIS

THE SOFTWARE IS PROVIDED "AS IS" AND "AS AVAILABLE," WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, OR NON-INFRINGEMENT. THE ENTIRE RISK AS TO THE QUALITY, PERFORMANCE, ACCURACY, AND RELIABILITY OF THE SOFTWARE REMAINS SOLELY WITH YOU.

2. NO GUARANTEE OF PERFORMANCE

WE MAKE NO WARRANTY OR REPRESENTATION THAT THE SOFTWARE WILL MEET YOUR REQUIREMENTS, OPERATE WITHOUT INTERRUPTION, BE TIMELY, SECURE, OR ERROR-FREE, OR THAT ANY DEFECTS OR ERRORS IN THE SOFTWARE WILL OR CAN BE CORRECTED.

3. LIMITATION OF DAMAGES

TO THE MAXIMUM EXTENT PERMITTED BY APPLICABLE LAW, IN NO EVENT SHALL THE AUTHORS, COPYRIGHT HOLDERS, OR DISTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, PUNITIVE, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; BUSINESS INTERRUPTION, OR SYSTEM FAILURE) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF OR INABILITY TO USE THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.