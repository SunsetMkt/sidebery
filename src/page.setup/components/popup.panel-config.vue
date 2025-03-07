<template lang="pug">
.ConfigPopup(ref="rootEl" tabindex="-1" @wheel="onWheel")
  h2(v-if="isTabsOrBookmarks") {{conf.name}}
  TextInput.title(
    v-if="!isTabsOrBookmarks"
    ref="nameInput"
    :line="true"
    :value="conf.name"
    :or="translate('panel.name_placeholder')"
    @update:value="onNameInput")

  SelectField(
    v-if="Utils.isTabsPanel(conf)"
    label="panel.icon_label"
    optLabel="settings.panel_icon_"
    :value="iconSVG"
    :opts="TABS_PANEL_ICON_OPTS"
    :color="color"
    @update:value="setIcon")
  SelectField(
    v-if="Utils.isBookmarksPanel(conf)"
    label="panel.icon_label"
    optLabel="settings.panel_icon_"
    :value="iconSVG"
    :opts="BOOKMARKS_PANEL_ICON_OPTS"
    :color="color"
    @update:value="setIcon")

  SelectField(
    v-if="!isTabsOrBookmarks"
    label="panel.color_label"
    :value="color"
    :opts="COLOR_OPTS"
    :icon="'#' + iconSVG"
    @update:value="setColor")

  .TextField.custom-icon(v-if="Utils.isTabsPanel(conf) || Utils.isBookmarksPanel(conf)")
    .body
      .label {{translate('panel.custom_icon')}}
      .btn(
        tabindex="-1"
        :data-active="state.customIconType === 'text'"
        @click="setCustomIconType('text')"
        @keydown.enter.space.prevent="setCustomIconType('text')") {{translate('panel.custom_icon_text_btn')}}
        .focus-fx
      .btn(
        tabindex="-1"
        :data-active="state.customIconType === 'url'"
        @click="setCustomIconType('url')"
        @keydown.enter.space.prevent="setCustomIconType('url')") {{translate('panel.custom_icon_url_btn')}}
        .focus-fx
      .btn(
        :data-active="state.customIconType === 'file'"
        @click="setCustomIconType('file')")
        .btn-label {{translate('panel.custom_icon_file_btn')}}
        .focus-fx
        input(
          tabindex="-1"
          type="file"
          accept="image/*"
          @input="openCustomIconFile"
          @keydown.enter.space="setCustomIconType('file')"
          @keyup="onFileInputKeyup")
      .img-box(v-if="state.customIconUrl")
        img(:src="state.customIconUrl" @load="onCustomIconLoad" @error="onCustomIconError")
      .img-rm(
        v-if="state.customIconUrl"
        tabindex="-1"
        @click="onCustomIconRm"
        @keydown.enter.space.prevent="onCustomIconRm")
        svg: use(xlink:href="#icon_remove")
        .focus-fx
    .body.-sub(v-if="state.customIconType === 'text'")
      TextInput(
        :value="state.customIconTextValue"
        :line="true"
        :or="translate('panel.custom_icon_text_placeholder')"
        @update:value="onCustomIconTextInput")
    .note.-sub(v-if="state.customIconType === 'text'") {{translate('panel.custom_icon_note')}}
    .body.-sub(v-if="state.customIconType === 'url'")
      TextInput(
        v-model:value="state.customIconUrlValue"
        :line="true"
        :or="translate('panel.custom_icon_url_placeholder')")
      .btn(
        tabindex="-1"
        @click="loadCustomIcon"
        @keydown.enter.space.prevent="loadCustomIcon") {{translate('panel.custom_icon_load')}}
        .focus-fx

  .sub-fields(v-if="Utils.isTabsPanel(conf) || Utils.isBookmarksPanel(conf)")
    ToggleField(
      label="panel.custom_icon_colorize"
      :inactive="!state.customIconOriginal"
      :value="state.customIconColorize"
      @update:value="toggleCustomIconColorize")

  ToggleField(
    label="panel.lock_panel_label"
    :value="conf.lockedPanel"
    @update:value="togglePanelLock")

  ToggleField(
    v-if="isNotTabsPanel(conf)"
    label="panel.temp_mode_label"
    :value="conf.tempMode"
    @update:value="toggleTempMode")

  ToggleField(
    label="panel.skip_on_switching"
    :value="conf.skipOnSwitching"
    @update:value="toggleSkipOnSwitching")

  ToggleField(
    v-if="Utils.isTabsPanel(conf)"
    label="panel.no_empty_label"
    :value="conf.noEmpty"
    @update:value="togglePanelNoEmpty")

  SelectField(
    v-if="Utils.isTabsPanel(conf)"
    label="panel.new_tab_ctx"
    :value="newTabCtx"
    :opts="containersOpts"
    :folded="true"
    @update:value="togglePanelNewTabCtx")
  .sub-fields
    ToggleField(
      v-if="Utils.isTabsPanel(conf)"
      label="panel.new_tab_ctx_reopen"
      :inactive="conf.newTabCtx === 'none'"
      :value="Settings.state.newTabCtxReopen"
      @update:value="toggleNewTabCtxReopen")

  SelectField(
    v-if="Utils.isTabsPanel(conf)"
    label="panel.drop_tab_ctx"
    :value="dropTabCtx"
    :opts="availableForAutoMoveContainersOpts"
    :folded="true"
    @update:value="togglePanelDropTabCtx")

  SelectField(
    v-if="Utils.isBookmarksPanel(conf)"
    label="panel.bookmarks_view_mode"
    optLabel="panel.bookmarks_view_mode_"
    :value="conf.viewMode"
    :opts="['tree', 'history']"
    @update:value="selectBookmarksViewMode")

  ToggleField(
    v-if="Utils.isBookmarksPanel(conf)"
    label="panel.auto_convert"
    :inactive="rootDirIsFF"
    :value="conf.autoConvert"
    @update:value="toggleAutoConvert")

  .InfoField.-folder(v-if="isBookmarks")
    .label {{translate('panel.root_id_label')}}
    .value {{rootPath}}
    .ctrls
      .btn(
        tabindex="-1"
        @click="setBookmarksRootId"
        @keydown.enter.space="setBookmarksRootId") {{translate('panel.root_id.choose')}}
        .focus-fx
      .btn(
        tabindex="-1"
        @click="resetBookmarksRootId"
        @keydown.enter.space="resetBookmarksRootId") {{translate('panel.root_id.reset')}}
        .focus-fx

  .InfoField(v-if="Utils.isTabsPanel(conf)")
    .label {{translate('panel.tab_move_rules')}}
    .btn(
      tabindex="-1"
      @click="openRulesPopup"
      @keydown.enter.space="openRulesPopup") {{getManageRulesBtnLabel(conf)}}
      .focus-fx
  .sub-fields(v-if="Utils.isTabsPanel(conf)")
    SelectField(
      label="panel.move_excluded_to"
      :inactive="!conf.moveRules.length"
      :value="conf.moveExcludedTo"
      :opts="availableForMoveExcludePanelsOpts"
      :folded="true"
      @update:value="toggleMoveExclude")

  .InfoField(v-if="Utils.isTabsPanel(conf)")
    .label {{translate('panel.new_tab_shortcuts')}}
    .btn(
      tabindex="-1"
      @click="openShortcutsPopup"
      @keydown.enter.space="openShortcutsPopup") {{getManageShortcutsBtnLabel(conf)}}
      .focus-fx
