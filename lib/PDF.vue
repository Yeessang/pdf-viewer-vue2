<template>
  <section
    class="pdf-wrapper"
  >
    <section
      ref="pdfWrapper"
      class="relative w-full h-full pt-[50px] rounded-[10px] overflow-hidden border border-slate-100"
      :key="renderKey"
    >
      <div class="pdf-toolbar">
        <Transition name="float-fade">
          <div
            v-show="showSmallMenu"
            ref="menuFloating"
            :class="[smallMenu ? 'pdf-toolbar-container-small' : 'pdf-toolbar-container']"
            :style="{ transformOrigin: `${menuFloatXY.x}px ${menuFloatXY.y}px` }"
          >
            <ul class="toolbar-group">
              <li class="toolbar-item" :class="[showThumbnail && 'toolbar-item-active']" @click="showThumbnail = !showThumbnail">
                <i class="icon iconfont icon-xiaosuolvetu"></i>
                <span>缩略图</span>
              </li>
              <li class="toolbar-item" :class="[showCatalog && 'toolbar-item-active']" @click="showCatalog = !showCatalog">
                <i class="icon iconfont icon-mulu"></i>
                <span>目录</span>
              </li>
            </ul>
            <ul class="toolbar-group">
              <li class="toolbar-item" :class="[pageScale === 'page-actual' && 'toolbar-item-active']" @click="changePageScale('page-actual')">
                <i class="icon iconfont icon-yuanchicun"></i>
                <span>原尺寸</span>
              </li>
              <li class="toolbar-item" :class="[pageScale === 'page-width' && 'toolbar-item-active']" @click="changePageScale('page-width')">
                <i class="icon iconfont icon-zishiyingkuandu"></i>
                <span>适应宽度</span>
              </li>
              <li class="toolbar-page">
                <input
                  type="number" 
                  class="pdf-input text-slate-600" 
                  max="40" 
                  min="1" 
                  :value="currentScale" 
                  @keyup.enter="inputScaleHandler"
                />
                <span class="toolbar-page-percent">%</span>
              </li>
              <li class="toolbar-item" :class="[pageScale === 'page-height' && 'toolbar-item-active']" @click="changePageScale('page-height')">
                <i class="icon iconfont icon-zishiyinggaodu"></i>
                <span>适应高度</span>
              </li>
              <li class="toolbar-page" v-show="!smallMenu">
                <input
                  type="number" 
                  class="pdf-input text-slate-600" 
                  max="40" 
                  min="1" 
                  :value="currentPage" 
                  @keyup.enter="pagePressHandler"
                />
                <span class="align-top mx-[5px]">/</span>
                <span class="toolbar-page-sum">{{ totalPage }}</span>
              </li>
              <li class="toolbar-item" :class="[spreadMode === 0 && 'toolbar-item-active']" @click="changeSpreadMode(0)">
                <i class="icon iconfont icon-danyeshitu"></i>
                <span>单页视图</span>
              </li>
              <li class="toolbar-item" :class="[spreadMode === 1 && 'toolbar-item-active']" @click="changeSpreadMode(1)">
                <i class="icon iconfont icon-shuangyeshitu"></i>
                <span>双页视图</span>
              </li>
              <li class="toolbar-item" :class="[rotation > 0 && 'toolbar-item-active']" @click="changeRotation">
                <i class="icon iconfont icon-rotate" :style="{ transform: `rotate(${rotation}deg)` }"></i>
                <span>旋转</span>
              </li>
              <li class="toolbar-item toolbar-item-active" @click="changeScrollMode">
                <i class="icon iconfont icon-hengxianggundong1" :class="[scrollMode === 0 && 'pdf-rotate-90']"></i>
                <span>{{ scrollMode === 0 ? '竖向滚动' : '横向滚动' }}</span>
              </li>
            </ul>
            <ul class="toolbar-group">
              <li class="toolbar-item" ref="reference" :class="[showSearch && 'toolbar-item-active']" @click="toggleSearch">
                <i class="icon iconfont icon-chaxun"></i>
                <span>查询</span>
              </li>
              <li class="toolbar-item" @click="print">
                <i class="icon iconfont icon-dayinji_o"></i>
                <span>打印</span>
              </li>
            </ul>
          </div>
        </Transition>
        <div 
          v-if="smallMenu"
          class="toolbar-page text-[11px] pdf-small-menu"
        >
          <input
            type="number" 
            class="pdf-input text-slate-600" 
            max="40" 
            min="1" 
            :value="currentPage" 
            @keyup.enter="pagePressHandler"
          />
          <span class="align-top mx-[5px]">/</span>
          <span class="toolbar-page-sum">{{ totalPage }}</span>
        </div>
        <div class="pdf-menu-setting toolbar-item" v-if="smallMenu" ref="menuReference" :class="[showSmallMenu && 'toolbar-item-active']" @click="toggleMenu">
          <div class="menu-group-setting" :class="[showSmallMenu && 'menu-group-setting-active']"></div>
        </div>
        <!-- <div 
          v-if="smallMenu" 
          ref="menuReference" 
          class="toolbar-item w-[30px] absolute right-0 h-[30px] leading-[30px] py-0 mt-[10px]" 
          :class="[showSmallMenu && 'toolbar-item-active']"
          @click="toggleMenu"
        >
          <i class="icon iconfont icon-gengduo"></i>
        </div> -->
      </div>
      <div class="absolute w-full bottom-[0] top-[50px] flex bg-slate-100">
        <Transition name="transform">
          <div class="pdf-thumbnail" v-show="showThumbnail || showCatalog">
            <PDFTree
              v-show="showCatalog"
              :tree-data="catalogTreeData"
              node-key="title"
              @node-click="clickCatalog"
            ></PDFTree>
            <div
              v-show="showThumbnail" 
              class="pdf-thumbnail-container" 
              ref="thumbnailContainer" 
            ></div>
          </div>
        </Transition>
        <div class="flex-1 relative">
          <div
            ref="pdfContainer" 
            class="pdf-container absolute w-full h-full overflow-auto"
          >
            <div class="pdfViewer"></div>
          </div>
        </div>
      </div>
      <div
        v-if="loadingPercentVisible"
        class="absolute top-0 bottom-0 right-0 left-0 bg-[rgba(255,255,255,.9)] z-[100]"
      >
        <div class="pdf-small-menu">
          <div class="relative w-[230px] h-[24px] rounded-[12px] overflow-hidden bg-slate-200">
            <div class="absolute w-full h-full bg-violet-300 transition-all" :style="{ transform: `translateX(${loadingPercent - 100}%)` }"></div>
          </div>
          <div class="mt-[10px] text-[12px] text-center text-gray-500">
            准备加载文档中，当前进度：{{ loadingPercent }}%
          </div>
        </div>
      </div>
      <div
        v-if="showPrint"
        class="absolute top-0 bottom-0 right-0 left-0 bg-[rgba(255,255,255,.9)] z-[100]"
      >
        <div class="pdf-small-menu">
          <div class="relative w-[230px] h-[24px] rounded-[12px] overflow-hidden bg-slate-200">
            <div class="absolute w-full h-full bg-violet-300 transition-all" :style="{ transform: `translateX(${progress - 100}%)` }"></div>
          </div>
          <div class="mt-[10px] text-[12px] text-center text-gray-500">
            准备打印文档中，当前进度：{{ progress }}%
          </div>
          <div class="mt-[10px] text-center">
            <button class="text-[12px] text-violet-400 py-[3px] px-[5px] rounded-[5px] transition-all hover:text-purple-400 bg-transparent border-none outline-none bg-transparent" @click="abort">取消打印</button>
          </div>
        </div>
      </div>
      <Transition name="float-fade">
        <div
          v-show="showSearch"
          ref="floating" 
          class="w-[165px] min-h-[90px] px-[6px] py-[5px] absolute bg-white rounded-[6px] z-[20] search-float transition-all duration-300 overflow-hidden"
          :style="{ transformOrigin: `${searchFloatXY.x}px ${searchFloatXY.y}px` }"
        >
          <div class="flex items-center">
            <input 
              type="text" 
              class="w-[100px] rounded-[3px] h-[27px] focus:border-violet-300 outline-none px-[5px] py-[3px] text-[12px] mr-[5px]"
              v-model="searchKey"
              @input="input"
              @keyup.enter="findNext"
            />
            <i class="icon iconfont icon-jiantou_xiangshang pdf-search-toggle" @click="findPrev"></i>
            <i class="icon iconfont icon-jiantou_xiangxia pdf-search-toggle" @click="findNext"></i>
          </div>
          <div class="flex flex-wrap text-[12px] m-auto text-slate-500">
            <div class="pdf-search-option" :class="[searchOptions.highlightAll && 'toolbar-item-active']" @click="toggleSearchOption('highlightAll')">全部高亮显示</div>
            <div class="pdf-search-option ml-[1px]" :class="[searchOptions.caseSensitive && 'toolbar-item-active']" @click="toggleSearchOption('caseSensitive')">区分大小写</div>
            <div class="pdf-search-option" :class="[searchOptions.matchDiacritics && 'toolbar-item-active']" @click="toggleSearchOption('matchDiacritics')">匹配变音符号</div>
            <div class="pdf-search-option ml-[1px]" :class="[searchOptions.entireWord && 'toolbar-item-active']" @click="toggleSearchOption('entireWord')">全词匹配</div>
          </div>
          <div class="text-[12px] text-slate-500 px-[5px] mt-[3px]" v-if="searchTotal > 0">
            第{{ searchIndex }}项，共{{ searchTotal }}项
          </div>
        </div>
      </Transition>
    </section>
  </section>
