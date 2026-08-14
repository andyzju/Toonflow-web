<template>
  <div class="placeholder">
    <div class="box">
      <div class="glyph">技</div>
      <h1>技能库</h1>
      <p>这里将承载可复用的文旅 Skill：范式、风格、提示词与审核红线。沉淀一次，处处复用。</p>
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
    background: linear-gradient(145deg, #c1442f, #8f2f21);
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
