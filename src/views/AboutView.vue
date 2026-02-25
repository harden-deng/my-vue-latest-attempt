<template>
  <div id="app">
    <!-- 菜单栏 -->
    <div class="menu" editor-component="menu">
      <div class="menu-item">
        <div class="menu-item__undo" :title="`撤销(${isApple ? '⌘' : 'Ctrl'}+Z)`" @click="handleUndo">
          <i></i>
        </div>
        <div class="menu-item__redo" :title="`重做(${isApple ? '⌘' : 'Ctrl'}+Y)`" @click="handleRedo">
          <i></i>
        </div>
        <div class="menu-item__painter" title="格式刷(双击可连续使用)" @click="handlePainter" @dblclick="handlePainterDouble">
          <i></i>
        </div>
        <div class="menu-item__format" title="清除格式" @click="handleFormat">
          <i></i>
        </div>
      </div>
      <div class="menu-divider"></div>
      <div class="menu-item">
        <div class="menu-item__font" @click="toggleOptions('font')">
          <span class="select" title="字体">{{ currentFont }}</span>
          <div class="options" :class="{ visible: visibleOptions.font }" @click.stop>
            <ul>
              <li v-for="font in fonts" :key="font.family" :data-family="font.family"
                :style="{ fontFamily: font.family }" :class="{ active: currentFont === font.name }"
                @click="handleFont(font.family)">
                {{ font.name }}
              </li>
            </ul>
          </div>
        </div>
        <div class="menu-item__size" @click="toggleOptions('size')">
          <span class="select" title="字号">{{ currentSize }}</span>
          <div class="options" :class="{ visible: visibleOptions.size }" @click.stop>
            <ul>
              <li v-for="size in sizes" :key="size.size" :data-size="size.size"
                :class="{ active: currentSize === size.name }" @click="handleSize(size.size)">
                {{ size.name }}
              </li>
            </ul>
          </div>
        </div>
        <div class="menu-item__size-add" :title="`增大字号(${isApple ? '⌘' : 'Ctrl'}+[)`" @click="handleSizeAdd">
          <i></i>
        </div>
        <div class="menu-item__size-minus" :title="`减小字号(${isApple ? '⌘' : 'Ctrl'}+])`" @click="handleSizeMinus">
          <i></i>
        </div>
        <div class="menu-item__bold" :title="`加粗(${isApple ? '⌘' : 'Ctrl'}+B)`" :class="{ active: isBold }"
          @click="handleBold">
          <i></i>
        </div>
        <div class="menu-item__italic" :title="`斜体(${isApple ? '⌘' : 'Ctrl'}+I)`" :class="{ active: isItalic }"
          @click="handleItalic">
          <i></i>
        </div>
        <div class="menu-item__underline" :title="`下划线(${isApple ? '⌘' : 'Ctrl'}+U)`" :class="{ active: isUnderline }">
          <i @click="handleUnderline"></i>
          <span class="select" @click.stop="toggleOptions('underline')"></span>
          <div class="options" :class="{ visible: visibleOptions.underline }" @click.stop>
            <ul>
              <li v-for="style in underlineStyles" :key="style" :data-decoration-style="style"
                @mousedown="handleUnderlineStyle(style)">
                <i></i>
              </li>
            </ul>
          </div>
        </div>
        <div class="menu-item__strikeout" title="删除线(Ctrl+Shift+X)" :class="{ active: isStrikeout }"
          @click="handleStrikeout">
          <i></i>
        </div>
        <div class="menu-item__superscript" :title="`上标(${isApple ? '⌘' : 'Ctrl'}+Shift+,)`"
          :class="{ active: isSuperscript }" @click="handleSuperscript">
          <i></i>
        </div>
        <div class="menu-item__subscript" :title="`下标(${isApple ? '⌘' : 'Ctrl'}+Shift+.)`"
          :class="{ active: isSubscript }" @click="handleSubscript">
          <i></i>
        </div>
        <div class="menu-item__color" title="字体颜色" :class="{ active: hasColor }" @click="handleColorClick">
          <i></i>
          <span :style="{ backgroundColor: colorValue }"></span>
          <input type="color" id="color" ref="colorInput" @input="handleColorChange" />
        </div>
        <div class="menu-item__highlight" title="高亮" :class="{ active: hasHighlight }" @click="handleHighlightClick">
          <i></i>
          <span :style="{ backgroundColor: highlightValue }"></span>
          <input type="color" id="highlight" ref="highlightInput" @input="handleHighlightChange" />
        </div>
      </div>
      <div class="menu-divider"></div>
      <div class="menu-item">
        <div class="menu-item__title" @click="toggleOptions('title')">
          <i></i>
          <span class="select" title="切换标题">{{ currentTitle }}</span>
          <div class="options" :class="{ visible: visibleOptions.title }" @click.stop>
            <ul>
              <li v-for="(title, index) in titles" :key="index" :data-level="title.level || null"
                :style="{ fontSize: title.size + 'px' }" :class="{ active: currentTitle === title.name }"
                @click="handleTitle(title.level)">
                {{ title.name }}
              </li>
            </ul>
          </div>
        </div>
        <div class="menu-item__left" :title="`左对齐(${isApple ? '⌘' : 'Ctrl'}+L)`" :class="{ active: rowFlex === 'left' }"
          @click="handleRowFlex('left')">
          <i></i>
        </div>
        <div class="menu-item__center" :title="`居中对齐(${isApple ? '⌘' : 'Ctrl'}+E)`"
          :class="{ active: rowFlex === 'center' }" @click="handleRowFlex('center')">
          <i></i>
        </div>
        <div class="menu-item__right" :title="`右对齐(${isApple ? '⌘' : 'Ctrl'}+R)`"
          :class="{ active: rowFlex === 'right' }" @click="handleRowFlex('right')">
          <i></i>
        </div>
        <div class="menu-item__alignment" :title="`两端对齐(${isApple ? '⌘' : 'Ctrl'}+J)`"
          :class="{ active: rowFlex === 'alignment' }" @click="handleRowFlex('alignment')">
          <i></i>
        </div>
        <div class="menu-item__justify" :title="`分散对齐(${isApple ? '⌘' : 'Ctrl'}+Shift+J)`"
          :class="{ active: rowFlex === 'justify' }" @click="handleRowFlex('justify')">
          <i></i>
        </div>
        <div class="menu-item__row-margin" @click="toggleOptions('rowMargin')">
          <i title="行间距"></i>
          <div class="options" :class="{ visible: visibleOptions.rowMargin }" @click.stop>
            <ul>
              <li v-for="margin in rowMargins" :key="margin" :data-rowmargin="margin"
                :class="{ active: currentRowMargin === margin }" @click="handleRowMargin(margin)">
                {{ margin }}
              </li>
            </ul>
          </div>
        </div>
        <div class="menu-item__list" :title="`列表(${isApple ? '⌘' : 'Ctrl'}+Shift+U)`" :class="{ active: hasList }"
          @click="toggleOptions('list')">
          <i></i>
          <div class="options" :class="{ visible: visibleOptions.list }" @click.stop>
            <ul>
              <li @click="handleList(null, null)">
                <label>取消列表</label>
              </li>
              <li v-for="list in listTypes" :key="list.type + list.style" :data-list-type="list.type"
                :data-list-style="list.style" @click="handleList(list.type, list.style)">
                <label>{{ list.label }}：</label>
                <ol v-if="list.type === 'ol'">
                  <li>________</li>
                </ol>
                <ul v-else :style="{ listStyleType: list.listStyleType }">
                  <li>________</li>
                </ul>
              </li>
            </ul>
          </div>
        </div>
      </div>
      <div class="menu-divider"></div>
      <div class="menu-item">
        <div class="menu-item__table" @click="showTablePanel = !showTablePanel">
          <i title="表格"></i>
        </div>
        <div class="menu-item__table__collapse" v-show="showTablePanel">
          <div class="table-close" @click="showTablePanel = false">×</div>
          <div class="table-title">
            <span class="table-select">{{ tableTitle }}</span>
            <span>表格</span>
          </div>
          <div class="table-panel" @mousemove="handleTableHover" @click="handleTableInsert"></div>
        </div>
        <div class="menu-item__image" @click="handleImageClick">
          <i title="图片"></i>
          <input type="file" id="image" ref="imageInput" accept=".png, .jpg, .jpeg, .svg, .gif"
            @change="handleImageChange" />
        </div>
        <div class="menu-item__hyperlink" title="超链接" @click="handleHyperlink">
          <i></i>
        </div>
        <div class="menu-item__separator" :class="{ active: hasSeparator }" @click="toggleOptions('separator')">
          <i title="分割线"></i>
          <div class="options" :class="{ visible: visibleOptions.separator }" @click.stop>
            <ul>
              <li v-for="sep in separators" :key="sep.value" :data-separator="sep.value"
                @mousedown="handleSeparator(sep.dash)">
                <i></i>
              </li>
            </ul>
          </div>
        </div>
        <div class="menu-item__watermark" @click="toggleOptions('watermark')">
          <i title="水印(添加、删除)"></i>
          <div class="options" :class="{ visible: visibleOptions.watermark }" @click.stop>
            <ul>
              <li data-menu="add" @mousedown="handleWatermark('add')">添加水印</li>
              <li data-menu="delete" @mousedown="handleWatermark('delete')">删除水印</li>
            </ul>
          </div>
        </div>
        <div class="menu-item__codeblock" title="代码块" @click="handleCodeBlock">
          <i></i>
        </div>
        <div class="menu-item__page-break" title="分页符" @click="handlePageBreak">
          <i></i>
        </div>
        <div class="menu-item__control" @click="toggleOptions('control')">
          <i title="控件"></i>
          <div class="options" :class="{ visible: visibleOptions.control }" @click.stop>
            <ul>
              <li v-for="control in controls" :key="control.type" :data-control="control.type"
                @click="handleControl(control.type)">
                {{ control.name }}
              </li>
            </ul>
          </div>
        </div>
        <div class="menu-item__checkbox" title="复选框" @click="handleCheckbox">
          <i></i>
        </div>
        <div class="menu-item__radio" title="单选框" @click="handleRadio">
          <i></i>
        </div>
        <div class="menu-item__latex" title="LateX" @click="handleLatex">
          <i></i>
        </div>
        <div class="menu-item__date" @click="toggleOptions('date')">
          <i title="日期"></i>
          <div class="options" :class="{ visible: visibleOptions.date }" @click.stop>
            <ul>
              <li v-for="format in dateFormats" :key="format.format" :data-format="format.format"
                @mousedown="handleDate(format.format, format.value)">
                {{ format.value }}
              </li>
            </ul>
          </div>
        </div>
        <div class="menu-item__block" title="内容块" @click="handleBlock">
          <i></i>
        </div>
      </div>
      <div class="menu-divider"></div>
      <div class="menu-item">
        <div class="menu-item__search" data-menu="search" @click="showSearchPanel = !showSearchPanel">
          <i></i>
        </div>
        <div class="menu-item__search__collapse" v-show="showSearchPanel">
          <div class="menu-item__search__collapse__search">
            <input type="text" v-model="searchText" @input="handleSearch" @keydown.enter="handleSearch" />
            <label class="search-result">{{ searchResult }}</label>
            <div class="arrow-left" @click="handleSearchPrev">
              <i></i>
            </div>
            <div class="arrow-right" @click="handleSearchNext">
              <i></i>
            </div>
            <span @click="handleSearchClose">×</span>
          </div>
          <div class="menu-item__search__collapse__replace">
            <input type="text" v-model="replaceText" />
            <button @click="handleReplace">替换</button>
          </div>
        </div>
        <div class="menu-item__print" :title="`打印(${isApple ? '⌘' : 'Ctrl'}+P)`" data-menu="print" @click="handlePrint">
          <i></i>
        </div>
      </div>
    </div>

    <!-- 目录 -->
    <div class="catalog" editor-component="catalog" v-show="showCatalog">
      <div class="catalog__header">
        <span>目录</span>
        <div class="catalog__header__close" @click="showCatalog = false">
          <i></i>
        </div>
      </div>
      <div class="catalog__main" ref="catalogMain"></div>
    </div>

    <!-- 编辑器容器 -->
    <div class="editor" ref="editorContainer"></div>

    <!-- 批注 -->
    <div class="comment" editor-component="comment"></div>

    <!-- 页脚 -->
    <div class="footer" editor-component="footer">
      <div>
        <div class="catalog-mode" title="目录" @click="showCatalog = !showCatalog">
          <i></i>
        </div>
        <div class="page-mode" @click="toggleOptions('pageMode')">
          <i title="页面模式(分页、连页)"></i>
          <div class="options" :class="{ visible: visibleOptions.pageMode }" @click.stop>
            <ul>
              <li data-page-mode="paging" :class="{ active: pageMode === 'paging' }" @click="handlePageMode('paging')">
                分页</li>
              <li data-page-mode="continuity" :class="{ active: pageMode === 'continuity' }"
                @click="handlePageMode('continuity')">连页</li>
            </ul>
          </div>
        </div>
        <span>可见页码：<span class="page-no-list">{{ visiblePageNos }}</span></span>
        <span>页面：<span class="page-no">{{ currentPageNo }}</span>/<span class="page-size">{{ totalPages }}</span></span>
        <span>字数：<span class="word-count">{{ wordCount }}</span></span>
        <span>行：<span class="row-no">{{ currentRow }}</span></span>
        <span>列：<span class="col-no">{{ currentCol }}</span></span>
      </div>
      <div class="editor-mode" :title="editorModeTitle" @click="handleModeChange">{{ currentModeName }}</div>
      <div>
        <div class="page-scale-minus" title="缩小(Ctrl+-)" @click="handleScaleMinus">
          <i></i>
        </div>
        <span class="page-scale-percentage" title="显示比例(点击可复原Ctrl+0)" @click="handleScaleRecovery">{{ scalePercentage
        }}%</span>
        <div class="page-scale-add" title="放大(Ctrl+=)" @click="handleScaleAdd">
          <i></i>
        </div>
        <div class="paper-size" @click="toggleOptions('paperSize')">
          <i title="纸张类型"></i>
          <div class="options" :class="{ visible: visibleOptions.paperSize }" @click.stop>
            <ul>
              <li v-for="paper in paperSizes" :key="paper.size" :data-paper-size="paper.size"
                :class="{ active: currentPaperSize === paper.size }" @click="handlePaperSize(paper.size)">
                {{ paper.name }}
              </li>
            </ul>
          </div>
        </div>
        <div class="paper-direction" @click="toggleOptions('paperDirection')">
          <i title="纸张方向"></i>
          <div class="options" :class="{ visible: visibleOptions.paperDirection }" @click.stop>
            <ul>
              <li data-paper-direction="vertical" :class="{ active: paperDirection === 'vertical' }"
                @click="handlePaperDirection('vertical')">纵向</li>
              <li data-paper-direction="horizontal" :class="{ active: paperDirection === 'horizontal' }"
                @click="handlePaperDirection('horizontal')">横向</li>
            </ul>
          </div>
        </div>
        <div class="paper-margin" title="页边距" @click="handlePaperMargin">
          <i></i>
        </div>
        <div class="fullscreen" :title="isFullscreen ? '退出全屏' : '全屏显示'" @click="handleFullscreen">
          <i></i>
        </div>
        <div class="editor-option" title="编辑器设置" @click="handleEditorOption">
          <i></i>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, nextTick, computed } from 'vue'