</template>

<script lang="ts" setup>
import { ref, reactive, computed, onMounted, nextTick } from 'vue'
import * as Utils from 'src/utils'
import * as Logs from 'src/services/logs'
import * as Popups from 'src/services/popups'
import { translate } from 'src/dict'
import { BKM_MENU_ID, FOLDER_NAME_DATA_RE, NOID } from 'src/defaults'
import { DEFAULT_CONTAINER_ID, COLOR_OPTS, PANEL_ICON_OPTS } from 'src/defaults'
import { BKM_ROOT_ID, RGB_COLORS } from 'src/defaults'
import { TextInputComponent, PanelConfig, BookmarksPanelConfig, TabsPanelConfig } from 'src/types'
import { HistoryPanelConfig } from 'src/types'
import { Settings } from 'src/services/settings'
import { Containers } from 'src/services/containers'
import { Permissions } from 'src/services/permissions'
import { Bookmarks } from 'src/services/bookmarks'
import * as Favicons from 'src/services/favicons.fg'
import { Styles } from 'src/services/styles'
import { SidebarConfigRState, saveSidebarConfig } from 'src/services/sidebar-config'
import TextInput from '../../components/text-input.vue'
import ToggleField from '../../components/toggle-field.vue'
import SelectField from '../../components/select-field.vue'

