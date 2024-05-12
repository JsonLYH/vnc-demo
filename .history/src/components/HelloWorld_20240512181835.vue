<template>
  <div class="vnc-container" id="screen">

  </div>
</template>

<script>
import RFB from '@novnc/novnc/core/rfb'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      rfb: null,
      desktopName: '',
      isFullscreen: false,
    }
  },
  mounted() {
    // document.getElementById('sendCtrlAltDelButton').onclick =
    //   this.sendCtrlAltDel

    this.connectVnc()
  },

  //销毁 断开连接
  beforeDestroy() {
    this.rfb && this.rfb._disconnect()
  },

  methods: {
    extractUrlInfo(urlString) {
  try {
    // 创建URL对象
    const url = new URL(urlString);

    // 提取主机名和端口
    const host = url.hostname;
    const port = url.port ? parseInt(url.port, 10) : null; // 如果URL中没有显式指定端口，port将为空字符串，因此我们需要检查并转换

    // 提取特定查询参数
    const queryParams = new URLSearchParams(url.search);
    const token = queryParams.get('token');

    // 返回结果对象
    return {
      host,
      port,
      token
    };
  } catch (error) {
    console.error('Invalid URL:', error);
    return null;
  }
},
    sendCtrlAltDel() {
      this.rfb.sendCtrlAltDel()
      return false
    },

    //连接
    connectVnc() {
      const PASSWORD = '123456'//VNC Server 密码
      const url = 'ws://192.168.2.99:6080/websockify'
      this.rfb = new RFB(document.getElementById('screen'), url, {
        // 向vnc 传递的一些参数，比如说虚拟机的开机密码等
        credentials: { password: PASSWORD },
      })
      this.rfb.addEventListener('connect', this.connectedToServer)
      this.rfb.addEventListener('disconnect', this.disconnectedFromServer)
      this.rfb.scaleViewport = false //scaleViewport指示是否应在本地扩展远程会话以使其适合其容器。禁用时，如果远程会话小于其容器，则它将居中，或者根据clipViewport它是否更大来处理。默认情况下禁用。
      this.rfb.resizeSession = false //是一个boolean指示是否每当容器改变尺寸应被发送到调整远程会话的请求。默认情况下禁用
      console.log(this.rfb)
    },

    status(text) {
      document.getElementById('status').textContent = text
    },

    connectedToServer(e) {
      // this.status('Connected to ' + this.desktopName)
      console.log('success', e)
    },

    disconnectedFromServer(e) {
      if (e.detail.clean) {
        // this.status('Disconnected')

        console.log('clean', e.detail.clean)
        //根据 断开信息的msg.detail.clean 来判断是否可以重新连接
        // this.connectVnc()
      } else {
        // this.status('Something went wrong, connection is closed')
        console.log('链接失败,重新链接中-------' + this.wsURL)
        // this.connectVnc()
      }
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.vnc-container{
  width: 100vw;
  height: 100vh;
}
</style>
