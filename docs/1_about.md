---
layout: page
title: About
permalink: /about/
mermaid: true
---

# What is the Ground Magnetometer?
The Ground Magnetometer is a low-cost, science grade magnetometer equipped with a sensor able to measure magnetic fields with a resolution of ~3 nT at 1 Hz. This resolution enables the magnetometer to detect both space-borne and ground geomagnetic activity, ranging from solar flares to geomagnetic storms. The instrument consists of a magnetic sensor (a RM3100, manufactured by PNI Corporation), which communicates with a Linux-based computer system; the system is operational once the necessary software is installed.

The Ground Magnetometer is comprised of two software components: a data collection software and a web-based dashboard.

# Installing the mag-usb Software

Mag-usb is the software responsible for reading data from the ground magnetometer. It must be installed on a computer running Linux or macOS. Windows is not supported.

Depending on your operating system, the steps for installing and running mag-usb differ significantly. Linux is considered the traditional OS for running mag-usb, while macOS is an alternative choice. Both installation methods are included here for completeness.

## Installing and Running on Linux

1) Ensure the following packages are installed on your computer.
   ```bash
   sudo apt-get install -y build-essential cmake git
   ```
2) Run `hostname -I` to find your computer's IP address. Write it down as it will be needed for later.
   * If you have admin access to your network's router, consider fixing your device's IP address to the network so it never changes.
2) Clone mag-usb to your computer.
   ```bash
   git clone https://github.com/wittend/mag-usb
   ```
3) Navigate to the project's root directory and use cmake to build the project.
   ```bash
   cd mag-usb
   cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DENABLE_WEBSOCKET=ON
   cmake --build build --target mag-usb
   ```
4) Copy the config.toml file in the repository to /etc/.
   ```bash
   sudo mkdir /etc/mag-usb
   sudo cp src/config.toml /etc/mag-usb/config.toml
   ```
5) Connect your MIB along with your magnetometer to your computer. The green and blue LEDs should each flash once before holding steady. If only the green LED flashes and it is blinking, try unplugging and plugging in the MIB again. If issues persist, use a different cable.
6) Run `ls -l /dev/ttyACM*` to see a list of connected devices. Your device should appear as "/dev/ttyACM0".
7) (Optional) If you have multiple USB devices connected to your computer (such as an RX-888), consider installing the provided udev rule in the repository to grant a stable device name for the MIB.
   ```bash
   sudo cp install/99-PololuI2C.rules /etc/udev/rules.d
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   ```
   Unplug and replug the adapter and run the ls in step 6 again to verify the changes. The device will now be listed as "/dev/ttyMAG0".
6) Sudo edit the config.toml using your preferred text editor. Nano is used in this example.
   ```bash
   sudo nano /etc/mag-usb/config.toml
   ```
   Under the [websocket] section, change enable from "false" to "true".
   ```toml
   [websocket]
   # Enable WebSocket output server.
   enable = true
   # Bind address and port for WebSocket clients.
   bind_address = "0.0.0.0"
   port = 8765
   ```
   If you installed the udev rules as in the previous step, under [i2c], change the value for portpath to "/dev/ttyMAG0".

   Press ctrl+x to save and exit.
8) Use sudo to grant your user dialout permissions to access connected devices.
   ```bash
   sudo usermod -aG dialout <USER>
   ```
9) Run mag-usb with `./build/mag-usb &`

## Installing and Running on macOS

Running mag-usb on macOS utilizes a Docker container along with hardware passthrough to circumvent the software being otherwise incompatible.

1) [Docker Desktop](https://www.docker.com/products/docker-desktop/) is required to run mag-usb on macOS. Follow the installation wizard to install Docker.
2) If Homebrew is not installed on your Mac, install it with this command.
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
3) Use Homebrew to install socat, which is necessary for hardware passthrough.
   ```bash
   brew install socat
   ```
4) Use [git](https://git-scm.com/) to clone the mag-usb repository. Install git if it's not already installed.
   ```bash
   git clone https://github.com/wittend/mag-usb
   ```
5) Plug your MIB with magnetometer into your computer. Both the green and blue LEDs on the MIB should flash once before holding steady. If only the green LED is lit and blinking, try unplugging and plugging in the MIB again or using a different cable.
6) Run `ls` to determine the name of your device. The MIB should be listed as `/dev/cu.usbmodem14#01`.
   ```bash
   ls -l /dev/cu.*
   ```
7) Use socat to establish a port listener to the connected device as found in the previous step.
   ```bash
   socat -d -d TCP-LISTEN:1234,reuseaddr,fork FILE:<DEVICE>,raw,echo=0
   ```
8) Open Docker Desktop to ensure the Docker engine is running.
9) In another Terminal, navigate to mag-usb's root directory and start a Docker container. Mag-usb will be pre-built inside the container.
   ```bash
   docker compose up
   ```

   This should give the following output:
   ```
   (1) Starting virtual TTY bridge...
   (2) Waiting for virtual device...
   READY!
      To edit config: nano /etc/mag-usb/config.toml
      To run mag-usb: ./mag-usb
   ```

   Press d inside the Terminal to detach the container.
10) Use docker compose to enter the container.
    ```bash
    docker compose exec host bash
    ```
11) Use nano inside the container to edit the config.toml file
    ```bash
    nano /etc/mag-usb/config.toml
    ```
    Under the [websocket] section, change enable from "false" to "true". Do not change anything else.
    ```toml
    [websocket]
    # Enable WebSocket output server.
    enable = true
    # Bind address and port for WebSocket clients.
    bind_address = "0.0.0.0"
    port = 8765
    ```
    Press ctrl+x to save and exit.
12) Run `./mag-usb &`. You may `exit` the container once mag-usb is running.

# Installing the gmag_webui Software

The gmag_webui software consists of a real-time dashboard that connects to mag-usb. The dashboard can be installed on the same computer as mag-usb, or it can be installed on a separate computer.

## Installing the Runtime Environment

The dashboard is served using the Deno runtime environment. Refer to [Deno's website](https://docs.deno.com/runtime/#install-deno) for how to install Deno for your specific OS.

[Docker Desktop](https://www.docker.com/products/docker-desktop/) can be used in lieu of installing Deno. However, it is more restricting in its use compared to installing Deno directly.

## Installing the Project

1) Clone the gmag_webui repository to your computer with git.
   ```bash
   git clone https://github.com/HamSCI/gmag_webui
   ```
2) Unzip the archive containing Font Awesome. On Windows or macOS, this can be done in a File Explorer or Finder window.

   On Linux, install the unzip command with your package manager.
   ```bash
   cd gmag_webui/vendor
   sudo apt-get install -y unzip
   unzip fontawesome-free-7.1.0-web.zip
   ```

## Running the Dashboard

In a Terminal or Command Prompt, navigate to the project's root directory.
* If Deno is installed, run `deno task start`.
* If Docker Desktop is installed, run `docker compose up -d frontend` after starting the Docker engine.

The dashboard is accessible on localhost:8000. If for any reason you need to change the port, a .env.example file is included that can be duplicated into a .env file.

### Installing Deploy Assets

On Ubuntu and Debian distributions of Linux, two services can be installed that start the dashboard on boot and automatically update it.

1) Navigate to gmag_webui's root directory on your computer.
2) Run `sudo ./deploy/install.sh` to install the services.
3) Run `sudo reboot` to reboot your computer.
4) Upon logging back in, verify that both services are running with `systemctl`.
   ```bash
   systemctl status gmag-webui.service
   systemctl status gmag-webui-update.service
   ```
