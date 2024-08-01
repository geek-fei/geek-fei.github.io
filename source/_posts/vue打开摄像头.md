---
title: vue打开摄像头
date: 2024-08-01 10:42:31
tags:
---

## 方式一：打开摄像头和关闭摄像头

在 Vue 中打开摄像头，通常需要结合前端的 JavaScript 技术以及相关的浏览器 API 来实现。以下是一个基本的示例步骤：

1. 获取用户权限
   可以使用 navigator.mediaDevices.getUserMedia() 方法来请求权限。
2. 处理获取到的媒体流
   一旦用户授予权限，getUserMedia() 方法会返回一个 MediaStream 对象，您可以将其与 <video> 元素关联，以显示摄像头的实时画面。



以下是一个简单的示例代码：

```html
<template>
  <div>
    <video ref="video" autoplay></video>
    <button @click="startCamera">打开摄像头</button>
    <button @click="handleClose">关闭摄像头</button>
  </div>
</template>
 
<script setup>
import {ref} from 'vue';
import { ElMessage } from 'element-plus';
  
const videoRef = ref();
// 打开摄像头starem
const videoStream = ref(null);

// 判断浏览器是否支持获取摄像头
const isCameraSupported = () => {
  if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
    try {
      navigator.mediaDevices.getUserMedia({ video: true });
      return true;
    } catch (err) {
      return false;
    }
  } else {
    return false;
  }
}

// 打开摄像头
const startCamera = () => {
  // 判断浏览器是否支持获取摄像头
  if(isCameraSupported()){
    // 打开摄像头
    navigator.mediaDevices.getUserMedia({ video: true })
      .then(stream => {
        // 临时存储打开的摄像头starem
        videoStream.value = stream;
        // 将摄像头starem赋值给<video>进行渲染摄像头画面
        videoRef.value.srcObject = stream;
      })
      .catch(error => {
        console.error('摄像头打开错误：', error);
      });
  } else {
    console.error('浏览器不支持获取摄像头');
  }
  
}

// 检查摄像头是否正常(判断摄像头画面是否为黑屏)
const hasCameraFeed = (videoElement) => {
  // 检查视频元素是否正在加载
  if (videoElement.readyState >= 3) {
    // 获取视频的当前帧数据
    const canvas = document.createElement('canvas');
    canvas.width = videoElement.videoWidth;
    canvas.height = videoElement.videoHeight;
    const ctx = canvas.getContext('2d');
    ctx.drawImage(videoElement, 0, 0, canvas.width, canvas.height);

    // 将图像数据转换为数组
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;

    // 检查图像数据是否全为黑色（表示没有画面）
    let allBlack = true;
    for (let i = 0; i < data.length; i += 4) {
      if (data[i]!== 0 || data[i + 1]!== 0 || data[i + 2]!== 0) {
        allBlack = false;
        break;
      }
    }

    return !allBlack;
  } else {
    return false;
  }
}
  
// 关闭调试摄像头
const handleClose = () => {
  // 检查摄像头临时存储的stream和画面是否为黑屏
  if (videoStream.value && hasCameraFeed()) {
    // 关闭摄像头
    videoStream.value.getTracks().forEach((track) => {
      track.stop();
    });
    // 降临时存储的摄像头stream赋值为null
    videoStream.value = null;
  } else {
    console.log('未打开摄像头');
    // TODO 检查摄像头是否正常
    ElMessage({
      message: '确保摄像头画面清晰，且没有遮挡',
      type: 'warning',
    })
    return;
  }
}
</script>
```

## 方式二：打开摄像头列表，选择要使用的摄像头

每台电脑回对应有不同的摄像头设备,获取连接多个摄像头设备，此时就得获取到所有摄像头列表，进行选择使用哪一个摄像头。

