<template>
  <div class="placeholder">
    <div class="box">
      <div class="glyph">发</div>
      <h1>发布中心</h1>
      <p>这里将承载成片发布：多平台分发、版本管理、封面与字幕导出。让每一段文旅漫剧抵达观众。</p>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.placeholder {
  height: 100%;
  display: grid;
  place-items: center;
}
.box {
  text-align: center;
  .glyph {
    width: 56px;
    height: 56px;
    margin: 0 auto 16px;
    border-radius: 14px;
    display: grid;
    place-items: center;
    font-family: "Kaiti SC", "STKaiti", "KaiTi", "楷体", serif;
    font-size: 28px;
    color: #fff;
    background: linear-gradient(145deg, #3e7d6a, #2a5c4f);
  }
  h1 {
    font-family: "Kaiti SC", "STKaiti", "KaiTi", "楷体", serif;
    font-size: 28px;
    font-weight: 700;
    color: var(--td-text-color-primary);
  }
  p {
    max-width: 420px;
    margin: 10px auto 0;
    color: var(--td-text-color-secondary);
    font-size: 14px;
    line-height: 1.7;
  }
}
</style>