import Editor, { RowFlex, TitleLevel, ListType, ListStyle, PageMode, PaperDirection, EditorMode, ElementType, ControlType } from 'hardengzd-canvas-editor'

// 检测是否为 Mac
const isApple = typeof navigator !== 'undefined' && /Mac OS X/.test(navigator.userAgent)

// 编辑器实例
const editorInstance = ref(null)
const editorContainer = ref(null)
const catalogMain = ref(null)
const colorInput = ref(null)
const highlightInput = ref(null)
const imageInput = ref(null)

// 响应式数据
const visibleOptions = reactive({
  font: false,
  size: false,
  underline: false,
  title: false,
  rowMargin: false,
  list: false,
  separator: false,
  watermark: false,
  control: false,
  date: false,
  pageMode: false,
  paperSize: false,
  paperDirection: false
})

const showTablePanel = ref(false)
const showSearchPanel = ref(false)
const showCatalog = ref(true)

// 编辑器状态
const currentFont = ref('微软雅黑')
const currentSize = ref('小四')
const currentTitle = ref('正文')
const currentRowMargin = ref(1)
const isBold = ref(false)
const isItalic = ref(false)
const isUnderline = ref(false)
const isStrikeout = ref(false)
const isSuperscript = ref(false)
const isSubscript = ref(false)
const hasColor = ref(false)
const hasHighlight = ref(false)
const colorValue = ref('#000000')
const highlightValue = ref('#ffff00')
const rowFlex = ref('left')
const hasList = ref(false)
const hasSeparator = ref(false)

