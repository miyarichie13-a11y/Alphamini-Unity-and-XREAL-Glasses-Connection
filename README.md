
# AlphaMini Guided Breathing with Unity and XREAL Glasses

This document outlines the complete setup and workflow for connecting an AlphaMini robot to a Unity application, creating an immersive, synchronized breathing experience on XREAL glasses.

The core of this project is a local network connection, where a Python script acts as a server to control the Unity app running on your phone.

***

## Prerequisites & Setup

### 1. Network Configuration

This is the most important step. Ensure your computer, the AlphaMini robot, and your Android phone are all connected to the **exact same Wi-Fi network**.

### 2. PC Setup

* **Python:** Your Python environment should have the **ubt-mini-sdk** and **pyserial** libraries installed.
* **Unity:** You should have Unity Hub and the correct Unity Editor version **(2022.3.28f1)** with Android Build Support installed.

---

## Unity Project Configuration

This phase configures your Unity project to work with the XREAL SDK and your local network.

### 1. Open and Switch Platform

* Open your Unity project.
* Go to **File > Build Settings...**, select **Android** from the list, and click **"Switch Platform"**.

### 2. Download & Import XREAL SDK

* Go to the official XREAL Developer website to download the latest **NRSDK for Unity**.
* In Unity, import the downloaded **.unitypackage** file by navigating to **Assets > Import Package > Custom Package...**.

### 3. Configure Player Settings

* Go to **Edit > Project Settings > Player**.
* Under the **Android** tab, ensure the following are set correctly:
    * **Package Name:** This must be a unique name (e.g., com.yourcompany.alphaminixr).
    * **Minimum API Level and Target API Level:** Set these according to the XREAL SDK's recommendations.

### 4. Implement the Android Networking Fix

This is a crucial step to allow your app to communicate with the Python script on your local network.

* **Create the Required Folder Structure:** Go to Assets/Plugins/Android/. Create a folder named **networkconfig.androidlib**.
* Inside that **.androidlib** folder, you will need to create three files we discussed previously: network_security_config.xml, AndroidManifest.xml, and build.gradle.

#### a. network_security_config.xml

This file explicitly tells Android to allow unencrypted (cleartext) traffic, which is necessary for your local TCP connection.

* Inside your **networkconfig.androidlib** folder, create the path res/xml/.
* Create a file named network_security_config.xml inside the xml folder and paste this code:

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <network-security-config>
        <base-config cleartextTrafficPermitted="true">
            <trust-anchors>
                <certificates src="system" />
            </trust-anchors>
        </base-config>
    </network-security-config>
    ```

#### b. AndroidManifest.xml (For the Library)

This is a minimal manifest for the library itself.

* Inside the root of the **networkconfig.androidlib** folder, create a file named AndroidManifest.xml and paste this code:

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="[http://schemas.android.com/apk/res/android](http://schemas.android.com/apk/res/android)">
        <uses-permission android:name="android.permission.INTERNET" />
        <application android:networkSecurityConfig="@xml/network_security_config" />
    </manifest>
    ```

#### c. build.gradle (For the Library)

This file tells Unity how to correctly package your library.

* Inside the root of the networkconfig.androidlib folder, create a file named build.gradle and paste this code:

    ```groovy
    plugins {
        id 'com.android.library'
    }
    android {
        namespace 'com.yourname.alphaminiproject.networkconfig' // Use your Package Name
        compileSdkVersion 33
    }
    ```

---

## Unity Scene Setup

Now, prepare your main scene to run the scripts.

### 1. Add Your Scripts to the Scene

* Create an empty GameObject and name it **NetworkManager**. Attach your **TCPClient.cs** and **MainThreadDispatcher.cs** scripts to this object.
* Create another GameObject to handle your visuals (e.g., **VisualController**) and attach your visual logic script (SphereController.cs) to it.

### 2. Link the Scripts in the Inspector

* Select the **NetworkManager** GameObject in your Hierarchy.
* In the Inspector, find the **TCP Client** script component.
* Find your computer's IP address (by typing ipconfig in the Command Prompt). Set the **Host** field to this IP address.
* Drag the **VisualController** GameObject from your Hierarchy into the public slot on the **TCP Client** component (e.g., the **Sphere Controller** field).

---

## Building the Unity App (.apk)

* Go to **File > Build Settings...**.
* Click the **"Add Open Scenes"** button to add your configured scene to the build list.
* Connect your Android phone to your computer with a USB cable.
* Click the **"Build and Run"** button. Unity will compile your project, create an .apk file, and automatically install it on your phone.

---

## The Runtime Workflow

This is the final sequence to run the full experience.

1.  **Start the Python Server:** On your computer, run your Python script. Wait for it to connect to the robot and display the **"Server listening... Waiting for Unity client..."** message.
2.  **Connect the Glasses:** Plug your XREAL glasses into your phone.
3.  **Launch the Unity App:** From your phone's app drawer, find and launch the application you just built. It will appear inside the glasses.
4.  **Connect & Play:** The Unity app will automatically connect to the running Python script, and the experience will begin.
5.  **Shutdown:** When you're finished, close the Unity app, then stop the Python script in the command prompt by pressing **Ctrl+C**.