</template>

<script>
import PDFTree from './PDFTree.vue';
import { PDF } from '../core/pdf';
import ResizeObserver from 'resize-observer-polyfill';
import { computePosition, flip, shift, size } from "@floating-ui/dom";
import { debounce } from '../core';
export default {
  name: "pdf-viewer-vue2",
  components: {
    PDFTree
  },
  props: {},
  data() {
    return {
      renderKey: 0,
      showThumbnail: false,
      currentPage: 1,
      totalPage: 0,
      loadingPercent: 0,
      loadingPercentVisible: false,
      smallMenu: false,
      showSmallMenu: false,
      spreadMode: 0,
      pageScale: 'page-actual',
      catalogTreeData: [],
      showCatalog: false,
      progress: 0,
      showPrint: false,
      searchKey: "",
      showSearch: false,
      searchIndex: 0,
      searchTotal: 0,
      searchOptions: {
        highlightAll: false, // 所有找到的都渲染
        matchDiacritics: false, // 匹配变音符号
        caseSensitive: false, // 区分大小写
        entireWord: false // 全词匹配
      },
      searchFloatXY: {
        x: 0,
        y: 0
      },
      menuFloatXY: {
        x: 0,
        y: 0
      },
      rotation: 0,
      scrollMode: 0,
      currentScale: 100
    }
  },
  methods: {
    inputScaleHandler(e) {
      let value = Number(e.target.value);
      this.changePageScale(value / 100)
    },
    changeScrollMode() {
      this.scrollMode = this.scrollMode === 0 ? 1 : 0
      if (this.pdfInstance) {
        this.pdfInstance.viewer.scrollMode = this.scrollMode
      }
    },
    changeRotation() {
      this.rotation = (this.rotation + 90) % 360
      if (this.pdfInstance) {
        this.pdfInstance.viewer.pagesRotation = this.rotation
      }
    },
    pagePressHandler(e) {
      let value = Number(e.target.value);
      value = Math.min(Math.max(0, value), this.totalPage);
      this.currentPage = value;
      if (this.pdfInstance) {
        this.pdfInstance.viewer.currentPageNumber = value;
      }
    },
    changeSpreadMode(value) {
      if (this.pdfInstance) {
        this.spreadMode = value
        this.pdfInstance.viewer.spreadMode = value;
      }
    },
    changePageScale(value) {
      if (this.pdfInstance) {
        this.pageScale = value
        this.pdfInstance.viewer.currentScaleValue = value;
        this.sideEffectFn?.()
        if (value === 'page-width' || value === 'page-height') {
          this.sideEffectFn = this.bindSize(this.$refs.pdfContainer, () => {
            this.pdfInstance.viewer.currentScaleValue = value;
          })
        }
      }
    },
    bindSize(dom, callback) {
      const ro = new ResizeObserver(() => {
        callback?.()
      });
      
      ro.observe(dom);
      return () => {
        ro.unobserve(dom)
      }
    },
    loadFile(data) {
      const config = {}
      if (typeof data == 'string') {
        config.url = data
      } else {
        config.data = data
      }
      this.renderKey++;
      this.showThumbnail = false;
      this.showCatalog = false;
      this.catalogTreeData = [];
      this.showSearch = false;
      this.loadingPercent = 0;
      this.rotation = 0;
      this.scrollMode = 0;
      this.currentScale = 100;
      this.resetSearch();
      this.$nextTick(() => {
        this.pdfInstance = new PDF({
          pdfContainer: this.$refs.pdfContainer,
          thumbnailContainer: this.$refs.thumbnailContainer,
          listeners: {
            onPagesLoaded: (v) => {
              this.currentPage = 1;
              this.totalPage = v.pagesCount;
              this.$emit("pagesLoaded", v);
            },
            onPageRendered: (v) => {
              this.$emit("pageRendered", v);
            },
            onPageChanging: (v) => {
              this.currentPage = v.pageNumber;
              this.$emit("pageChanging", v);
            },
            onFindChange: (v) => {
              this.searchIndex = v.matchesCount.current;
              this.searchTotal = v.matchesCount.total;
              this.$emit("findChange", v)
            },
            onLoadProgress: (v) => {
              this.loadingPercentVisible = v < 100
              this.loadingPercent = v
            },
            onScaleChanging: (v) => {
              console.log("scaleChanging", v)
              this.currentScale = parseInt(v.scale * 100)
              this.$emit("scaleChanging", v)
            }
          }
        });
        this.pdfInstance.loadFile(config);
      })
    },
    clickCatalog(node) {
      const dest = node.dest
      this.pdfInstance?.link?.goToDestination(dest)
    },
    async print() {
      if (!this.pdfInstance) return
      this.showPrint = true;
      this.progress = 0;
      const { abort } = await this.pdfInstance?.printPDF(
        (v) => this.progress = v,
        () => this.showPrint = false
      );
      this.abortPrint = abort;
    },
    abort() {
      this.abortPrint?.();
    },
    toggleSearchOption(key) {
      this.searchOptions[key] = !this.searchOptions[key]
      if (!this.searchKey) return
      if (key !== 'highlightAll') {
        this.searchTotal = 0
        this.searchIndex = 0
      }
      this.input()
    },
    _search() {
      this.pdfInstance?.find(this.searchKey, this.searchOptions)
    },
    findPrev() {
      this.pdfInstance?.findPrev(this.searchKey, this.searchOptions)
    },
    findNext() {
      this.pdfInstance?.findNext(this.searchKey, this.searchOptions)
    },
    resetSearch() {
      this.searchKey = "";
      this.searchIndex = 0;
      this.searchTotal = 0;
    },
    toggleSearch() {
      this.showSearch = !this.showSearch;
      if (this.showSearch) {
        this.resetSearch();
        this.$nextTick(() => {
          computePosition(this.$refs.reference, this.$refs.floating, {
            placement: "bottom",
            middleware: [flip(), shift()]
          }).then(({ x, y, placement }) => {
            if (placement.includes("bottom")) {
              this.searchFloatXY.x = 120
              this.searchFloatXY.y = -10
            } else if (placement.includes("top")) {
              this.searchFloatXY.x = 120
              this.searchFloatXY.y = this.$refs.floating.offsetHeight + 10
            }
            Object.assign(this.$refs.floating.style, {
              top: `${y}px`,
              left: `${x - 3}px`
            });
          });
        })
      } else {
        this.resetSearch();
        this.input();
      }
    },
    toggleMenu() {
      this.showSmallMenu = !this.showSmallMenu;
      if (this.showSmallMenu) {
        this.$nextTick(() => {
          computePosition(this.$refs.menuReference, this.$refs.menuFloating, {
            placement: "bottom",
            middleware: [flip(), shift(), size({
              apply({availableHeight, elements}) {
                Object.assign(elements.floating.style, {
                  maxHeight: `${Math.max(100, availableHeight - 15)}px`,
                  overflowY: 'auto'
                });
              },
            })]
          }).then(({ x, y, placement }) => {
            if (placement.includes("bottom")) {
              this.menuFloatXY.x = x / 2
              this.menuFloatXY.y = 10
            } else if (placement.includes("top")) {
              this.menuFloatXY.x = x / 2
              this.menuFloatXY.y = this.$refs.menuFloating.offsetHeight + 10
            }
            Object.assign(this.$refs.menuFloating.style, {
              top: `${y + 10}px`,
              left: `${x - 3}px`
            });
          });
        })
      }
    },
    destroy() {
      if (this.pdfInstance) this.pdfInstance.pdf?.destroy?.()
    }
  },
  watch: {
    renderKey: {
      handler: function() {
        this.$nextTick(() => {
          const dom = this.$refs.pdfWrapper
          if (dom) {
            this.bindSize(dom, () => {
              const isSmallMenu = dom.clientWidth < 700
              this.showSmallMenu = !isSmallMenu
              setTimeout(() => {
                this.smallMenu = isSmallMenu
              }, 200)
            })
          }
        })
      },
      immediate: true
    },
    showThumbnail(newValue) {
      if (newValue) {
        this.pdfInstance?.initThumbnailViewer();
        setTimeout(() => {
          this.pdfInstance?.thumbViewer?.scrollThumbnailIntoView(this.currentPage);
        })
        this.showCatalog = false
      }
    },
    async showCatalog(newValue) {
      if (newValue) {
        const outline = await this.pdfInstance?.pdf?.getOutline()
        this.catalogTreeData = outline
        this.showThumbnail = false
      }
    }
  },
  mounted() {
    this.input = debounce(this._search, 200)
  },
  beforeDestroy() {
    this.destroy()
  }
}
</script>