interface ContainerOption {
  value: string
  icon?: string
  color?: string
  tooltip?: string
  title?: string
}

interface PanelOption {
  value: ID
  icon?: string
  color?: string
  tooltip?: string
  title?: string
}

const URL_RE = /^https?:\/\/.+/
const TABS_PANEL_ICON_OPTS = [{ value: 'icon_tabs', icon: '#icon_tabs' }, ...PANEL_ICON_OPTS]
const BOOKMARKS_PANEL_ICON_OPTS = [
  { value: 'icon_bookmarks', icon: '#icon_bookmarks' },
  ...PANEL_ICON_OPTS,
]
const defaultContainerTooltip = translate('panel.ctr_tooltip_default')
const noneContainerTooltip = translate('panel.ctr_tooltip_none')
const nonePanelTooltip = translate('panel.panel_tooltip_none')

const state = reactive({
  customIconUrl: '',
  customIconType: '',
  customIconTextValue: '',
  customIconUrlValue: '',
  customIconOriginal: '',
  customIconColorize: false,
})

let updCustomIconTimeout: number

const rootEl = ref<HTMLElement | null>(null)
const nameInput = ref<TextInputComponent | null>(null)

const props = defineProps<{ conf: PanelConfig }>()

const isBookmarks = computed<boolean>(() => Utils.isBookmarksPanel(props.conf))
const isTabs = computed<boolean>(() => Utils.isTabsPanel(props.conf))
const isTabsOrBookmarks = computed<boolean>(() => !isTabs.value && !isBookmarks.value)
const containersOpts = computed<ContainerOption[]>(() => {
  const result: ContainerOption[] = []

  for (let c of Object.values(Containers.reactive.byId)) {
    result.push({ value: c.id, color: c.color, icon: '#' + c.icon, title: c.name, tooltip: c.name })
  }

  result.push({
    value: 'none',
    color: 'inactive',
    icon: '#icon_none',
    title: noneContainerTooltip,
    tooltip: noneContainerTooltip,
  })

  return result
})
const availableForAutoMoveContainersOpts = computed<ContainerOption[]>(() => {
  const result: ContainerOption[] = []
  const used: Record<string, boolean> = {}

  for (const id of SidebarConfigRState.nav) {
    const p = SidebarConfigRState.panels[id]
    if (!Utils.isTabsPanel(p) || p.id === props.conf.id) continue
    for (const ruleConf of p.moveRules) {
      if (ruleConf.containerId && !ruleConf.url) used[ruleConf.containerId] = true
    }
  }

  if (!used[DEFAULT_CONTAINER_ID]) {
    result.push({
      value: DEFAULT_CONTAINER_ID,
      color: 'toolbar',
      icon: '#icon_ff',
      title: defaultContainerTooltip,
      tooltip: defaultContainerTooltip,
    })
  }

  for (let c of Object.values(Containers.reactive.byId)) {
    if (!used[c.id]) {
      const icon = '#' + c.icon
      result.push({ value: c.id, color: c.color, icon, title: c.name, tooltip: c.name })
    }
  }

  result.push({
    value: 'none',
    color: 'inactive',
    icon: '#icon_none',
    title: noneContainerTooltip,
    tooltip: noneContainerTooltip,
  })

  return result
})
const availableForMoveExcludePanelsOpts = computed<PanelOption[]>(() => {
  const result: PanelOption[] = []

  for (const id of SidebarConfigRState.nav) {
    const p = SidebarConfigRState.panels[id]
    if (!Utils.isTabsPanel(p)) continue
    if (p.id === props.conf.id) continue
    const icon = p.iconIMG ? p.iconIMG : '#' + p.iconSVG
    result.push({ value: p.id, color: p.color, icon, title: p.name })
  }

  result.push({
    value: NOID,
    color: 'inactive',
    icon: '#icon_none',
    title: nonePanelTooltip,
    tooltip: nonePanelTooltip,
  })

  return result
})
const iconSVG = computed<string>(() => props.conf.iconSVG ?? '')
const color = computed<string>(() => props.conf.color || '')
const newTabCtx = computed<string>(() => {
  return (Utils.isTabsPanel(props.conf) && props.conf.newTabCtx) || ''
})
const dropTabCtx = computed<string>(() => {
  return (Utils.isTabsPanel(props.conf) && props.conf.dropTabCtx) || ''
})
const rootPath = computed<string>(() => {
  if (!Utils.isBookmarksPanel(props.conf)) return ''
  if (!props.conf.rootId) return '/'
  if (props.conf.rootId === BKM_ROOT_ID) return '/'

  let parent = Bookmarks.reactive.byId[props.conf.rootId]
  if (!parent) return translate('panel.root_id_wrong')

  const path: string[] = []
  while (parent) {
    const titleExec = FOLDER_NAME_DATA_RE.exec(parent.title)
    if (titleExec) path.push(titleExec[1])
    else path.push(parent.title)
    parent = Bookmarks.reactive.byId[parent.parentId]
  }

  return '/' + path.reverse().join('/')
})
const rootDirIsFF = computed<boolean>(() => {
  if (!Utils.isBookmarksPanel(props.conf)) return false
  return (props.conf.rootId as string).endsWith('___')
})