// 页面状态
const pageMode = ref('paging')
const paperDirection = ref('vertical')
const currentPaperSize = ref('794*1123')
const scalePercentage = ref(100)
const currentPageNo = ref(1)
const totalPages = ref(1)
const visiblePageNos = ref('1')
const wordCount = ref(0)
const currentRow = ref(0)
const currentCol = ref(0)
const isFullscreen = ref(false)

// 编辑器模式
const modeIndex = ref(0)
const modeList = [
  { mode: EditorMode.EDIT, name: '编辑模式' },
  { mode: EditorMode.CLEAN, name: '清洁模式' },
  { mode: EditorMode.READONLY, name: '只读模式' },
  { mode: EditorMode.FORM, name: '表单模式' },
  { mode: EditorMode.PRINT, name: '打印模式' },
  { mode: EditorMode.DESIGN, name: '设计模式' }
]

const currentModeName = computed(() => modeList[modeIndex.value].name)
const editorModeTitle = computed(() => '编辑模式(编辑、清洁、只读、表单、设计)')

// 表格相关
const tableTitle = ref('插入')
const tableRowIndex = ref(0)
const tableColIndex = ref(0)

// 搜索相关
const searchText = ref('')
const replaceText = ref('')
const searchResult = ref('')

// 数据配置
const fonts = [
  { family: 'Microsoft YaHei', name: '微软雅黑' },
  { family: '华文宋体', name: '华文宋体' },
  { family: '华文黑体', name: '华文黑体' },
  { family: '华文仿宋', name: '华文仿宋' },
  { family: '华文楷体', name: '华文楷体' },
  { family: 'Arial', name: 'Arial' },
  { family: 'Segoe UI', name: 'Segoe UI' }
]