<style>
.transform-enter-active,
.transform-leave-active {
  transform: translateX(-200px);
}

.transform-enter-from,
.transform-leave-to {
  
}

.float-fade-enter-active,
.float-fade-leave-active {
  opacity: 0;
  transform: scale(0);
}

.float-fade-enter-from,
.float-fade-leave-to {
  opacity: 0;
  transform: scale(0);
}

.float-fade-enter-to,
.float-fade-leave-from {
  opacity: 1;
  transform: scale(1);
}

.pdf-rotate-90 {
  transform: rotate(-90deg);
}

.search-float {
  box-shadow: 0 0 3px 3px rgba(0, 0, 0, 0.06);
}
.search-float input {
  border: 1px solid #e5e7eb;
}
.pdf-small-menu {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
.pdf-menu-setting { 
  position: absolute; 
  right: 0; 
  width: 30px;
  height: 30px;
  line-height: 30px;
  margin-top: 10px;
}
 
.pdf-menu-setting .menu-group-setting { 
  width: 16px; 
  height: 17px; 
  position: relative;
  top: -1px;
}
.pdf-menu-setting .menu-group-setting:before, .pdf-menu-setting .menu-group-setting:after { 
  content: ""; 
  display: block; 
  width: 16px; 
  height: 2px; 
  background: #000; 
  border-radius: 2px; 
  position: absolute; 
  left: 0; 
  -webkit-transition: all 0.35s ease-in-out; 
  transition: all 0.35s ease-in-out;
}
.pdf-menu-setting .menu-group-setting:before { 
  top: 5px; 
  box-shadow: 0 10px #000;
}
.pdf-menu-setting .menu-group-setting:after { 
  bottom: 5px; 
}


.pdf-menu-setting .menu-group-setting-active:before { 
  top: 10px; 
  box-shadow: none; 
  -webkit-transform: rotate(225deg); 
  transform: rotate(225deg); 
}
.pdf-menu-setting .menu-group-setting-active:after { 
  bottom: 5px; 
  -webkit-transform: rotate(135deg); 
  transform: rotate(135deg); 
}
</style>
