<script setup lang='ts'>
import { postRoomsnAPI } from '@/package_nzgx/services/openbook'
import { useMemberStore } from '@/package_nzgx/stores'
import type { LoginResult } from '@/package_nzgx/types/member'
import { useWebSocketStore } from '@/package_nzgx/stores'
import { onMounted, onUnmounted, ref } from 'vue'
import { WebSocketService } from '@/package_nzgx/services/WebSocketService'
import { initAllInfo } from '@/package_nzgx/services/initInfo'
import { AuthorizationDM } from '@/services/play'
import { getInfoById, updateInfoById } from '@/package_nzgx/services/updateInfo'
import { allClues } from '@/package_nzgx/services/clues'

const now = new Date();
function formatDate(date) {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    const hours = String(date.getHours()).padStart(2, '0');
    const minutes = String(date.getMinutes()).padStart(2, '0');
    const seconds = String(date.getSeconds()).padStart(2, '0');

    return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
}
const futureDate = new Date(now.getTime() + 6 * 60 * 60 * 1000);
const formattedFutureDate = formatDate(futureDate);
console.log("延后六小时的时间:", formattedFutureDate);

const memberStore = useMemberStore()
const webSocketStore = useWebSocketStore();

const code = ref('')
const login = async () => {
  memberStore.setInfo(initAllInfo)
  const res = await AuthorizationDM(code.value)
  console.log(res.token)
  loginSuccess(res)
}

const getCode = () => {

  // 获取code
  uni.login({
    provider: 'weixin',
    success: res => {
      console.log(res.code);
      code.value = res.code
    }
  });
}
const loginSuccess = (profile: LoginResult) => {
  //保存会员信息
  memberStore.setProfile(profile)

  uni.showToast({ icon: 'success', title: '登录成功' })

}
const openBook = async () => {
  uni.showLoading({
    title: '加载中'
  });
  const res = await postRoomsnAPI({
    name: '测试房间',
    end_time: formattedFutureDate
  })
  console.log(res.data.room_number);
  memberStore.setRoomId(res.data.room_number)

  setTimeout(() => {
    joinGame('gm')
    uni.hideLoading()
  }, 2000);
}
const joinRoom2 = (_role: string) => {
  memberStore.setVirtualRoleId(_role)
  uni.navigateTo({
    url: `/package_nzgx/pages/player/player`
  })
}
const joinGame = (_role: string) => {

  memberStore.setVirtualRoleId(_role);

  // 创建 WebSocket 连接
  const wsService = new WebSocketService(`token=${memberStore.profile.token}&room_id=${memberStore.roomId}&virtual_role_id=gm`);
  wsService.connect()
  // 监听 WebSocket 连接成功事件
  wsService.onOpen = () => {
    console.log("WebSocket 连接成功");

    // 连接成功后执行后续操作
    webSocketStore.gameWebSocketService = wsService;

    setTimeout(() => {

      initInfo();
    }, 500);
  };

  // 监听连接错误或关闭事件
  wsService.onError = (error) => {
    console.error("WebSocket 连接失败", error);
    // 在这里可以添加错误处理逻辑
  };

  wsService.onClose = () => {
    console.log("WebSocket 连接已关闭");
    // 在这里可以添加连接关闭后的处理逻辑
  };
}

const initInfo = async() => {
  webSocketStore.gameSend(
    initAllInfo
  )

}

//更新原始流程/线索集信息
const updateOriInfo = () => {
  updateInfoById(1,'flowInfo',initAllInfo)
  updateInfoById(2,'clue',allClues)
}
onMounted(() => {
  if (code.value === '') {
    getCode()
  }
});

onUnmounted(() => {
});
</script>

<template>
  <view class="flex-column-sb ">
    <button @tap="login">用户登录</button>
    <!-- <button @tap="code='0b3Guqml24nqYd4Ex3nl2rm2fn1Guqmk';login()">用户浏览器登录</button> -->
    <button @tap="openBook">DM创建房间并加入游戏</button>
    <button @tap="joinGame('gm')">DM加入游戏</button>

    <button @tap="joinRoom2('1')">玩家1加入游戏</button>
    <button @tap="joinRoom2('2')">玩家2加入游戏</button>
    <button @tap="joinRoom2('3')">玩家3加入游戏</button>
    <button @tap="joinRoom2('4')">玩家4加入游戏</button>
    <button @tap="joinRoom2('5')">玩家5加入游戏</button>
    <button @tap="joinRoom2('6')">玩家6加入游戏</button>
    <button @tap="initInfo">初始化游戏数据</button>
    <button @tap="updateOriInfo">向数据库更新原始流程信息和线索</button>

  </view>
</template>

<style scoped></style>