onMounted(() => {
  init()

  if (!Bookmarks.reactive.tree.length) Bookmarks.load()

  rootEl.value?.addEventListener('keydown', onDocumentKeydown)
})

async function init(): Promise<void> {
  await nextTick()
  setCustomIconType()
  if (props.conf.iconIMG) {
    state.customIconUrl = props.conf.iconIMG
    if (props.conf.iconIMGSrc) {
      if (state.customIconType === 'text') state.customIconTextValue = props.conf.iconIMGSrc
      else if (state.customIconType === 'url') state.customIconUrlValue = props.conf.iconIMGSrc
    }
  }
  if (nameInput.value) {
    nameInput.value.recalcTextHeight()
    nameInput.value.focus()
  }
}

function isNotTabsPanel(conf: PanelConfig): conf is BookmarksPanelConfig | HistoryPanelConfig {
  return !!conf && !Utils.isTabsPanel(conf)
}

function onDocumentKeydown(e: KeyboardEvent) {
  if (e.code === 'ArrowDown' || (!e.shiftKey && e.code === 'Tab')) {
    e.preventDefault()
    e.stopPropagation()
    focusField(1)
  } else if (e.code === 'ArrowUp' || (e.shiftKey && e.code === 'Tab')) {
    e.preventDefault()
    e.stopPropagation()
    focusField(-1)
  }
}

