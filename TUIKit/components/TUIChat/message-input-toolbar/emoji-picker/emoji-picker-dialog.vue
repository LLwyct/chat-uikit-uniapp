<template>
  <div
    :class="['emoji-picker', !isPC && 'emoji-picker-h5']"
    ref="emojiPickerDialog"
  >
    <ul
      :class="['emoji-picker-list', !isPC && 'emoji-picker-h5-list']"
      ref="emojiPickerListRef"
    >
      <li
        class="emoji-picker-list-item"
        v-for="(childrenItem, childrenIndex) in currentEmojiList"
        :key="childrenIndex"
        @click="select(childrenItem, childrenIndex)"
      >
        <img
          v-if="currentTabItem.type === EMOJI_TYPE.BASIC"
          class="emoji"
          :src="currentTabItem.url + basicEmojiMap[childrenItem]"
        />
        <img
          v-else-if="currentTabItem.type === EMOJI_TYPE.BIG"
          class="emoji-big"
          :src="currentTabItem.url + childrenItem + '@2x.png'"
        />
        <img
          v-else
          class="emoji-custom"
          :src="currentTabItem.url + childrenItem"
        />
      </li>
    </ul>
    <ul class="emoji-picker-tab">
      <li
        class="emoji-picker-tab-item"
        v-for="(item, index) in list"
        :key="index"
        @click="toggleEmojiTab(index)"
      >
        <Icon
          v-if="item.type === EMOJI_TYPE.BASIC"
          class="icon"
          :file="faceIcon"
        ></Icon>
        <img
          v-else-if="item.type === EMOJI_TYPE.BIG"
          class="icon-big"
          :src="item.url + item.list[0] + '@2x.png'"
        />
        <img v-else class="icon-custom" :src="item.url + item.list[0]" />
      </li>
      <li class="send-btn" v-if="isUniFrameWork" @click="sendMessage">发送</li>
    </ul>
  </div>
</template>
<script lang="ts" setup>
import {
  TUIGlobal,
  TUIChatService,
  TUIStore,
  StoreName,
  IConversationModel,
  SendMessageParams,
} from "@tencentcloud/chat-uikit-engine";
import { ref } from "../../../../adapter-vue";
import { isUniFrameWork } from "../../../../utils/is-uni";
import { emojiList, basicEmojiMap } from "../../utils/emojiList";
import { IEmojiList, IEmojiListItem } from "../../../../interface";
import { EMOJI_TYPE } from ".././../../../constant";
import Icon from "../../../common/Icon.vue";
import faceIcon from "../../../../assets/icon/face.png";

const emits = defineEmits(["insertEmoji", "onClose", "sendMessage"]);

const isPC = ref(TUIGlobal.getPlatform() === "pc");
const list = ref<IEmojiList>(emojiList);
const currentTabIndex = ref<number>(0);
const currentTabItem = ref<IEmojiListItem>(list?.value[0]);
const currentEmojiList = ref<Array<string>>(list?.value[0]?.list);
const currentConversation = ref();
const emojiPickerDialog = ref();
const emojiPickerListRef = ref();

TUIStore.watch(StoreName.CONV, {
  currentConversation: (conversation: typeof IConversationModel) => {
    currentConversation.value = conversation;
  },
});

const toggleEmojiTab = (index: number) => {
  currentTabIndex.value = index;
  currentTabItem.value = list?.value[index];
  currentEmojiList.value = list?.value[index]?.list;
  // 滚动条回滚到顶部
  // 原生 web & h5
  if (!isUniFrameWork) {
    emojiPickerListRef?.value && (emojiPickerListRef.value.scrollTop = 0);
  }
};

const select = (item: any, index: number) => {
  const options: any = {
    name: item,
    type: currentTabItem?.value?.type,
  };
  switch (currentTabItem?.value?.type) {
    case EMOJI_TYPE.BASIC:
      options.url = currentTabItem?.value?.url + basicEmojiMap[item];
      emits("insertEmoji", options);
      break;
    case EMOJI_TYPE.BIG:
      sendFaceMessage(index, currentTabItem.value);
      break;
    case EMOJI_TYPE.CUSTOM:
      sendFaceMessage(index, currentTabItem.value);
      break;
    default:
      break;
  }
  isPC.value && emits("onClose");
};

const sendFaceMessage = (index: number, listItem: IEmojiListItem) => {
  const options = {
    to:
      currentConversation?.value?.groupProfile?.groupID ||
      currentConversation?.value?.userProfile?.userID,
    conversationType: currentConversation?.value?.type,
    payload: {
      index: listItem.index,
      data: listItem.list[index],
    },
  } as typeof SendMessageParams;
  TUIChatService.sendFaceMessage(options);
};

const sendMessage = () => {
  emits("sendMessage");
};
</script>
<style lang="scss" scoped src="./style/index.scss"></style>
