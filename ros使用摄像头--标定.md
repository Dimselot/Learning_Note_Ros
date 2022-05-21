
[摄像头使用和标定](https://www.corvin.cn/535.html#:~:text=%E2%80%9D%EF%BC%8C%20%E5%AF%B9%E4%BA%8E%22unknown,control%20%27focus_auto%27%22%E7%9A%84%E8%AD%A6%E5%91%8A%E6%98%AF%E7%94%B1%E4%BA%8E%E5%90%AF%E5%8A%A8%E7%9A%84%E6%91%84%E5%83%8F%E5%A4%B4%E4%B8%8D%E5%85%B7%E5%A4%87%E8%87%AA%E5%8A%A8%E5%AF%B9%E7%84%A6%E5%8A%9F%E8%83%BD%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%90%AF%E5%8A%A8%E7%9A%84%E6%91%84%E5%83%8F%E5%A4%B4%E5%85%B7%E5%A4%87%E8%87%AA%E5%8A%A8%E5%AF%B9%E7%84%A6%E5%8A%9F%E8%83%BD%E7%9A%84%E8%AF%9D%EF%BC%8C%E5%88%99%E6%B2%A1%E6%9C%89%E8%BF%99%E4%B8%AA%E8%AD%A6%E5%91%8A%E6%8F%90%E7%A4%BA%E3%80%82)

# 通过apt来便捷安装usb_cam
```
sudo apt-get install ros-kinetic-usb-cam

git clone https://github.com/ros-drivers/usb_cam
```



# 启动摄像头


```
roslaunch usb_cam usb_cam-test.launch
```
校准摄像头
在kinetic中安装校准软件包命令如下：
```
sudo apt install ros-kinetic-camera-calibration
```

在使用该校准软件包时需要启动uvc_camera软件包，这样我们才能使用camera-clibration软件进行校准，最终生成的校准yaml文件是可以在uvc_camera和usb_cam软件包中通用的。

安装uvc_camera的命令如下：
在kinetic中安装命令：
```
sudo apt install ros-kinetic-uvc-camera
```


# 查看默认的相机信息

首先启动uvc_camera节点，通过如下命令启动，注意需要先启动roscore节点后再启动该节点：
```
rosrun uvc_camera uvc_camera_node

```


# 准备校准用的棋盘
8x6棋盘横向有9个方块，所以有8个交叉点，而竖向有7个方块，有6个交叉点，所以它被称为8x6棋盘
使用如下命令来进行启动校准节点：

rosrun camera_calibration cameracalibrator.py --size 8x6 --square` 0.0245 `image:=/image_raw
--size是上述棋盘的宽度和高度， --square` 0.0245是棋盘的一个小方格的实际尺寸`,由于使用A4纸张打印出来是24.5毫米（我们可以使用尺子测量一下每个格子的宽度即可），所以下面命令中写了0.0245


在校准过程中需要将棋盘`对着相机朝着左/右/上/下/前/后移动`，还需要`倾斜棋盘`，这是个比较无聊的过程，我们需要拿着校准棋盘不断的移动，晃荡个大概5分钟左右就差不多了，看到基本上绿色已经填满了。校准所需的所有图像都记录下来之后，CALIBRATE按钮会被激活。点击这个按钮后会进行实际的校准计算，这需要大约3到5分钟。此时就可以将校准板放下了，耐心等待计算完成后，点击SAVE按钮保存校准信息，存储的地址显示在执行校准的终端窗口中，是存储在某一个/tmp目录（如“/tmp/calibrationdata.tar.gz”）中。

接下来将生成的校准文件移动到当前用户的home目录下：
![[附件/Pasted image 20220520235243.png]]

解压结果的压缩包
```
mv ost.txt ost.ini
```


在文件夹内
```
rosrun camera_calibration_parsers convert ost.ini ost.yaml
```


创建~/.ros/camera_info
```
mkdir ~/.ros/camera_info

mv ost.yaml ~/.ros/camera_info/

```




最后只需要打开head_camera.yaml文件修改一下就能保证该校准参数文件正常使用了：将camera_name:narrow_stereo 中的narrow_strereo 改为跟`文件名相同`的head_camera(head_camera.yaml)就可以了
![[附件/Pasted image 20220520235303.png]]



#  使用rosbag记录日志
当我们在对视频中图像信息流进行处理时经常`需要不断的来打开摄像头进行测试`，但是这样太过麻烦，而且需要一直插着摄像头，此时我们就可以将相机拍摄的视频保存下来，这样我们可以`通过不断的向话题中重发消息`就可以`模拟有相机的情况`了，而且我们还可以`将该bag包分发给别人进行测试`。

rqt_bag是一个可以将消息进行`可视化的GUI工具`，ROS日志信息中的rosbag是基于文本的，对于图像数据类的消息管理就需要用rqt_bag了，它可以进行`存储、回放和压缩话题消息`，而且所有的命令都是基于按钮制作的，所以很容易使用，就跟常用的视频编辑器一样，可以在`时间轴上拖拽查看相机图像`

首先使用roslaunch启动usb_cam节点，使用如下命令启动即可：
```
roslaunch usb_cam usb_cam-test.launch
```

使用rosbag来记录指定话题的数据，我这里保存的是压缩图像的话题，这样保存的日志文件很小，方便我们进行调试，使用如下命令开始记录话题输出：
```
rosbag record /usb_cam/image_raw/compressed
```

![[附件/Pasted image 20220520235217.png]]

从开始记录日志开始相机现在画面内容都会被录制下来，此时我们就可以在相机前进行各种图像录制，当录制完成后只需要ctrl+c来中断record即可。

接下来准备开始数据回放，首先需要将`usb_cam的launch给中断`了，不然等下的数据回放话题会跟这个原始的有冲突。要想在rqt中加载bag日志信息，只要在`中断中`输入rqt选中[插件Plugins]->[日志Logging]->[数据包bag]，然后选择坐上方的`文件夹图标（Load Bag）`加载刚才录制的bag文件即可
![[附件/Pasted image 20220520235505.png]]
加载刚才保存的bag文件后选中话题，`选中Publish`后就可以在image_view中看到刚才录制的画面了，如果没有的话可以尝试点击刷新按钮：
![[附件/Pasted image 20220520235609.png]]