function focusField(dir: -1 | 1) {
  if (!document.activeElement || !rootEl.value) return

  const selectors = `[data-inactive="false"] .focus-el[tabindex="-1"],
.TextInput input,
.img-rm[tabindex="-1"],
.btn input,
.btn[tabindex="-1"]`
  const focusable = Array.from(rootEl.value.querySelectorAll(selectors))

  let focusIndex = focusable.findIndex(el => el === document.activeElement)
  if (focusIndex === -1 && dir === -1) focusIndex = focusable.length
  const nextFocusEl = focusable[focusIndex + dir]
  if (nextFocusEl instanceof HTMLElement) nextFocusEl.focus()
}

function onWheel(e: WheelEvent): void {
  if (!rootEl.value) return
  const scrollOffset = rootEl.value.scrollTop
  const maxScrollOffset = rootEl.value.scrollHeight - rootEl.value.offsetHeight
  if (scrollOffset === 0 && e.deltaY < 0) e.preventDefault()
  if (scrollOffset === maxScrollOffset && e.deltaY > 0) e.preventDefault()
}

function onNameInput(value: string): void {
  props.conf.name = value
  if (value) saveSidebarConfig(500)
}

function setIcon(value: string): void {
  props.conf.iconSVG = value
  saveSidebarConfig(250)
}

function setColor(value: browser.ColorName): void {
  props.conf.color = value
  if (props.conf.iconIMG) recolorCustomIcon()
  saveSidebarConfig(250)
}

// ---
// -- Custom icon
// -
function setCustomIconType(type?: string): void {
  if (type === undefined) {
    if (props.conf.iconIMG) {
      const normSrc = props.conf.iconIMGSrc?.trim()
      if (!normSrc) state.customIconType = 'file'
      else if (URL_RE.test(normSrc)) state.customIconType = 'url'
      else state.customIconType = 'text'
    } else {
      state.customIconType = ''
    }
  } else {
    if (state.customIconType === type) state.customIconType = ''
    else state.customIconType = type
  }
}

function onCustomIconTextInput(value: string): void {
  state.customIconTextValue = value
  updCustomIconTimeout = Utils.wait(updCustomIconTimeout, 500, () => {
    const normValue = state.customIconTextValue?.trim()
    if (!normValue) {
      state.customIconUrl = ''
      props.conf.iconIMG = ''
      props.conf.iconIMGSrc = ''
      saveSidebarConfig()
      return
    }

    props.conf.iconIMGSrc = normValue

    drawTextIcon()
  })
}

async function loadCustomIcon(): Promise<void> {
  if (!Permissions.reactive.webData) {
    const result = await Permissions.request('<all_urls>')
    if (!result) return
  }

  // TODO: Validation

  const normValue = state.customIconUrlValue?.trim()
  props.conf.iconIMGSrc = normValue

  if (normValue) state.customIconUrl = normValue
}

async function onCustomIconLoad(e: Event): Promise<void> {
  if (!state.customIconUrl) return
  if (state.customIconUrl.startsWith('data:image')) return

  const img = e.target as HTMLImageElement
  let canvas = document.createElement('canvas')
  let ctx = canvas.getContext('2d')
  if (!ctx) return
  canvas.width = img.naturalWidth
  canvas.height = img.naturalHeight
  ctx.imageSmoothingEnabled = false
  ctx.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight)
  let base64 = canvas.toDataURL('image/png')

  try {
    props.conf.iconIMG = await prepareCustomIcon(base64)
  } catch {
    return
  }
  state.customIconUrl = props.conf.iconIMG
  saveSidebarConfig()
}

function onCustomIconError(): void {
  state.customIconUrl = props.conf.iconIMG ?? ''
  // TODO: Some visual representation of this error
}

