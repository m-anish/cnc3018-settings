# Webcam RTSP Streamer and MediaMTX Setup

This guide walks you through installing and configuring `mediamtx` (MediaMTX) for WebRTC streaming, configuring `ffmpeg` to stream a webcam feed via RTSP, and setting up systemd unit files to automatically start the services at boot.

### Prerequisites

- Raspberry Pi 3B+ (or other compatible Raspberry Pi model)
- A webcam connected via USB
- Access to a terminal with `sudo` privileges
- An active internet connection for installing packages

---

### 1. Install `mediamtx`

We'll download the latest precompiled ARM64v8 tarball for the Raspberry Pi 3B and configure `mediamtx` for use.

1. **Download and Install MediaMTX:**

   Go to the [latest release page of MediaMTX](https://github.com/bluenviron/mediamtx/releases/latest), find the ARM64v8 tarball (suitable for the Raspberry Pi 3B), and download it. You can do this from your Raspberry Pi directly or from another device and then transfer the file.

   Use `wget` to download the tarball:

   ```bash
   wget https://github.com/bluenviron/mediamtx/releases/download/v0.16.0/mediamtx-linux-arm64v8.tar.gz
   ```

2. **Extract the tarball and install the executable:**

   After downloading, extract the contents:

   ```bash
   tar -xvzf mediamtx-linux-arm64v8.tar.gz
   ```

   Move the `mediamtx` binary to `/usr/local/bin`:

   ```bash
   sudo mv mediamtx /usr/local/bin/
   ```

3. **Configure MediaMTX:**

   The configuration file for `mediamtx` is located at `/etc/mediamtx/mediamtx.yml`. You may need to create the directory and configuration file manually:

   ```bash
   sudo mkdir -p /etc/mediamtx
   sudo nano /etc/mediamtx/mediamtx.yml
   ```

   You can modify this file based on your requirements (e.g., WebRTC configuration).

---

### 2. Install `ffmpeg`

We'll use `ffmpeg` to stream video from your webcam to an RTSP server.

1. **Install `ffmpeg` and dependencies:**

   ```bash
   sudo apt update
   sudo apt install -y ffmpeg
   ```

   Ensure `ffmpeg` is installed correctly by running:

   ```bash
   ffmpeg -version
   ```

---

### 3. Create Systemd Unit Files

We will create two systemd unit files:
- **`mediamtx.service`** to start MediaMTX on boot.
- **`rtsp-webcam-streamer.service`** to stream the webcam using `ffmpeg` to RTSP and start after `mediamtx`.

#### **3.1. Create `mediamtx.service`**

1. **Create the unit file:**

   ```bash
   sudo nano /etc/systemd/system/mediamtx.service
   ```

2. **Add the following content:**

   ```ini
   [Unit]
   Description=MediaMTX Real-time media server and media proxy
   After=network.target

   [Service]
   ExecStart=/usr/local/bin/mediamtx /etc/mediamtx/mediamtx.yml
   Restart=on-failure
   RestartSec=10s
   StartLimitBurst=10
   User=root
   Group=root
   Environment=LD_LIBRARY_PATH=/usr/local/lib  # Adjust if needed

   [Install]
   WantedBy=multi-user.target
   ```

3. **Reload systemd and enable the service:**

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable mediamtx.service
   sudo systemctl start mediamtx.service
   ```

   You can check the service status with:

   ```bash
   sudo systemctl status mediamtx.service
   ```

#### **3.2. Create `rtsp-webcam-streamer.service`**

1. **Create the unit file:**

   ```bash
   sudo nano /etc/systemd/system/rtsp-webcam-streamer.service
   ```

2. **Add the following content:**

   ```ini
   [Unit]
   Description=Webcam RTSP Streamer (FFmpeg)
   After=network.target
   After=mediamtx.service
   Requires=mediamtx.service

   [Service]
   ExecStartPre=/bin/sleep 5  # Wait for 5 seconds before starting ffmpeg
   ExecStart=/usr/bin/ffmpeg -f v4l2 -framerate 5 -s 800x600 -i /dev/v4l/by-id/usb-046d_081b_44519B90-video-index0 \
   -f alsa -channels 1 -sample_rate 16000 -i hw:2,0 \
   -vcodec h264_v4l2m2m -b:v 400k -maxrate 400k -bufsize 400k \
   -acodec opus -b:a 8k \
   -strict experimental \
   -f rtsp -rtsp_transport tcp rtsp://localhost:8554/stream
   Restart=on-failure
   RestartSec=10s
   StartLimitBurst=10
   User=root
   Group=root

   [Install]
   WantedBy=multi-user.target
   ```

3. **Reload systemd and enable the service:**

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable rtsp-webcam-streamer.service
   sudo systemctl start rtsp-webcam-streamer.service
   ```

   Check the service status with:

   ```bash
   sudo systemctl status rtsp-webcam-streamer.service
   ```

---

### 4. Test the Stream

Once both services are running, you can test the webcam stream in a browser using `mediamtx`.

1. Open a browser and visit `http://<your-ip>:8080/stream`.

   Replace `<your-ip>` with the IP address of your Raspberry Pi.

2. You should be able to view the webcam stream served via WebRTC from the `mediamtx` server.

---

### 5. Troubleshooting

- Ensure that the camera is recognized by running:

  ```bash
  ls /dev/v4l/by-id/
  ```

- Check logs for each service to troubleshoot issues:

  ```bash
  sudo journalctl -u mediamtx.service
  sudo journalctl -u rtsp-webcam-streamer.service
  ```

---

### Conclusion

You've now set up `mediamtx` to stream webcam data as WebRTC and used `ffmpeg` to stream the webcam feed over RTSP. Both services are configured to automatically start at boot with systemd, and you can access the webcam feed via a browser using WebRTC.