```html
<template>
  <div>
    <video ref="video" autoplay></video>
    <!-- 摄像头列表: 选择摄像头,根据摄像头deviceId打开对应设备的摄像头 -->
    <el-select v-model='currentVideoDeviceId' @change='startCamera'>
      <el-option
        v-for="item in videoList"
        :key="item.deviceId"
        :label="item.label"
        :value="item.deviceId"
      />
    </el-select>
    <button @click="getCameraList">打开摄像头</button>
    <button @click="handleClose">关闭摄像头</button>
  </div>
</template>
 
<script setup>
import {ref} from 'vue';
import { ElMessage } from 'element-plus';
  
const videoRef = ref();
// 打开摄像头starem
const videoStream = ref(null);
// 所有摄像头数据
const videoList = ref([]);
// 当前摄像头设备Id
const currentVideoDeviceId = ref('');

// 判断浏览器是否支持获取摄像头
const isCameraSupported = () => {
  if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
    try {
      navigator.mediaDevices.getUserMedia({ video: true });
      return true;
    } catch (err) {
      return false;
    }
  } else {
    return false;
  }
}
  
// 获取摄像头列表
const getCameraList = () => {
  // 判断浏览器是否支持获取摄像头
  if(isCameraSupported()){
    navigator.mediaDevices.enumerateDevices()
      .then(devices => {
        const cameras = devices.filter(device => device.kind === 'videoinput');
        console.log('所有摄像头',cameras);
        /**
          所有摄像头列表数据:
          [{
            deviceId: "03ce2165757a21b4c7e31c243b0e74b2ab42cd408a542900709155a7dfa936fa",
            groupId: "2431f3cf786e40c3534393d63a20d35a48e38c5056a8c360bc808249ce594ed0",
            kind: "videoinput",
            label: "BisonCam,NB Pro (5986:9102)"
          },...]

          label字段为摄像头设备的名称, deviceId为摄像头设备对应的唯一值
        */
        
        // TODO 将所有摄像头赋值给下拉选择 cameras
        videoList.value = cameras;

        // 默认打开第一个摄像头
        startCamera(cameras[0].deviceId);
      })
      .catch(error => console.error('获取摄像头列表失败:', error));
  } else {
    console.error('浏览器不支持获取摄像头');
  }
  
}

/**
  根据摄像头的 deviceId 打开对应的摄像头
*/
const startCamera = (deviceId) => {
  // 获取摄像头deviceId
  const constraints = {
    video: { deviceId: { exact: deviceId } }
  };

  // 根据摄像头deviceId打开摄像头
  navigator.mediaDevices.getUserMedia(constraints)
      .then(stream => {
        // 临时存储摄像有的startem
        videoStream.value = stream;
        // 赋值给<video>渲染摄像头的画面
        videoRef.value.srcObject = stream;
      })
      .catch(error => {
        console.error('摄像头打开错误：', error);
      });
}

// 检查摄像头是否正常(判断摄像头画面是否为黑屏)
const hasCameraFeed = (videoElement) => {
  // 检查视频元素是否正在加载
  if (videoElement.readyState >= 3) {
    // 获取视频的当前帧数据
    const canvas = document.createElement('canvas');
    canvas.width = videoElement.videoWidth;
    canvas.height = videoElement.videoHeight;
    const ctx = canvas.getContext('2d');
    ctx.drawImage(videoElement, 0, 0, canvas.width, canvas.height);

    // 将图像数据转换为数组
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;

    // 检查图像数据是否全为黑色（表示没有画面）
    let allBlack = true;
    for (let i = 0; i < data.length; i += 4) {
      if (data[i]!== 0 || data[i + 1]!== 0 || data[i + 2]!== 0) {
        allBlack = false;
        break;
      }
    }

    return !allBlack;
  } else {
    return false;
  }
}
  
// 关闭调试摄像头
const handleClose = () => {
  // 检查摄像头临时存储的stream和画面是否为黑屏
  if (videoStream.value && hasCameraFeed()) {
    // 关闭摄像头
    videoStream.value.getTracks().forEach((track) => {
      track.stop();
    });
    // 降临时存储的摄像头stream赋值为null
    videoStream.value = null;
  } else {
    // TODO 检查摄像头是否正常
    ElMessage({
      message: '确保摄像头画面清晰，且没有遮挡',
      type: 'warning',
    })
    return;
  }
}
</script>
```