async function drawTextIcon() {
  if (!props.conf.iconIMGSrc) return
  let canvas = document.createElement('canvas')
  let ctx = canvas.getContext('2d')
  if (!ctx) return

  let [txt, color, font] = props.conf.iconIMGSrc.split('::')
  let x = 16
  let y = 16
  if (!color) {
    if (Styles.reactive.toolbarColorScheme === 'light') color = '#000000'
    if (Styles.reactive.toolbarColorScheme === 'dark') color = '#ffffff'
  }

  canvas.width = 32
  canvas.height = 32
  ctx.imageSmoothingEnabled = false
  ctx.fillStyle = color
  ctx.textBaseline = 'middle'
  ctx.textAlign = 'center'

  // // Find the max font size to fit canvas
  // let fontSize = 30
  // const maxFontSize = 50
  // let offset: number | null = 0
  // while (fontSize <= maxFontSize) {
  //   fontSize++
  //   font = `${fontSize}px system-ui`
  //   const offsetProbe = isTextFit(ctx, txt, font, 30, 30)
  //   if (offsetProbe !== null) offset = offsetProbe
  //   else break
  // }

  // Default font
  if (!font) font = '32px system-ui'

  // Vertically center the icon
  let offset = 0
  const offsetProbe = isTextFit(ctx, txt, font, 30, 30)
  if (offsetProbe !== null) offset = offsetProbe
  y += offset

  ctx.font = font

  ctx.fillText(txt, x, y, 32)

  let base64 = canvas.toDataURL('image/png')

  props.conf.iconIMG = await prepareCustomIcon(base64)
  state.customIconUrl = props.conf.iconIMG
  saveSidebarConfig()
}

function isTextFit(
  ctx: CanvasRenderingContext2D,
  txt: string,
  font: string,
  maxW: number,
  maxH: number
): number | null {
  ctx.font = font
  let metrics
  try {
    metrics = ctx.measureText(txt)
  } catch {
    return null
  }
  const w = metrics.width
  const h = metrics.actualBoundingBoxAscent + metrics.actualBoundingBoxDescent

  if (h <= maxH && w <= maxW) {
    return (metrics.actualBoundingBoxAscent - metrics.actualBoundingBoxDescent) / 2
  } else {
    return null
  }
}

async function prepareCustomIcon(base64icon: string): Promise<string> {
  state.customIconOriginal = base64icon
  if (state.customIconColorize) {
    let color = RGB_COLORS[props.conf.color]
    if (props.conf.color === 'toolbar') {
      if (Styles.reactive.toolbarColorScheme === 'light') color = '#000000'
      if (Styles.reactive.toolbarColorScheme === 'dark') color = '#ffffff'
    }
    base64icon = await Favicons.fillIcon(base64icon, color)
  }
  return await Favicons.resizeFavicon(base64icon)
}

function openCustomIconFile(importEvent: Event) {
  const target = importEvent.target as HTMLInputElement
  let file = target.files?.[0]
  if (!file) return onCustomIconRm()

  let reader = new FileReader()
  reader.onload = async fileEvent => {
    if (!fileEvent.target?.result) {
      onCustomIconRm()
      return Logs.err('Cannot import data: No file content')
    }

    let base64icon
    try {
      base64icon = await prepareCustomIcon(fileEvent.target.result as string)
    } catch {
      onCustomIconRm()
      return
    }
    props.conf.iconIMG = base64icon
    props.conf.iconIMGSrc = ''
    state.customIconTextValue = ''
    state.customIconUrlValue = ''
    state.customIconUrl = props.conf.iconIMG
    saveSidebarConfig()
  }
  reader.readAsDataURL(file)
}

function onCustomIconRm() {
  setCustomIconType('')
  state.customIconTextValue = ''
  state.customIconUrlValue = ''
  state.customIconOriginal = ''
  state.customIconUrl = ''
  props.conf.iconIMG = ''
  props.conf.iconIMGSrc = ''
  saveSidebarConfig()
}

function toggleCustomIconColorize() {
  state.customIconColorize = !state.customIconColorize
  recolorCustomIcon()
}

