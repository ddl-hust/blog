---
ctitle: ESP8266 开关
data: 2019-09-23
---

在办公室希望连接到家里的电脑，内网穿透方案选的是 frp，但家里的电脑在走的时候会忘记打开，亦或者死机蓝屏的时候出现故障无法手动重启。正好手头里有两台路由器，一台 `R6300 V2` 刷了梅林固件，一台 `WNDR 3700 V4` 刷了 `OpenWRT`  ，另外还有一块 `ESP8266 NodeMcu Lua WIFI V3 ESP-12N` 的板子。另外还需要一个 3.3 v/5v 的继电器，某宝上 3.9 包邮。

方案图



### 路由器控制 USB 电源开关

其实我第一个想到的方案就是直接通过路由器的 USB 接口控制继电器来控制 PC 主板开关，网上也有文章说通过 echo 命令控制 USB 电源。

[Turning USB power on and off](https://openwrt.org/docs/guide-user/hardware/usb.overview) 这个是通过 GPIO 引脚驱动来控制的

```bash
# 打开电源
echo 1 > /sys/class/gpio/gpioN/value
# 关闭电源
echo 0 > /sys/class/gpio/gpioN/value
```

但我的 R6300V2 上没有这些 GPIO 的引脚设备文件

```bash
lede@R6300V2-5501:/tmp/home/root# tree /sys/class/gpio/
/sys/class/gpio/
└── gpio
    ├── dev
    ├── subsystem -> ../../gpio
    └── uevent
lede@R6300V2-5501:/sys/class/gpio/gpio# cat dev
254:0
lede@R6300V2-5501:/sys/class/gpio/gpio# cat uevent
MAJOR=254
MINOR=0
DEVNAME=gpio
```

无功而返，最后在又找到了另外一种通过命令行来控制 USB 电源开关的办法 [Controlling a USB power supply (on/off) with linux](https://stackoverflow.com/questions/4702216/controlling-a-usb-power-supply-on-off-with-linux) 

```bash
# disable external wake-up; do this only once
echo disabled > /sys/bus/usb/devices/usb1/power/wakeup 

echo on > /sys/bus/usb/devices/usb1/power/level       # turn on
echo suspend > /sys/bus/usb/devices/usb1/power/level  # turn off
```

但我的R6300V2路由器里也没有这个文件，无功而返。但在这个 USB 设备文件里却发现了有意思的事情

```bash
lede@R6300V2-5501:/tmp/home/root# ls -al /sys/bus/usb/devices/ | awk '{print $9,$10,$11}'
./
../
1-0:1.0 -> ../../../devices/pci0000:00/0000:00:0b.1/usb1/1-0:1.0/
1-1 -> ../../../devices/pci0000:00/0000:00:0b.1/usb1/1-1/
1-1:1.0 -> ../../../devices/pci0000:00/0000:00:0b.1/usb1/1-1/1-1:1.0/
2-0:1.0 -> ../../../devices/pci0000:00/0000:00:0b.0/usb2/2-0:1.0/
usb1 -> ../../../devices/pci0000:00/0000:00:0b.1/usb1/
usb2 -> ../../../devices/pci0000:00/0000:00:0b.0/usb2/
lede@R6300V2-5501:/tmp/home/root#
```

R6300V2 支持的 USB 设备类型驱动有以下几种

```bash
lede@R6300V2-5501:/sys/bus/usb/drivers# ls
asix/        cdc_mbim/    cdc_wdm/     qmi_wwan/    usb/         usbfs/
cdc_ether/   cdc_ncm/     hub/         rndis_host/  usb-storage/ usblp/
```

虽然 R6300V2 号称有一个 USB3.0 和一个 USB2.0 ，但设备速度还是 480Mbps ，USB2.0 的速度，不明白这是为什么？

```bash
lede@R6300V2-5501:/sys/devices/pci0000:00/0000:00:0b.1/usb1# cat speed
480
```

最终通过路由器控制 USB 电源开关的方法无功而返，只能使用 ESP8266 结合继电器来实现了。

```
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ESP8266WebServer.h>
//指定开关控制的阵脚
#define LED 2
//配置静态 IP
IPAddress staticIP(192, 168, 0, 230);
IPAddress gateway(192, 168, 0, 1);
IPAddress subnet(255, 255, 255, 0);

//SSID and Password of your WiFi router
const char* ssid = " ";
const char* password = " ";
const char* user = "esp8266";
const char* pass = "esp8266";
const char* realm = "Custom Auth Realm";
String authFailResponse = "Authentication Failed";
const char MAIN_page[] PROGMEM = R"=====(
<!DOCTYPE html><html><body><a href="ledOn">ON</a><br><a href="ledOff">OFF</a></body></html>
)=====";

//Declare a global object variable from the ESP8266WebServer class.
ESP8266WebServer server(80); //Server on port 80

//===============================================================
// This routine is executed when you open its IP in browser
//===============================================================
void handleRoot() {
 Serial.println("You called root page");
 String s = MAIN_page; //Read HTML contents
 server.send(200, "text/html", s); //Send web page
}

void handleLEDon() {
 Serial.println("LED on page");
 digitalWrite(LED,LOW); //LED is connected in reverse
 server.send(200, "text/html", "LED is ON"); //Send ADC value only to client ajax request
}

void handleLEDoff() {
 Serial.println("LED off page");
 digitalWrite(LED,HIGH); //LED off
 server.send(200, "text/html", "LED is OFF"); //Send ADC value only to client ajax request
}
//==============================================================
//                  SETUP
//==============================================================
void setup(void){
  Serial.begin(115200);

  WiFi.begin(ssid, password);     //Connect to your WiFi router
  Serial.println("");

  //Onboard LED port Direction output
  pinMode(LED,OUTPUT);
  //Power on LED state off
  digitalWrite(LED,HIGH);

  WiFi.disconnect();  //Prevent connecting to wifi based on previous configuration
  WiFi.begin(ssid, password);
  WiFi.config(staticIP, subnet, gateway);
  WiFi.mode(WIFI_STA); //WiFi mode station (connect to wifi router only

  // Wait for connection
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  //If connection successful show IP address in serial monitor
  Serial.println("");
  Serial.print("Connected to ");
  Serial.println(ssid);
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());  //IP address assigned to your ESP

  server.on("/", []() {
    if (!server.authenticate(user,pass))
    {
      return server.requestAuthentication(DIGEST_AUTH, realm, authFailResponse);
    }
    Serial.println("You called root page");
        String s = MAIN_page;
        server.send(200, "text/html", s);
  });
  server.on("/ledOn", handleLEDon); 
  server.on("/ledOff", handleLEDoff);
  server.begin();
  Serial.println("HTTP server started");
}

void loop(void){
  server.handleClient();
}
```

