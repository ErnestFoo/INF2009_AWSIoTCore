**AWS IoT: Real-Time Device Data to AWS IoT Dashboard**

**Objective:** By the end of this session, participants will understand 
1. Introduction to AWS IoT Core 
2. How to create a IoT thing/device in AWS (aka RaspberryPi in our case)
2. How to send real-time device data to AWS IoT Core and 
3. How to ingest device data in AWS Dynamo DB via IoT rule.

---

**Prerequisites:**
1. Raspberry Pi with Raspbian OS installed.
2. MicroSD card (16GB or more recommended).
4. Internet connectivity (Wi-Fi or Ethernet).
5. Basic knowledge of Python and Linux commands.
6. AWS Account setup with free credits.

---

**1. Introduction**

 AWS IoT core is a cloud managed service, which lets you connect to edge devices and makes it possible to develop applications that can act on the data generated by edge devices. It support a large volumne of messages and can route those messages to AWS services or other devices reliably and securely. One of the main aspects of AWS IoT code is the device communication and supports three main protocol: MQTT, HTTP, and websocket. In this lab, we will learn how to connect a thing aka device in IoT core, establish secure connection from raspberryPi over MQTT protocol and then ingest real-time data into a no-sql dynamodB database.

**2.Setup IoT Thing aka Device in IoT Core**

- **Part 1.** Let us first create thing in AWS IoT core. Thing in IoT core means any edge device and in our case it would be a raspberryPi device. Head over to your AWS console and go to IoT Core. Please ensure you have configured your aws region to Singapore i.e., _ap-southeast-1_ (You can switch region from your console top right selecton). Now, we will be creating a IoT thing of type raspberryPi as shown in below videos. Please follow and setup IoT thing as shown.
 
https://github.com/user-attachments/assets/7e295f39-d728-4b97-9358-fa0d2df21b5e

https://github.com/user-attachments/assets/b78d9212-9613-4a52-bad0-6f70459cf56e

https://github.com/user-attachments/assets/ca08f3cd-ebf8-49f6-993c-31fcbd878573





-  Set up and activate a virtual environment named "dlonedge" for this experiment (to avoid conflicts in libraries) as below:
  ```bash
  sudo apt install python3-venv
  python3 -m venv dlonedge
  source dlonedge/bin/activate
  ```

- Installing PyTorch and OpenCV:
  ```bash
  pip install paho-mqtt
  pip install --upgrade psutil
  ```

- Same as last lab, for video capture we’re going to be using OpenCV to stream the video frames. The model we are going to use in this lab is MobileNetV2, which takes in image sizes of 224x224. We are targeting 30fps for the model but we will request a slightly higher framerate of 36 fps than that so there is always enough frames and bandwidth of image pre-processing and model prediction.

- **Part 1.** [sample code](Codes/mobile_net.py) is used to directly load pre-trained MobileNetV2 model, doing model inference and finally, Observe the fps as shown in screenshot below when run on RaspberryPi 4B. As shown, with no optimization of model, we could only achieve of 5-6 fps much below our desired target.

  ![image1](https://github.com/user-attachments/assets/8e3cf302-45f3-41c9-85a5-a1bd118d30c4)

- **Part 2.** Edit line number 11 as shown below to enable quantization in [sample code](Codes/mobile_net.py) to use quantized version of MobileNetV2 model.

  ```message = json.dumps({"time": int(time.time()),"quality": "GOOD","hostname": "rpiedge","value": psutil.cpu_percent()},indent=2)
  ```

    Finally, observe the fps as shown in screenshot below after using quantized model of MobileNetV2. We can now achieve close to 30 fps as required because of smaller footprint of quantized model.

    ![image2](https://github.com/user-attachments/assets/7086f300-4edf-4c41-a799-c496001ee1d1)

    [Quantization](https://pytorch.org/docs/stable/quantization.html) techniques enable computations and tensor storage at reduced bitwidths compared to floating-point precision. In a quantized model, some or all operations use this lower precision, resulting in a smaller model size and the ability to leverage hardware-accelerated vector operations.

- **Part 3.** Uncomment lines 57-61 in [sample code](Codes/mobile_net.py) to print the top 10 predictions in real-time as shown in below video.

https://github.com/user-attachments/assets/5ee2a4c8-1988-4021-b194-aa0786a1ebfc


**3. Quantization using Pytorch**
- Neural networks typically use 32-bit floating point precision for activations, weights, and computations. Quantization reduces this precision to smaller data types (like 8-bit integers), decreasing memory and speeding up computation. This compression is not lossless, as lower precision sacrifices dynamic range and resolution. Thus, a balance must be struck between model accuracy and the efficiency gains from quantization.

- In this section, we would learn how to use Pytorch to perform different quantization methods on a neural network architecture. There are two ways in general to quantize a deep learning model:

    1. Post Training Quantization: After we have a trained model, we can convert the model to a quantized model by converting 32 bit floating point weights and activations to 8 bit integer, but we may see some accuracy loss for some types of models.
    2. Quantization Aware Training: During training, we insert fake quantization operators into the model to simulate the quantization behavior and convert the model to a quantized model after training based on the model with fake quantize operators. This is harder to apply than post-training quantization since it requires retraining the model, but typically gives better accuracy.

- Please refer to [sample jupyter notebook file](Codes/PyTorch_Quantisation.ipynb), which demonstrates how to quantized a pre-trained model using post-training quantization approach as well as quantization aware training. Please run the sample code preferably in google colab if you do not have computer with good hardware specs.


**4. Homework and Optional Exercise**
- Try running quantized version of some large language models (like llama, mixtral etc.) on Raspberry Pi. This [link](https://www.dfrobot.com/blog-13498.html) demonstrates some of the LLMs on Raspberry Pi 4 and 5.
- Take any complex Deep Learning Model like resnet, mobileNet OR your own architecure and try different quantization methods as explained in section 3. Once deployed on RaspberryPi, Observe the size, performance and speed of the quantized model.
