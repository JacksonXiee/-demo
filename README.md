# Vue 3 + Vite + wavesurfer

## 项目介绍

项目结合 vue3 + vite + wavesurfer + NeteaseCloudMusicApi + ant design vue 完成了一个简易音频播放器 demo，实现的功能如下：

- 歌单列表
- 根据不同播放顺序播放音乐
- 调整音量
- 音频波形图显示
- 进度条也呈现在音频波形图上
- 根据列表切换歌曲

## 启动

在 NeteaseCloudMusicApi 文件夹下打开终端，执行以下命令，将服务端跑起来，以通过伪造请求头去获取音乐 API。
` npm i `
﻿﻿`node app.js `

在到项目主目录下打开终端，执行以下代码，将页面跑起来
` npm i `
﻿﻿`npm run dev `

## 注意

- 服务端默认是 3000 端口
- 前端请求端口写死也是 3000，请保证服务端端口为 3000，以保证请求能正常获取
