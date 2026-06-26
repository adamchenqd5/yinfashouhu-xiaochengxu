<script>
export default {
  onLaunch: function () {
    console.log('银发守护者 - App启动')
  },
  onShow: function () {
    console.log('App显示')
  },
  onHide: function () {
    console.log('App隐藏')
  },
}
</script>

<style>
/* 全局样式 */
page {
  font-family: -apple-system, BlinkMacSystemFont, "Helvetica Neue", "PingFang SC", "Microsoft YaHei", sans-serif;
  -webkit-font-smoothing: antialiased;
  background: #EDEDED;
}

/* 重置 button 样式 */
button {
  margin: 0;
  padding: 0;
  background: none;
  border: none;
  outline: none;
}

button::after {
  border: none;
}
</style>
