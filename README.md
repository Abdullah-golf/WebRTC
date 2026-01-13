# 🚀 安装与运行步骤 (Installation & Usage)
# go2rtc
## 步骤 1: 下载 go2rtc
根据你的系统架构下载对应的二进制文件。
### 对于 x86_64 (大多数 PC/工控机):
```Bash
wget https://github.com/AlexxIT/go2rtc/releases/latest/download/go2rtc_linux_amd64
chmod +x go2rtc
```
### 对于 ARM64 (树莓派/Jetson/RDK):
```Bash
wget https://github.com/AlexxIT/go2rtc/releases/latest/download/go2rtc_linux_arm64
chmod +x go2rtc
```
## 步骤 2: 创建配置文件
在 go2rtc 同级目录下创建 go2rtc.yaml 文件。

```Bash
nano go2rtc.yaml
```

```YAML
streams:
  #USB 摄像头推流配置 (根据实际情况调整分辨率和码率)
  camera_usb: exec:ffmpeg -hide_banner -f v4l2 -input_format mjpeg -video_size 640x480 -i /dev/video0 -c:v libx264 -g 30 -preset ultrafast -tune zerolatency -b:v 2000k -f rtsp {output}
```

## 步骤 3: 启动服务
直接运行二进制文件即可启动服务。

```Bash
./go2rtc_linux_amd64 -c go2rtc.yaml
```

## 步骤 4: 访问视频流
打开浏览器（推荐 Chrome 或 Safari），访问管理面板：
局域网访问: http://localhost:1984
远程访问: http://你的穿透域名:端口

# SAKURA frp
## 第一步：注册并创建隧道（网页端操作）
你先不需要在机器人上操作，先去 Sakura Frp 官网配置“开路”。
   - 注册账号： 去 Sakura Frp 官网 注册一个账号
   - 创建隧道： 进入管理面板，点击左侧 “穿透” -> “隧道列表” -> “创建隧道”
   - 选择节点： 这是关键！选一个离你物理距离近的国内节点
   - 隧道类型： 选择 TCP
   - 本地地址： 127.0.0.1
   - 本地端口： 1984 (这是 Go2RTC 的网页和 MSE 端口)
   - 远程端口： 留空（随机分配）或者填一个它允许范围内的端口
   - 点击 “完成创建”
   - 远程管理： 双击节点启动

## 第二步：在机器人上安装客户端（Ubuntu 端操作）
具体可以参考：https://doc.natfrp.com/launcher/usage.html
   - 下载客户端： Sakura Frp 官方提供了一键安装脚本
```Bash
# 进入下载目录
cd ~/Desktop

# 下载 (这是 Sakura Frp 的官方客户端，叫 nyale 或 frpc)
sudo bash -c ". <(wget -O- https://doc.natfrp.com/launcher.sh)"
```
   - 要求输入访问密钥
     <img width="359" height="314" alt="image" src="https://github.com/user-attachments/assets/5de132c5-0205-413e-95b3-f73deb611c0f" />
   - 要求设定远程管理密码
   - 有关命令
```Bash
systemctl enable --now natfrp.service
systemctl status natfrp.service

# 查看日志，确认看到 "远程管理连接成功" 的输出
journalctl -u natfrp.service -f
```

## 第三步：调整 Go2RTC 配置（关键！）
因为你的带宽变大了，而且我们现在主要走 TCP，你需要修改 go2rtc.yaml 来“解放”画质。
```YAML
streams:
  # Sakura Frp 带宽很大，我们可以把码率从 800k 提升到 6000k甚至更高
  # 分辨率也可以尝试改回 2K
  camera_usb: exec:ffmpeg -hide_banner -f v4l2 -input_format mjpeg -video_size 2560x1440 -i /dev/video0 -c:v libx264 -g 30 -preset superfast -tune zerolatency -b:v 6000k -f rtsp {output}
```

## 第四步：公网访问
拿起你的手机或另一台电脑。
打开浏览器，输入 Sakura Frp 告诉你的那个地址： http://cn-js-1.natfrp.cloud:34567 (注意改成自己的地址)。
会要求访问密码，在隧道刘表可见。
<img width="1121" height="234" alt="image" src="https://github.com/user-attachments/assets/b2021fdf-ef6f-4c66-9a47-7a27ab6ff21b" />