const sizes = [
  { size: 56, name: '初号' },
  { size: 48, name: '小初' },
  { size: 34, name: '一号' },
  { size: 32, name: '小一' },
  { size: 29, name: '二号' },
  { size: 24, name: '小二' },
  { size: 21, name: '三号' },
  { size: 20, name: '小三' },
  { size: 18, name: '四号' },
  { size: 16, name: '小四' },
  { size: 14, name: '五号' },
  { size: 12, name: '小五' },
  { size: 10, name: '六号' },
  { size: 8, name: '小六' }
]

const titles = [
  { level: null, name: '正文', size: 16 },
  { level: TitleLevel.FIRST, name: '标题1', size: 26 },
  { level: TitleLevel.SECOND, name: '标题2', size: 24 },
  { level: TitleLevel.THIRD, name: '标题3', size: 22 },
  { level: TitleLevel.FOURTH, name: '标题4', size: 20 },
  { level: TitleLevel.FIFTH, name: '标题5', size: 18 },
  { level: TitleLevel.SIXTH, name: '标题6', size: 16 }
]

const rowMargins = [1, 1.25, 1.5, 1.75, 2, 2.5, 3]
const underlineStyles = ['solid', 'double', 'dashed', 'dotted', 'wavy']

const listTypes = [
  { type: ListType.OL, style: ListStyle.DECIMAL, label: '有序列表', listStyleType: 'decimal' },
  { type: ListType.UL, style: ListStyle.CHECKBOX, label: '复选框列表', listStyleType: '☑️ ' },
  { type: ListType.UL, style: ListStyle.DISC, label: '实心圆点列表', listStyleType: 'disc' },
  { type: ListType.UL, style: ListStyle.CIRCLE, label: '空心圆点列表', listStyleType: 'circle' },
  { type: ListType.UL, style: ListStyle.SQUARE, label: '空心方块列表', listStyleType: '☐ ' }
]

