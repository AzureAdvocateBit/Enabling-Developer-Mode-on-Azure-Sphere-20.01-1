# Enabling Developer Mode on Azure Sphere 20.01

[Azure Sphere Command Line Utility](https://docs.microsoft.com/azure-sphere/reference/overview?WT.mc_id=iot-0000-dglover#overview-of-azsphere?WT.mc_id=github-blog-dglover)

Quick write up if you get stuck enabling developer mode on your Azure Sphere after updating to 20.01 Retail Eval feed. 

After updating my Azure Sphere device to 20.01 to the *Retail Eval* feed it all worked fine for High Level application development but when I tried to enable Real Time debugging hit the issue where I could no long enable debugging on the device for High Level or Real Time apps.

```bash
azsphere dev edv
```

```text
azsphere device enable-development
Getting device group 'Development' for product 'MyProduct'.
Device ID: '3EA5EF0407D0539999999999999999999999999999999999999999647A96B8183CE07C1817D02CDD95B0D8B8E5428D1A201A8C34137935'
Downloading device capability configuration.
Setting device group to 'Development' with ID '9f9999999-061d-4a76-8982-999999978c8'.
Successfully disabled application updates.
Enabling application development capability on attached device.
Applying device capability configuration to device.
error: The device did not accept the device capability configuration. Please check the Azure Sphere OS on your device is up to date using 'azsphere device show-deployment-status'.
```

The error message:

```text
error: The device did not accept the device capability configuration. Please check the Azure Sphere OS on your device is up to date using 'azsphere device show-deployment-status'.
```

A collogue figured out a fix and I'm writing up for others.

---

## Overview

Essentially, create a new Device Group, upload a app image package to the group, then move the device to the group, ensure the device wifi connected, then reset the device.  A few moments later the device started, the image was deployed, and the device is now enabled for local deployment and debugging.

**Note**, I had an existing Product Group aptly named *MyProduct*.

Follow this guide.

---

## Step 1: Clear any previously selected device capability configuration

[azsphere device commands](https://docs.microsoft.com/azure-sphere/reference/azsphere-device?WT.mc_id=iot-0000-dglover)

```bash
azsphere dev cap select -n
```

---

## Step 2: Create a Device Group with OSFeedType RetailEval and an App update

[azsphere device group commands](https://docs.microsoft.com/azure-sphere/reference/azsphere-device-group?WT.mc_id=iot-0000-dglover)

```bash
azsphere dg create -n Test1 -a On -o RetailEval -pn MyProduct
```

---

## Step 3: Upload a User App to the Device Group

### Create an app package

From the **Azure Sphere Command Line**, change to your application **out** folder.

Run the following command, be sure to change the *--input* to match your approot directory name, set the *--output* file to match your needs.

[azsphere image-package commands](https://docs.microsoft.com/azure-sphere/reference/azsphere-image-package?WT.mc_id=iot-0000-dglover)

```bash
azsphere image-package pack-application --input approotAzureSphereIoTCentral --output glovebox.imagepackage
```

### Upload the application package

[azsphere image commands](https://docs.microsoft.com/azure-sphere/reference/azsphere-image?WT.mc_id=iot-0000-dglover)

```bash
azsphere image add --filepath glovebox.imagepackage --temporary --force
```

Example output from the image add command.

**Make a note of the Image ID.**

```text
warn: 'glovebox.imagepackage' targets Beta APIs that may change or be removed in future.
Uploading image from file 'glovebox.imagepackage':
--> Image ID:       99999999-84f1-9999-aa71-b70c999999
--> Component ID:   9999999-66da-9999-bae1-ac2999999627
--> Component name: 'AzureSphereIoTCentral'
Retaining temporary state for uploaded image.
Successfully uploaded image with ID 'f999999-84f1-9999-aa71-b70cf159999' and name 'AzureSphereIoTCentral' to component with ID '2599999-9999-9999-bae1-99999cdd3627'.
```

### Add the App Package to the Device Group you created

[azsphere device-group commands](https://docs.microsoft.com/azure-sphere/reference/azsphere-device-group?WT.mc_id=iot-0000-dglover)

```bash
azsphere device-group deployment create -ii  99999999-84f1-9999-aa71-b70c999999 -dgn Test1
```

---

## Step 4: Move the the Attached Device to the newly create Device Group

[azsphere device commands](https://docs.microsoft.com/azure-sphere/reference/azsphere-device?WT.mc_id=iot-0000-dglover)

```bash
azsphere dev update -pn MyProduct -dgn "Test1"
```

---

## Step 5: Ensure your Device is Wifi Connected

[azsphere device commands](https://docs.microsoft.com/azure-sphere/reference/azsphere-device?WT.mc_id=iot-0000-dglover)

```bash
azsphere dev wifi show-status
```

## Step 6: Reset your Azure Sphere

Press the reset button on your Azure Sphere, wait a few moments, the device will restart, the app package will download and start, and you device will be enabled for debugging again!
