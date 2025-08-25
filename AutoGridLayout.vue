<template>
  <!--
    网格容器，绑定动态样式 gridVars（根据不同屏幕设置列数和间距）
    插槽用于插入任意内容（如卡片、图块等）
  -->
  <div class="auto-grid-container" ref="containerRoot" :style="{ ...gridVars, paddingRight: hasScrollbar ? '8px' : '0px' }">
    <slot />
  </div>
</template>

<script>
export default {
  name: "AutoGrid",
  props: {
    // 定义不同断点下的列数，支持 tailwind 风格的命名
    cols: {
      type: Object,
      default: () => ({
        default: 1, // <640px 的默认列数
        sm: 2, // ≥640px
        md: 3, // ≥768px
        lg: 4, // ≥1024px
        xl: 5, // ≥1280px
        "2xl": 6, // ≥1536px
        "3xl": 6, // ≥1920px
        "4xl": 8, // ≥2560px
        "5xl": 12, // ≥3840px
      }),
    },
    // 列间距
    gap: {
      type: String,
      default: "16px",
    },
  },
  data() {
    return {
      hasScrollbar: false, // 是否有垂直滚动条
      mutationObserver: null, // 用于监听插槽内容变化的 MutationObserver
    };
  },
  computed: {
    // 将 props 中的 cols 转换为 CSS 变量，以便在样式中使用
    gridVars() {
      const v = {
        "--grid-gap": this.gap,
        "--grid-cols": this.cols.default,
        "--grid-cols-sm": this.cols.sm,
        "--grid-cols-md": this.cols.md,
        "--grid-cols-lg": this.cols.lg,
        "--grid-cols-xl": this.cols.xl,
        "--grid-cols-2xl": this.cols["2xl"],
        "--grid-cols-3xl": this.cols["3xl"],
        "--grid-cols-4xl": this.cols["4xl"],
        "--grid-cols-5xl": this.cols["5xl"],
      };
      return v;
    },
  },
  mounted() {
    // 首次挂载时计算一次
    this.$nextTick(() => {
      this.updateColumns();
      this.observeSlotChanges();
    });

    // 监听窗口 resize，动态更新 maxColumns、computedGap
    window.addEventListener("resize", this.updateColumns);
  },

  beforeDestroy() {
    window.removeEventListener("resize", this.updateColumns);
  },
  methods: {
    updateColumns() {
      if (!this.$refs.containerRoot) return;

      const el = this.$refs.containerRoot;
      this.hasScrollbar = el.scrollHeight > el.clientHeight;
    },
    observeSlotChanges() {
      const container = this.$refs.containerRoot;
      if (!container) return;

      this.mutationObserver = new MutationObserver(() => {
        this.$nextTick(this.updateColumns);
      });

      this.mutationObserver.observe(container, {
        childList: true,
        subtree: true,
      });
    },
  },
};
</script>

<style scoped>
/*
  基础网格容器样式：
  - 使用 CSS grid 布局
  - 列数、间距通过 CSS 变量控制
*/
.auto-grid-container {
  width: 100%;
  display: grid;
  grid-gap: var(--grid-gap); /* 设置间距 */
  grid-template-columns: repeat(var(--grid-cols), 1fr); /* 默认列数 */

  align-content: start; /* 防止行被均匀拉伸填满容器高度 */
  align-items: start; /* 网格项在单元格内从顶部对齐，不被拉伸 */
  /* 可选：让自动生成的行高度按照内容决定 */
  grid-auto-rows: min-content;
}

.auto-grid-container > * {
  /* 关键：允许自己在 Grid 里收缩 */
  min-width: 0;
  /* min-height: 0; */
}

/* 可选：自定义滚动条，仅 WebKit/Chromium 生效 */
.auto-grid-container::-webkit-scrollbar {
  width: 8px;
}
.auto-grid-container::-webkit-scrollbar-track {
  background: transparent;
}
.auto-grid-container::-webkit-scrollbar-track-piece {
  background: transparent !important;
}
.auto-grid-container::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 0, 0.3);
  border-radius: 4px;
}

/* sm ≥ 640px，小屏手机横屏或小平板 */
@media (min-width: 640px) {
  .auto-grid-container {
    grid-template-columns: repeat(var(--grid-cols-sm), 1fr);
  }
}

/* md ≥ 768px，常见平板设备 */
@media (min-width: 768px) {
  .auto-grid-container {
    grid-template-columns: repeat(var(--grid-cols-md), 1fr);
  }
}

/* lg ≥ 1024px，笔记本或桌面小屏 */
@media (min-width: 1024px) {
  .auto-grid-container {
    grid-template-columns: repeat(var(--grid-cols-lg), 1fr);
  }
}

/* xl ≥ 1280px，标准桌面宽度（如1080p） */
@media (min-width: 1280px) {
  .auto-grid-container {
    grid-template-columns: repeat(var(--grid-cols-xl), 1fr);
  }
}

/* 2xl ≥ 1536px，较大屏幕 */
@media (min-width: 1536px) {
  .auto-grid-container {
    grid-template-columns: repeat(var(--grid-cols-2xl), 1fr);
  }
}

/* 3xl ≥ 1920px，全高清显示器或大屏 */
@media (min-width: 1920px) {
  .auto-grid-container {
    grid-template-columns: repeat(var(--grid-cols-3xl), 1fr);
  }
}

/* 4xl ≥ 2560px，2K 宽屏显示器或双屏 */
@media (min-width: 2560px) {
  .auto-grid-container {
    grid-template-columns: repeat(var(--grid-cols-4xl), 1fr);
  }
}

/* 5xl ≥ 3840px，4K 或更大屏幕显示器 */
@media (min-width: 3840px) {
  .auto-grid-container {
    grid-template-columns: repeat(var(--grid-cols-5xl), 1fr);
  }
}
</style>