const separators = [
  { value: '0,0', dash: [] },
  { value: '1,1', dash: [1, 1] },
  { value: '3,1', dash: [3, 1] },
  { value: '4,4', dash: [4, 4] },
  { value: '7,3,3,3', dash: [7, 3, 3, 3] },
  { value: '6,2,2,2,2,2', dash: [6, 2, 2, 2, 2, 2] }
]

const controls = [
  { type: ControlType.TEXT, name: '文本' },
  { type: ControlType.NUMBER, name: '数值' },
  { type: ControlType.SELECT, name: '列举' },
  { type: ControlType.DATE, name: '日期' },
  { type: ControlType.CHECKBOX, name: '复选框' },
  { type: ControlType.RADIO, name: '单选框' }
]

const paperSizes = [
  { size: '794*1123', name: 'A4' },
  { size: '1593*2251', name: 'A2' },
  { size: '1125*1593', name: 'A3' },
  { size: '565*796', name: 'A5' },
  { size: '412*488', name: '5号信封' },
  { size: '450*866', name: '6号信封' },
  { size: '609*862', name: '7号信封' },
  { size: '862*1221', name: '9号信封' },
  { size: '813*1266', name: '法律用纸' },
  { size: '813*1054', name: '信纸' }
]

// 初始化表格面板
const initTablePanel = () => {
  const tablePanel = document.querySelector('.table-panel')
  if (!tablePanel) return

  tablePanel.innerHTML = ''
  for (let i = 0; i < 10; i++) {
    const tr = document.createElement('tr')
    tr.classList.add('table-row')
    for (let j = 0; j < 10; j++) {
      const td = document.createElement('td')
      td.classList.add('table-cel')
      tr.appendChild(td)
    }
    tablePanel.appendChild(tr)
  }
}

// 工具函数
const toggleOptions = (key) => {
  Object.keys(visibleOptions).forEach(k => {
    if (k !== key) visibleOptions[k] = false
  })
  visibleOptions[key] = !visibleOptions[key]
}

const closeAllOptions = () => {
  Object.keys(visibleOptions).forEach(k => {
    visibleOptions[k] = false
  })
}

// 菜单处理函数
const handleUndo = () => {
  editorInstance.value?.command.executeUndo()
}

const handleRedo = () => {
  editorInstance.value?.command.executeRedo()
}

let painterTimeout = null
let isFirstPainterClick = true

const handlePainter = () => {
  if (isFirstPainterClick) {
    isFirstPainterClick = false
    painterTimeout = setTimeout(() => {
      editorInstance.value?.command.executePainter({ isDblclick: false })
      isFirstPainterClick = true
    }, 200)
  } else {
    clearTimeout(painterTimeout)
  }
}

const handlePainterDouble = () => {
  isFirstPainterClick = true
  clearTimeout(painterTimeout)
  editorInstance.value?.command.executePainter({ isDblclick: true })
}

const handleFormat = () => {
  editorInstance.value?.command.executeFormat()
}

