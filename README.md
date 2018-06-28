---
services: iot-edge, custom-vision-service, iot-hub
platforms: python
author: ebertrams
---

# Custom Vision + Azure IoT Edge on a Raspberry Pi 3

> [!NOTE]
> This sample still uses the IoT Edge Preview bits. To use it with the new IoT Edge GA bits, you will need to update the **Camera capture** and **SenseHat display** modules manually to use the latest IoT SDK versions. The **Custom vision** should remain the same. 

This is a sample showing how to deploy a Custom Vision model to a Raspberry Pi 3 device running Azure IoT Edge. This solution is made of 3 modules:

- **Camera capture** - this module captures the video stream from a USB camera, sends the frames for analysis to the custom vision module and shares the output of this analysis to the edgeHub. This module is written in python and uses [OpenCV](https://opencv.org/) to read the video feed.
- **Custom vision** - it is a web service over HTTP running locally that takes in images and classifies them based on a custom model built via the [Custom Vision website](https://azure.microsoft.com/en-us/services/cognitive-services/custom-vision-service/). This module has been exported from the Custom Vision website and slightly modified to run on a ARM architecture. You can modify it by updating the model.pb and label.txt files to update the model.
- **SenseHat display** - this module gets messages from the edgeHub and blinks the raspberry Pi's senseHat according to the tags specified in the inputs messages. This module is written in python and requires a [SenseHat](https://www.raspberrypi.org/products/sense-hat/) to work.

## Get started
1- Update the module.json files of the 3 modules above to point to your own Azure Container Registry

2- Build the full solution by running the `Build IoT Edge Solution` command from the [Azure IoT Edge extension in VS Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).

## Prerequisites

You can run this solution on either of the following hardware:

- **Raspberry Pi 3**: Set up Azure IoT Edge on a Raspberry Pi 3 ([instructions](https://blog.jongallant.com/2017/11/azure-iot-edge-raspberrypi/)) with a [SenseHat](https://www.raspberrypi.org/products/sense-hat/) and use the arm32v7 module tags.

- **Simulated Azure IoT Edge device** (such as a PC): Set up Azure IoT Edge ([instructions on Windows](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-simulate-device-windows), [instructions on Linux](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-simulate-device-linux)) and use the amd64 module tags. A test x64 deployment manifest is already available. To use it, rename the 'deployment.template.test-amd-64' to 'deployment.template.json', then build the IoT Edge solution from this manifest and deploy it to an x64 device.

## Deploy custom vision to an x64 device running Ubuntu

- On the Azure portal, create an [IoT Hub and an IoT Edge device](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-simulate-device-linux) registered to the hub. Create an Azure Container Registry.
- Install Ubuntu on an x64 device.
    - Install python-pip, [Docker](https://docs.docker.com/install/linux/docker-ce/ubuntu) and [IoT edge runtime](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-simulate-device-linux#install-and-start-the-iot-edge-runtime). Start the runtime. Add your [registry key to runtime](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-csharp-module#add-registry-credentials-to-edge-runtime) by "```sudo iotedgectl login --address <container registry address> --username <username> --password <password>```" 
- On a development system, install and start [Docker](https://docs.docker.com/docker-for-windows/install/#install-docker-for-windows-desktop-app).
    - Switch to Linux containers by right clicking on the Docker icon in the task bar and selecting that option.
    - Install [Visual Studio Code](https://code.visualstudio.com) and [Azure IoT Edge extension](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge). Clone the custom vision repository and open the custom-vision-service-iot-edge-raspberry-pi folder in Visual Studio Code.
    - Modify the repository value in each module's module.json to point to your Azure Container Registry.
    - Rename deployment.template.json to something else, e.g. deployment.template.json.arm32v7. Rename deployment.template.test-amd64.json to deployment.template.json.
    - Select View > Command Pallete > Run Azure IoT Edge: Build IoT Edge Solution.
    - During the build, if you see the message "unauthorized: authentication required", type in the command terminal "```docker login <container registry address>```". Enter the login and password from Azure portal > Azure Container Registry > Access keys.
    - If the build succeeds, your Azure Containter Registry will list the modules in its repository.
    - Mouse over Azure IoT Hub Devices in Visual Studio Code and click on ... Click on [Set IoT Hub Connection String](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-csharp-module#view-generated-data). Get your connection string by selecting your IoT hub in Azure portal and select Shared access policies > iothubowner.
    - Deploy the modules to the edge device by right clicking on deployment.json in Visual Studio Code > Create Deployment for IoT Edge Device.
    - View data from the modules by right clicking the device in Visual Studio Code > Start Monitoring D2C Message.

