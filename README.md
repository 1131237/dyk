<!doctype html>
<html lang="zh-CN">
<head>
<meta charset="utf-8" />
<meta
  name="viewport"
  content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no"
/>
<title>可移动的爱心</title>
<style>
  html, body {
    height: 100%;
    margin: 0;
    background: #fff5f7; /* 柔和背景 */
    -webkit-tap-highlight-color: transparent;
    touch-action: none; /* 兼容微信手势控制 */
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }

  .stage {
    position: relative;
    width: 100%;
    height: 100%;
    overflow: hidden;
  }

  /* 爱心容器，便于拖动与居中 */
  .heart-wrap {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
    width: 140px;
    height: 130px;
    cursor: grab;
    touch-action: none; /* 确保触摸事件都给到此元素 */
  }

  /* 用伪元素拼出爱心（两圆+一个方块旋转） */
  .heart {
    position: absolute;
    width: 100px;
    height: 100px;
    left: 20px;  /* 让图形在容器中居中些 */
    top: 15px;
    transform: rotate(-45deg);
  }

  .heart::before,
  .heart::after {
    content: "";
    position: absolute;
    width: 100px;
    height: 100px;
    background: linear-gradient(135deg, #ff6b81 0%, #ff2d55 100%);
    border-radius: 50%;
    box-shadow: 0 10px 20px rgba(255,45,85,0.25), inset 0 -6px 12px rgba(255,255,255,0.25);
  }

  .heart::before {
    left: 0;
    top: 0;
  }

  .heart::after {
    left: 50px;
    top: 0;
  }

  /* 下方的方块（拼接出爱心尖角） */
  .heart-core {
    position: absolute;
    width: 100px;
    height: 100px;
    background: linear-gradient(135deg, #ff6b81 0%, #ff2d55 100%);
    left: 0;
    top: 50px;
    transform: rotate(45deg);
    border-bottom-left-radius: 20px;
    box-shadow: 0 10px 20px rgba(255,45,85,0.25), inset 0 -6px 12px rgba(255,255,255,0.25);
  }

  /* 小光点动画，提升质感 */
  .shine {
    position: absolute;
    width: 18px;
    height: 18px;
    left: 78px;
    top: 22px;
    background: radial-gradient(circle at 30% 30%, rgba(255,255,255,0.95), rgba(255,255,255,0.2) 70%, transparent 71%);
    border-radius: 50%;
    filter: blur(0.2px);
    animation: twinkle 1.8s ease-in-out infinite;
    transform: rotate(45deg);
  }

  @keyframes twinkle {
    0%, 100% { opacity: 0.7; transform: rotate(45deg) scale(1); }
    50%      { opacity: 1;   transform: rotate(45deg) scale(1.08); }
  }

  /* 底部提示 */
  .hint {
    position: absolute;
    left: 50%;
    bottom: 22px;
    transform: translateX(-50%);
    color: #ff2d55;
    font-size: 14px;
    opacity: 0.9;
    user-select: none;
  }
</style>
</head>
<body>
  <div class="stage" id="stage">
    <div class="heart-wrap" id="heartWrap" aria-label="可拖动的爱心" role="img">
      <div class="heart">
        <div class="heart-core"></div>
        <div class="shine"></div>
      </div>
    </div>
    <div class="hint">按住并拖动爱心移动位置</div>
  </div>

<script>
(function () {
  // 简单双指缩放支持（可选）
  const wrap = document.getElementById('heartWrap');
  const stage = document.getElementById('stage');

  let isDragging = false;
  let startX = 0, startY = 0;
  let wrapStartLeft = 0, wrapStartTop = 0;

  // 记录 transform: translate(-50%,-50%) 初始居中偏移，拖动时转成绝对定位
  function ensureAbsolutePosition() {
    const style = window.getComputedStyle(wrap);
    if (style.transform !== 'none') {
      const rect = wrap.getBoundingClientRect();
      const stageRect = stage.getBoundingClientRect();
      // 计算为相对 stage 的 left/top
      wrap.style.left = (rect.left - stageRect.left) + 'px';
      wrap.style.top  = (rect.top  - stageRect.top)  + 'px';
      wrap.style.transform = 'none';
    }
  }

  function onPointerDown(clientX, clientY) {
    ensureAbsolutePosition();
    isDragging = true;
    wrap.style.cursor = 'grabbing';
    const rect = wrap.getBoundingClientRect();
    const stageRect = stage.getBoundingClientRect();
    wrapStartLeft = rect.left - stageRect.left;
    wrapStartTop  = rect.top  - stageRect.top;
    startX = clientX;
    startY = clientY;
  }

  function onPointerMove(clientX, clientY) {
    if (!isDragging) return;
    const dx = clientX - startX;
    const dy = clientY - startY;

    // 限制在舞台内
    const stageRect = stage.getBoundingClientRect();
    const rect = wrap.getBoundingClientRect();
    let newLeft = wrapStartLeft + dx;
    let newTop  = wrapStartTop  + dy;

    // 边界限制
    const maxLeft = stageRect.width - rect.width;
    const maxTop  = stageRect.height - rect.height;
    newLeft = Math.max(0, Math.min(maxLeft, newLeft));
    newTop  = Math.max(0, Math.min(maxTop,  newTop));

    wrap.style.left = newLeft + 'px';
    wrap.style.top  = newTop  + 'px';
  }

  function onPointerUp() {
    isDragging = false;
    wrap.style.cursor = 'grab';
  }

  // 触摸事件（兼容微信）
  stage.addEventListener('touchstart', function (e) {
    const t = e.targetTouches[0];
    if (!t) return;
    onPointerDown(t.clientX, t.clientY);
    e.preventDefault();
  }, { passive: false });

  stage.addEventListener('touchmove', function (e) {
    const t = e.targetTouches[0];
    if (!t) return;
    onPointerMove(t.clientX, t.clientY);
    e.preventDefault();
  }, { passive: false });

  stage.addEventListener('touchend', function () {
    onPointerUp();
  });

  // 鼠标事件（方便电脑测试）
  stage.addEventListener('mousedown', function (e) {
    onPointerDown(e.clientX, e.clientY);
  });
  window.addEventListener('mousemove', function (e) {
    onPointerMove(e.clientX, e.clientY);
  });
  window.addEventListener('mouseup', onPointerUp);

  // 防止页面滚动影响拖动体验
  document.addEventListener('gesturestart', function (e) { e.preventDefault(); });

  // 初始：轻微弹跳效果
  setTimeout(() => {
    const y0 = wrap.offsetTop;
    let t = 0;
    const timer = setInterval(() => {
      t += 1/60;
      const dy = Math.sin(t * 6) * 1.5; // 小幅度
      wrap.style.top = (y0 + dy) + 'px';
      if (t > 1.2) {
        wrap.style.top = y0 + 'px';
        clearInterval(timer);
      }
    }, 1000/60);
  }, 200);
})();
</script>
</body>
</html>