async function recolorCustomIcon() {
  if (state.customIconOriginal) {
    props.conf.iconIMG = await prepareCustomIcon(state.customIconOriginal)
    state.customIconUrl = props.conf.iconIMG
    saveSidebarConfig()
  }
}

function togglePanelLock(): void {
  props.conf.lockedPanel = !props.conf.lockedPanel
  saveSidebarConfig()
}

function toggleTempMode(): void {
  if (!isNotTabsPanel(props.conf)) return
  props.conf.tempMode = !props.conf.tempMode
  saveSidebarConfig()
}

function toggleSkipOnSwitching(): void {
  props.conf.skipOnSwitching = !props.conf.skipOnSwitching
  saveSidebarConfig()
}

function togglePanelNoEmpty(): void {
  if (!Utils.isTabsPanel(props.conf)) return
  props.conf.noEmpty = !props.conf.noEmpty
  saveSidebarConfig()
}

function togglePanelNewTabCtx(value: string): void {
  if (!Utils.isTabsPanel(props.conf)) return
  props.conf.newTabCtx = value
  saveSidebarConfig()
}

async function toggleNewTabCtxReopen() {
  if (!Permissions.reactive.webData && !Settings.state.newTabCtxReopen) {
    const result = await Permissions.request('<all_urls>')
    if (!result) return
  }

  Settings.state.newTabCtxReopen = !Settings.state.newTabCtxReopen
  Settings.saveDebounced(150)
}

function togglePanelDropTabCtx(value: string): void {
  if (!Utils.isTabsPanel(props.conf)) return
  props.conf.dropTabCtx = value
  saveSidebarConfig()
}

async function setBookmarksRootId(): Promise<void> {
  if (!Utils.isBookmarksPanel(props.conf)) return

  if (!Permissions.reactive.bookmarks) {
    const result = await browser.permissions.request({ origins: [], permissions: ['bookmarks'] })
    if (!result) return
  }

  const result = await Bookmarks.openBookmarksPopup({
    title: translate('popup.bookmarks.select_root_folder'),
    controls: [{ label: 'btn.save' }],
    location: BKM_MENU_ID,
    locationField: true,
    locationTree: true,
  })

  if (result && result.location) {
    props.conf.rootId = result.location
    saveSidebarConfig(500)
  }
}

function resetBookmarksRootId(): void {
  if (!Utils.isBookmarksPanel(props.conf)) return
  props.conf.rootId = BKM_ROOT_ID
  saveSidebarConfig(500)
}

function selectBookmarksViewMode(value: string): void {
  if (!Utils.isBookmarksPanel(props.conf)) return
  props.conf.viewMode = value
  saveSidebarConfig()
}

function toggleAutoConvert(): void {
  if (!Utils.isBookmarksPanel(props.conf)) return
  props.conf.autoConvert = !props.conf.autoConvert
  saveSidebarConfig()
}

function openRulesPopup() {
  Popups.openTabMoveRulesPopup(props.conf)
}

function openShortcutsPopup(): void {
  Popups.openNewTabShortcutsPopup(props.conf)
}

function getManageRulesBtnLabel(panel: TabsPanelConfig): string {
  let label = translate('panel.tab_move_rules_manage_btn')
  if (panel.moveRules.length) return label + ` (${panel.moveRules.length})`
  else return label
}

function getManageShortcutsBtnLabel(panel: TabsPanelConfig): string {
  const label = translate('panel.new_tab_shortcuts_manage_btn')
  if (panel.newTabBtns.length) return label + ` (${panel.newTabBtns.length})`
  else return label
}

function onFileInputKeyup(e: KeyboardEvent) {
  if (e.code === 'Escape') {
    e.stopPropagation()
    rootEl.value?.focus()
  }
}

function toggleMoveExclude(id: ID) {
  if (!Utils.isTabsPanel(props.conf)) return
  props.conf.moveExcludedTo = id
  saveSidebarConfig()
}
</script>