const handleFont = (family) => {
  editorInstance.value?.command.executeFont(family)
  closeAllOptions()
}

const handleSize = (size) => {
  editorInstance.value?.command.executeSize(size)
  closeAllOptions()
}

const handleSizeAdd = () => {
  editorInstance.value?.command.executeSizeAdd()
}

const handleSizeMinus = () => {
  editorInstance.value?.command.executeSizeMinus()
}

const handleBold = () => {
  editorInstance.value?.command.executeBold()
}

const handleItalic = () => {
  editorInstance.value?.command.executeItalic()
}

const handleUnderline = () => {
  editorInstance.value?.command.executeUnderline()
  closeAllOptions()
}

const handleUnderlineStyle = (style) => {
  editorInstance.value?.command.executeUnderline({ style })
  closeAllOptions()
}

const handleStrikeout = () => {
  editorInstance.value?.command.executeStrikeout()
}

const handleSuperscript = () => {
  editorInstance.value?.command.executeSuperscript()
}

const handleSubscript = () => {
  editorInstance.value?.command.executeSubscript()
}

const handleColorClick = () => {
  colorInput.value?.click()
}

const handleColorChange = (e) => {
  const color = e.target.value
  colorValue.value = color
  editorInstance.value?.command.executeColor(color)
}

const handleHighlightClick = () => {
  highlightInput.value?.click()
}

const handleHighlightChange = (e) => {
  const color = e.target.value
  highlightValue.value = color
  editorInstance.value?.command.executeHighlight(color)
}

const handleTitle = (level) => {
  editorInstance.value?.command.executeTitle(level || null)
  closeAllOptions()
}

const handleRowFlex = (flex) => {
  editorInstance.value?.command.executeRowFlex(flex)
  rowFlex.value = flex
}

const handleRowMargin = (margin) => {
  editorInstance.value?.command.executeRowMargin(margin)
  currentRowMargin.value = margin
  closeAllOptions()
}

const handleList = (type, style) => {
  editorInstance.value?.command.executeList(type, style)
  closeAllOptions()
}

const handleTableHover = (e) => {
  const celSize = 16
  const rowMarginTop = 10
  const celMarginRight = 6
  const { offsetX, offsetY } = e

  const cells = document.querySelectorAll('.table-cel')
  cells.forEach(cell => cell.classList.remove('active'))

  tableColIndex.value = Math.ceil(offsetX / (celSize + celMarginRight)) || 1
  tableRowIndex.value = Math.ceil(offsetY / (celSize + rowMarginTop)) || 1

  const rows = document.querySelectorAll('.table-row')
  rows.forEach((row, rowIdx) => {
    if (rowIdx < tableRowIndex.value) {
      const cols = row.querySelectorAll('.table-cel')
      cols.forEach((col, colIdx) => {
        if (colIdx < tableColIndex.value) {
          col.classList.add('active')
        }
      })
    }
  })

  tableTitle.value = `${tableRowIndex.value}×${tableColIndex.value}`
}

const handleTableInsert = () => {
  editorInstance.value?.command.executeInsertTable(tableRowIndex.value, tableColIndex.value)
  showTablePanel.value = false
  tableTitle.value = '插入'
  tableRowIndex.value = 0
  tableColIndex.value = 0
}

const handleImageClick = () => {
  imageInput.value?.click()
}

const handleImageChange = (e) => {
  const file = e.target.files?.[0]
  if (!file) return

  const reader = new FileReader()
  reader.readAsDataURL(file)
  reader.onload = () => {
    const image = new Image()
    image.src = reader.result
    image.onload = () => {
      editorInstance.value?.command.executeImage({
        value: reader.result,
        width: image.width,
        height: image.height
      })
      e.target.value = ''
    }
  }
}

const handleHyperlink = () => {
  const name = editorInstance.value?.command.getRangeText() || ''
  const url = prompt('请输入链接地址:', '')
  if (!url) return

  editorInstance.value?.command.executeHyperlink({
    url,
    valueList: name.split('').map(n => ({ value: n, size: 16 }))
  })
}

const handleSeparator = (dash) => {
  editorInstance.value?.command.executeSeparator(dash)
  closeAllOptions()
}

const handleWatermark = (action) => {
  if (action === 'add') {
    const data = prompt('请输入水印内容:', '')
    if (!data) return
    // 这里需要调用水印添加API，具体实现根据canvas-editor的API
  } else {
    // 删除水印
  }
  closeAllOptions()
}

const handleCodeBlock = () => {
  const code = prompt('请输入代码:', '')
  if (!code) return
  // 实现代码块插入
}

const handlePageBreak = () => {
  editorInstance.value?.command.executePageBreak()
}

const handleControl = (type) => {
  // 实现控件插入，需要根据类型显示不同的对话框
  console.log('Insert control:', type)
  closeAllOptions()
}

