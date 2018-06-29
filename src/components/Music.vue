<template>
  <div class="music">
    <header>
      <h1>可视化音乐</h1>
    </header>
    <div class="left">
      <ul>
        <li>music 1</li>
      </ul>
    </div>
    <div class="right"></div>
  </div>
</template>
<script>
export default {
  data () {
    return {
    }
  }
}
</script>
<style lang="scss" scoped>
.music {
  font: 14px "Lucida Grande", Helvetica, Arial, sans-serif;
  background: #000;
  color: #fff;
  text-align: center;
  height: 1000px;
}
header, .left, .right {
  position: absolute;
  border:solid #fff 1px;
}
header {
  left: 10px;
  top: 10px;
  right: 10px;
  height: 150px;
  h1 {
    font-size: 40px;
    height: 60px;
    line-height: 60px;
  }
}
.left {
  left: 10px;
  top: 170px;
  bottom: 10px;
  width: 200px;
  ul {
    overflow: auto;
    li {
      height: 30px;
      line-height: 30px;
      cursor: pointer;
      list-style: none;
      &.selected {
        color: green;
      }
    }
  }
}
.right {
  left: 220px;
  top: 170px;
  right: 10px;
  bottom: 10px;
}
</style>
