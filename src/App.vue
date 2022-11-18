<template>
  <div class="App">
    <div class="main">
      <div id="wavesurfer" class="wavesurfer"></div>
      <div id="wave-timeline" ref="wave-timeline"></div>
      <div class="editor">
        <div class="controller">
          <div>当前播放：{{ curMusic.name }}</div>
          <a-select
            ref="select"
            v-model:value="selectValue"
            style="width: 120px"
            @focus="focus"
            @change="handleChange"
          >
            <a-select-option value="1">顺序播放</a-select-option>
            <a-select-option value="2">循环播放</a-select-option>
            <a-select-option value="3">随机播放</a-select-option>
          </a-select>
          <a-button
            class="btn"
            type="primary"
            shape="circle"
            size="large"
            @click="pre"
          >
            <template #icon><step-backward-outlined /></template>
          </a-button>
          <a-button
            class="btn"
            type="primary"
            shape="circle"
            size="large"
            @click="playPause"
          >
            <template #icon> <play-circle-outlined /> </template>
          </a-button>
          <a-button
            class="btn"
            type="primary"
            shape="circle"
            size="large"
            @click="next"
          >
            <template #icon> <step-forward-outlined /> </template>
          </a-button>
          <div class="sliderBox">
            音量：<a-slider
              class="slider"
              v-model:value="curMusicVolume"
              :min="1"
              :max="100"
            />
          </div>
        </div>
        <div class="list">
          <a-list size="small" item-layout="horizontal" :data-source="Listdata">
            <template #renderItem="{ item, index }">
              <a-list-item @click="handleItem(index)">
                歌名：{{ item.name }}，专辑：{{ item.al.name }} ，歌手：{{
                  item.ar[0].name
                }}</a-list-item
              >
            </template>
            <template #header>
              <div>音乐列表</div>
            </template>
          </a-list>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watchEffect, computed, watch } from 'vue';
import Wavesurfer from 'wavesurfer.js';
import Timeline from 'wavesurfer.js/dist/plugin/wavesurfer.timeline.js';
import {
  StepBackwardOutlined,
  StepForwardOutlined,
  PlayCircleOutlined
} from '@ant-design/icons-vue';
import * as _ from 'lodash';

//记录挂载后的实例
let wavesurferVue;

//音乐列表
const Listdata = ref([]);

//当前音乐地址
// const curMusicUrl = computed(() => {
//   return `https://music.163.com/song/media/outer/url?id=${curMusicId.value}.mp3`;
// });
const curMusicUrl = ref('');
//当前列表序号
const curMusicIndex = ref(0);
//当前播放音乐
const curMusic = computed(() => Listdata.value[curMusicIndex.value]);
//当前音量大小
const curMusicVolume = ref(100);
//播放模式
const selectValue = ref('1');
onMounted(() => {
  //实例化音频器
  const wavesurfer = Wavesurfer.create({
    container: '#wavesurfer',
    waveColor: '#409EFF',
    progressColor: '#409EFF',
    //   backgroundColor:'#1d2741',
    backend: 'MediaElement',
    mediaControls: false,
    audioRate: '1',
    //使用时间轴插件
    plugins: [
      Timeline.create({
        container: '#wave-timeline',
        secondaryColor: '#409EFF', //次要时间标签颜色
        secondaryFontColor: '#409EFF',
        primaryColor: '#409EFF', //主要时间标签颜色
        primaryFontColor: '#409EFF'
        // labelPadding: 2
      })
    ]
  });
  wavesurferVue = wavesurfer;
  //当前播放完下一首
  wavesurfer.on('finish', () => {
    next();
  });
  //一旦歌曲路径发生变化则重新载入播放
  watchEffect(() => {
    if (curMusicUrl.value !== '') {
      wavesurferVue.load(curMusicUrl.value);
      playPause();
    }
  });

  //音量控制
  const MysetVolume = _.debounce(() => {
    console.log(curMusicVolume.value / 100);
    wavesurferVue.setVolume(curMusicVolume.value / 100);
  }, 250);
  watch(curMusicVolume, MysetVolume);
});

//获取歌单
const fetchList = async () => {
  const result = await fetch(
    'http://localhost:3000/playlist/track/all?id=7753002568' +
      '&timestamp=' +
      Date.now()
  );
  if (result.status === 200) {
    result.text().then((res) => {
      Listdata.value = JSON.parse(res).songs;
      console.log(Listdata.value);
      //默认先获取列表第一首的id
      fetchMusicUrl(Listdata.value[0].id);
    });
  }
};

//获取歌曲
const fetchMusicUrl = async (id) => {
  const result = await fetch(
    `http://localhost:3000/song/url/v1?id=${id}&level=exhigh&timestamp=` +
      Date.now(),
    { credentials: 'include' }
  );
  if (result.status === 200) {
    result.text().then((res) => {
      curMusicUrl.value = JSON.parse(res).data[0].url;
    });
  }
};
fetchList();
//一旦index变化则重新获取url更新url
watchEffect(() => {
  if (Listdata.value[curMusicIndex.value])
    fetchMusicUrl(Listdata.value[curMusicIndex.value].id);
});
// //获取游客cookie
// const fetchCookie = async () => {
//   const result = await fetch(`http://localhost:3000/register/anonimous`);
//   if (result.status === 200) {
//     result.text().then((res) => {
//       console.log(JSON.parse(res).cookie);
//     });
//   }
// };
// fetchCookie();

//获取随机数
const getRandom = (min, max) => Math.floor(Math.random() * (max - min)) + min;
//播放暂停上下首
const playPause = function () {
  wavesurferVue.playPause();
};

const pre = () => {
  switch (selectValue.value) {
    case '2':
      //执行语句；
      wavesurferVue.play(0);
      break;
    case '3':
      //执行语句；
      curMusicIndex.value = getRandom(0, Listdata.value.length);
      break;
    default:
      if (curMusicIndex.value > 0) curMusicIndex.value--;
      else {
        curMusicIndex.value = Listdata.value.length - 1;
      }
  }
};
const next = () => {
  switch (selectValue.value) {
    case '2':
      //执行语句；
      wavesurferVue.play(0);
      break;
    case '3':
      //执行语句；
      curMusicIndex.value = getRandom(0, Listdata.value.length);
      break;
    default:
      //执行语句；
      if (curMusicIndex.value < Listdata.value.length - 1)
        curMusicIndex.value++;
      else {
        curMusicIndex.value = 0;
      }
  }
};

//点击列表元素
const handleItem = (index) => {
  curMusicIndex.value = index;
};
</script>

<style scoped lang="less">
.App {
  .main {
    display: flex;
    width: 800px;
    background-color: rgba(255, 255, 255);
    flex-direction: column;

    .wavesurfer {
      flex: 1;
    }

    .editor {
      margin-top: 20px;
      flex: 1;

      .controller {
        display: flex;
        align-items: center;
        justify-content: space-around;
        .btn {
          margin-top: 10px;
          margin-left: 20px;
        }

        .sliderBox {
          display: flex;
          align-items: center;
          .slider {
            width: 100px;
          }
        }
      }
    }
  }
}
</style>
