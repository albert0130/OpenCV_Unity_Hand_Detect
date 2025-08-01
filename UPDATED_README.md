
# OpenCV + Unity: Real-Time 3D Hand Gesture Tracking

## Overview

This project demonstrates how to use **Python**, **OpenCV**, and **Mediapipe** to perform **real-time hand gesture recognition**, and stream the results to a **Unity 3D environment** via **UDP sockets**. The Unity side visualizes hand gestures using a skeletal model composed of spheres and lines.

## Demo

🔗 [Live Demo](https://hackathon2022.juejin.cn/#/works/detail?unique=WJoYomLPg0JOYs8GazDVrw)  
![Demo Preview](https://img-blog.csdnimg.cn/6d4646f75ffc43039bc83401150ab20c.png)

More details and technical articles available on [CSDN](https://blog.csdn.net/weixin_50679163?type=edu)

---

## Getting Started

### Python Dependencies

Install via pip:

```bash
pip install mediapipe==0.8.9.1
```

Or clone and install from source:  
🔗 [Mediapipe GitHub](https://github.com/google/mediapipe)

### Environment

- Python 3.7  
- Mediapipe 0.8.9.1  
- OpenCV-Python 4.5.5.64  
- OpenCV-contrib-Python 4.5.5.64  
- NumPy 1.21.6  

![Env Setup](https://img-blog.csdnimg.cn/e49c1c0fa21c4df08ee58e57ce57f8b4.png)

---

## Gesture Recognition & Capture

Mediapipe provides 21 hand landmarks, which we capture using a webcam, process with OpenCV, and transmit via UDP to Unity.

![Landmark Reference](https://img-blog.csdnimg.cn/5c947c080dbd49339d64b3c0e988ea41.png)

### Camera Setup

```python
import cv2

cap = cv2.VideoCapture(0)
while True:
    success, img = cap.read()
    imgRGB = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    cv2.imshow("Camera", img)
    cv2.waitKey(1)
```

### FPS Calculation

```python
import time

pTime = 0
while True:
    cTime = time.time()
    fps = 1 / (cTime - pTime)
    pTime = cTime
    cv2.putText(img, str(int(fps)), (10, 70), cv2.FONT_HERSHEY_PLAIN, 3, (255, 0, 255), 3)
```

### UDP Socket Communication

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
serverAddressPort = ("127.0.0.1", 5052)
sock.sendto(str.encode(str(data)), serverAddressPort)
```

---

## Full Python Script

### `Hands.py`

```python
from cvzone.HandTrackingModule import HandDetector
import cv2
import socket

cap = cv2.VideoCapture(0)
cap.set(3, 1280)
cap.set(4, 720)
_, img = cap.read()
h, w, _ = img.shape
detector = HandDetector(detectionCon=0.8, maxHands=2)

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
serverAddressPort = ("127.0.0.1", 5052)

while True:
    success, img = cap.read()
    hands, img = detector.findHands(img)
    data = []
    if hands:
        lmList = hands[0]["lmList"]
        for lm in lmList:
            data.extend([lm[0], h - lm[1], lm[2]])
        sock.sendto(str.encode(str(data)), serverAddressPort)
    cv2.imshow("Image", img)
    cv2.waitKey(1)
```

---

## Unity Setup

### Hand Model

In Unity, create 21 **Sphere GameObjects** for the finger tips and joints, and 21 **Cube/LineRenderers** to connect them.

### UDP Receiver (C#)

```csharp
using UnityEngine;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading;

public class UDPReceive : MonoBehaviour
{
    Thread receiveThread;
    UdpClient client;
    public int port = 5052;
    public string data;

    void Start()
    {
        receiveThread = new Thread(ReceiveData);
        receiveThread.IsBackground = true;
        receiveThread.Start();
    }

    void ReceiveData()
    {
        client = new UdpClient(port);
        while (true)
        {
            IPEndPoint anyIP = new IPEndPoint(IPAddress.Any, 0);
            byte[] dataByte = client.Receive(ref anyIP);
            data = Encoding.UTF8.GetString(dataByte);
        }
    }
}
```

### Hand Rendering in Unity

```csharp
public class HandTracking : MonoBehaviour
{
    public UDPReceive udpReceive;
    public GameObject[] handPoints;

    void Update()
    {
        string data = udpReceive.data.Trim('[', ']');
        string[] points = data.Split(',');

        for (int i = 0; i < 21; i++)
        {
            float x = 7 - float.Parse(points[i * 3]) / 100;
            float y = float.Parse(points[i * 3 + 1]) / 100;
            float z = float.Parse(points[i * 3 + 2]) / 100;
            handPoints[i].transform.localPosition = new Vector3(x, y, z);
        }
    }
}
```

### Line Connection (`LineRenderer`)

```csharp
public class LineCode : MonoBehaviour
{
    public Transform origin;
    public Transform destination;
    LineRenderer lineRenderer;

    void Start()
    {
        lineRenderer = GetComponent<LineRenderer>();
        lineRenderer.startWidth = 0.1f;
        lineRenderer.endWidth = 0.1f;
    }

    void Update()
    {
        lineRenderer.SetPosition(0, origin.position);
        lineRenderer.SetPosition(1, destination.position);
    }
}
```

---

## Final Result

![Result Preview](https://img-blog.csdnimg.cn/f26208cf46094cbdb3f914602b75daad.png)

---

## Credits

- [Mediapipe](https://github.com/google/mediapipe) by Google
- Unity Engine
- OpenCV and Python Community

---

## License

This project is for educational and research purposes. Feel free to modify and expand on it for your own work.

**Happy Coding! 🎮🖐️**