const handleCheckbox = () => {
  editorInstance.value?.command.executeInsertElementList([{
    type: ElementType.CHECKBOX,
    checkbox: { value: false },
    value: ''
  }])
}

const handleRadio = () => {
  editorInstance.value?.command.executeInsertElementList([{
    type: ElementType.RADIO,
    checkbox: { value: false },
    value: ''
  }])
}

const handleLatex = () => {
  const value = prompt('请输入LaTeX文本:', '')
  if (!value) return
  editorInstance.value?.command.executeInsertElementList([{
    type: ElementType.LATEX,
    value
  }])
}

const dateFormats = ref([
  { format: 'yyyy-MM-dd', value: '' },
  { format: 'yyyy-MM-dd hh:mm:ss', value: '' }
])

const updateDateFormats = () => {
  const date = new Date()
  const year = date.getFullYear().toString()
  const month = (date.getMonth() + 1).toString().padStart(2, '0')
  const day = date.getDate().toString().padStart(2, '0')
  const hour = date.getHours().toString().padStart(2, '0')
  const minute = date.getMinutes().toString().padStart(2, '0')
  const second = date.getSeconds().toString().padStart(2, '0')

  dateFormats.value[0].value = `${year}-${month}-${day}`
  dateFormats.value[1].value = `${year}-${month}-${day} ${hour}:${minute}:${second}`
}

const handleDate = (format, value) => {
  editorInstance.value?.command.executeInsertElementList([{
    type: ElementType.DATE,
    value: '',
    dateFormat: format,
    valueList: [{ value }]
  }])
  closeAllOptions()
}

const handleBlock = () => {
  // 实现内容块插入
  console.log('Insert block')
}

const handleSearch = () => {
  editorInstance.value?.command.executeSearch(searchText.value || null)
  updateSearchResult()
}

const handleSearchPrev = () => {
  editorInstance.value?.command.executeSearchNavigatePre()
  updateSearchResult()
}

const handleSearchNext = () => {
  editorInstance.value?.command.executeSearchNavigateNext()
  updateSearchResult()
}

const handleSearchClose = () => {
  showSearchPanel.value = false
  searchText.value = ''
  replaceText.value = ''
  editorInstance.value?.command.executeSearch(null)
  searchResult.value = ''
}

const updateSearchResult = () => {
  const result = editorInstance.value?.command.getSearchNavigateInfo()
  if (result) {
    searchResult.value = `${result.index}/${result.count}`
  } else {
    searchResult.value = ''
  }
}

const handleReplace = () => {
  if (searchText.value && searchText.value !== replaceText.value) {
    editorInstance.value?.command.executeReplace(replaceText.value)
  }
}

const handlePrint = () => {
  editorInstance.value?.command.executePrint()
}

const handlePageMode = (mode) => {
  editorInstance.value?.command.executePageMode(mode)
  pageMode.value = mode
  closeAllOptions()
}

const handleScaleMinus = () => {
  editorInstance.value?.command.executePageScaleMinus()
}

const handleScaleAdd = () => {
  editorInstance.value?.command.executePageScaleAdd()
}

const handleScaleRecovery = () => {
  editorInstance.value?.command.executePageScaleRecovery()
}

const handlePaperSize = (size) => {
  const [width, height] = size.split('*').map(Number)
  editorInstance.value?.command.executePaperSize(width, height)
  currentPaperSize.value = size
  closeAllOptions()
}

const handlePaperDirection = (direction) => {
  editorInstance.value?.command.executePaperDirection(direction)
  paperDirection.value = direction
  closeAllOptions()
}

const handlePaperMargin = () => {
  const margins = editorInstance.value?.command.getPaperMargin() || [100, 120, 100, 120]
  const top = prompt('上边距:', margins[0])
  const right = prompt('右边距:', margins[1])
  const bottom = prompt('下边距:', margins[2])
  const left = prompt('左边距:', margins[3])

  if (top && right && bottom && left) {
    editorInstance.value?.command.executeSetPaperMargin([
      Number(top),
      Number(right),
      Number(bottom),
      Number(left)
    ])
  }
}

const handleFullscreen = () => {
  if (!document.fullscreenElement) {
    document.documentElement.requestFullscreen()
  } else {
    document.exitFullscreen()
  }
}

const handleEditorOption = () => {
  const options = editorInstance.value?.command.getOptions()
  const newOptions = prompt('编辑器配置:', JSON.stringify(options, null, 2))
  if (newOptions) {
    try {
      editorInstance.value?.command.executeUpdateOptions(JSON.parse(newOptions))
    } catch (e) {
      console.error('Invalid JSON:', e)
    }
  }
}

const handleModeChange = () => {
  modeIndex.value = (modeIndex.value + 1) % modeList.length
  const { mode, name } = modeList[modeIndex.value]
  editorInstance.value?.command.executeMode(mode)
}

