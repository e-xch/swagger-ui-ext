<template>
  <div class="history">
    <scroll-bar>
      <ul>
        <li
          v-for="item in history"
          :key="item.id"
          @click="openInNewTab(item)"
          @contextmenu.prevent="showMenu($event, item)"
        >
          <span class="multi-line">{{ item.url }}</span>
        </li>
      </ul>
    </scroll-bar>
    <context-menu :data="menuData" :show.sync="menuShow" @select="menuSelect" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { RequestData } from "@/type/RequestData";
import Bus from "@/util/Bus";
import { BusEvent } from "@/type/BusEvent";
import historyDB from "@/api/HistoryDB";
import ContextMenu from "@/components/ContextMenu.vue";
import { MenuData } from "@/type/ComponentType";
import { isBlank } from "@/util/TextUtil";
import ScrollBar from "@/components/ScrollBar.vue";

@Component({
  components: {
    ContextMenu,
    ScrollBar
  }
})
export default class History extends Vue {
  private history: Array<RequestData> = [];

  private sourceHistory: Array<RequestData> = [];

  private reqData!: RequestData;

  private menuData: MenuData = {
    items: [
      { command: "openInCurrentTab", text: "当前标签打开" },
      { command: "openInBackgroundTab", text: "后台打开" },
      { command: "deleteHistory", text: "删除历史" }
    ],
    position: { top: "0px", left: "0px" }
  };

  private menuShow = false;

  created() {
    this.initHistory();
    Bus.$on(BusEvent.ADD_HISTORY, this.addHistory);
    Bus.$on(BusEvent.SEARCH_HISTORY, this.search);
  }

  private initHistory() {
    historyDB.all().then((history: Array<RequestData>) => {
      this.sourceHistory = this.history = history;
    });
  }

  private showMenu(e: MouseEvent, data: RequestData) {
    this.menuData.position = { top: e.pageY + "px", left: e.pageX + "px" };
    this.menuShow = true;
    this.reqData = data;
  }

  private addHistory(reqData: RequestData) {
    historyDB.remove(reqData);
    historyDB.add(reqData);
    this.initHistory();
  }

  /**
   * 新窗口打开
   * @param reqData
   */
  private openInNewTab(reqData: RequestData) {
    Bus.$emit(BusEvent.OPEN_TAB, reqData, { type: BusEvent.OPEN_IN_NEW_TAB });
  }

  private openInCurrentTab(reqData: RequestData) {
    Bus.$emit(BusEvent.OPEN_TAB, reqData, { type: BusEvent.OPEN_IN_CURRENT_TAB });
  }

  private openInBackgroundTab(reqData: RequestData) {
    Bus.$emit(BusEvent.OPEN_TAB, reqData, { type: BusEvent.OPEN_IN_BACKGROUND_TAB });
  }

  private deleteHistory(reqData: RequestData) {
    historyDB.remove(reqData);
    this.initHistory();
  }

  private menuSelect(command: string) {
    switch (command) {
      case "openInCurrentTab":
        this.openInCurrentTab(this.reqData);
        break;
      case "openInBackgroundTab":
        this.openInBackgroundTab(this.reqData);
        break;
      case "deleteHistory":
        this.deleteHistory(this.reqData);
        break;
    }
  }

  private search(value: string) {
    if (isBlank(value)) {
      this.history = this.sourceHistory;
      return;
    }
    this.history = this.history.filter(history => history.includes(value));
  }
}
</script>

<style lang="scss" scoped>
.history {
  ul {
    padding-left: 10px;
    margin: 0;
    height: calc(100vh - 168px);
  }
  ul li {
    list-style: none;
    color: #606266;
    font-size: 16px;
    padding: 2px 0;

    .multi-line {
      overflow-wrap: break-word;
      word-break: break-all;
      overflow: hidden;
      height: auto;
    }
  }
  ul li:hover {
    background-color: #f5f7fa;
  }
}
</style>