// 更新目录
const updateCatalog = async () => {
  if (!catalogMain.value) return
  const catalog = await editorInstance.value?.command.getCatalog()
  if (!catalog) {
    catalogMain.value.innerHTML = ''
    return
  }

  catalogMain.value.innerHTML = ''
  const appendCatalog = (parent, items) => {
    items.forEach(item => {
      const itemDom = document.createElement('div')
      itemDom.classList.add('catalog-item')

      const contentDom = document.createElement('div')
      contentDom.classList.add('catalog-item__content')
      const span = document.createElement('span')
      span.innerText = item.name
      contentDom.appendChild(span)
      contentDom.onclick = () => {
        editorInstance.value?.command.executeLocationCatalog(item.id)
      }

      itemDom.appendChild(contentDom)
      if (item.subCatalog && item.subCatalog.length) {
        appendCatalog(itemDom, item.subCatalog)
      }
      parent.appendChild(itemDom)
    })
  }

  appendCatalog(catalogMain.value, catalog)
}

// 初始化编辑器
onMounted(async () => {
  await nextTick()

  if (!editorContainer.value) return

  // 初始化表格面板
  initTablePanel()

  // 创建编辑器实例
  editorInstance.value = new Editor(
    editorContainer.value,
    {
      header: [{
        value: '第一人民医院',
        size: 32,
        rowFlex: RowFlex.CENTER
      }, {
        value: '\n门诊病历',
        size: 18,
        rowFlex: RowFlex.CENTER
      }],
      main: [{
        value: '欢迎使用 Canvas Editor！\n\n这是一个功能强大的富文本编辑器，支持多种文本格式、插入元素和丰富的编辑功能。'
      }],
      footer: [{
        value: 'canvas-editor',
        size: 12
      }]
    },
    {
      width: 794,
      height: 1123,
      scale: 1,
      pageGap: 20
    }
  )

  // 监听事件
  if (editorInstance.value?.listener) {
    editorInstance.value.listener.rangeStyleChange = (payload) => {
      // 更新样式状态
      isBold.value = payload.bold || false
      isItalic.value = payload.italic || false
      isUnderline.value = payload.underline || false
      isStrikeout.value = payload.strikeout || false
      isSuperscript.value = payload.type === ElementType.SUPERSCRIPT
      isSubscript.value = payload.type === ElementType.SUBSCRIPT
      hasColor.value = !!payload.color
      hasHighlight.value = !!payload.highlight
      if (payload.color) colorValue.value = payload.color
      if (payload.highlight) highlightValue.value = payload.highlight
      rowFlex.value = payload.rowFlex || 'left'
      hasList.value = !!payload.listType
      hasSeparator.value = payload.type === ElementType.SEPARATOR

      // 更新字体和字号
      if (payload.font) {
        const font = fonts.find(f => f.family === payload.font)
        if (font) currentFont.value = font.name
      }
      if (payload.size) {
        const size = sizes.find(s => s.size === payload.size)
        if (size) {
          currentSize.value = size.name
        } else {
          currentSize.value = `${payload.size}`
        }
      }

      // 更新标题
      if (payload.level) {
        const title = titles.find(t => t.level === payload.level)
        if (title) currentTitle.value = title.name
      } else {
        currentTitle.value = '正文'
      }

      // 更新行间距
      if (payload.rowMargin) {
        currentRowMargin.value = payload.rowMargin
      }
    }

    editorInstance.value.listener.visiblePageNoListChange = (payload) => {
      visiblePageNos.value = payload.map(i => i + 1).join('、')
    }

    editorInstance.value.listener.pageSizeChange = (payload) => {
      totalPages.value = payload
    }

    editorInstance.value.listener.intersectionPageNoChange = (payload) => {
      currentPageNo.value = payload + 1
    }

    editorInstance.value.listener.pageScaleChange = (payload) => {
      scalePercentage.value = Math.floor(payload * 10 * 10)
    }

    editorInstance.value.listener.pageModeChange = (payload) => {
      pageMode.value = payload
    }
  }

  // 点击外部关闭选项
  document.addEventListener('click', (e) => {
    const visibleDom = document.querySelector('.options.visible')
    if (visibleDom && !visibleDom.contains(e.target)) {
      closeAllOptions()
    }
  }, { capture: true })

  // 全屏监听
  document.addEventListener('fullscreenchange', () => {
    isFullscreen.value = !!document.fullscreenElement
  })

  // 更新日期格式
  updateDateFormats()

  // 更新目录
  updateCatalog()

  // 定时更新目录（可选）
  // setInterval(() => {
  //   updateCatalog()
  // }, 2000)
})
</script>

<style>
@import url('../style.css');

#app {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>