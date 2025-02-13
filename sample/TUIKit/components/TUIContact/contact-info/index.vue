<template>
  <div
    :class="['TUI-contact-info', !isPC && 'TUI-contact-info-h5']"
    v-if="showContactInfo"
  >
    <div
      v-if="!isPC"
      :class="[
        'TUI-contact-info-header',
        !isPC && 'TUI-contact-info-h5-header',
      ]"
    >
      <div
        :class="[
          'TUI-contact-info-header-icon',
          !isPC && 'TUI-contact-info-h5-header-icon',
        ]"
        @click="resetContactSearchingUIData"
      >
        <Icon :file="backSVG" />
      </div>
      <div
        :class="[
          'TUI-contact-info-header-title',
          !isPC && 'TUI-contact-info-h5-header-title',
        ]"
      >
        {{ TUITranslateService.t("添加好友/群聊") }}
      </div>
    </div>
    <div
      :class="['TUI-contact-info-basic', !isPC && 'TUI-contact-info-h5-basic']"
    >
      <div
        :class="[
          'TUI-contact-info-basic-text',
          !isPC && 'TUI-contact-info-h5-basic-text',
        ]"
      >
        <div
          :class="[
            'TUI-contact-info-basic-text-name',
            !isPC && 'TUI-contact-info-h5-basic-text-name',
          ]"
        >
          {{ generateContactInfoName(contactInfoData) }}
        </div>
        <div
          :class="[
            'TUI-contact-info-basic-text-other',
            !isPC && 'TUI-contact-info-h5-basic-text-other',
          ]"
          v-for="item in contactInfoBasicList"
          :key="item.label"
        >
          {{
            `${TUITranslateService.t(item.label)}: 
            ${item.data}`
          }}
        </div>
      </div>
      <img
        :class="[
          'TUI-contact-info-basic-avatar',
          !isPC && 'TUI-contact-info-h5-basic-avatar',
        ]"
        :src="generateAvatar(contactInfoData)"
      />
    </div>
    <div
      :class="['TUI-contact-info-more', !isPC && 'TUI-contact-info-h5-more']"
      v-if="contactInfoMoreList[0]"
    >
      <div
        :class="[
          'TUI-contact-info-more-item',
          !isPC && 'TUI-contact-info-h5-more-item',
          item.labelPosition === CONTACT_INFO_LABEL_POSITION.TOP
            ? 'TUI-contact-info-more-item-top'
            : 'TUI-contact-info-more-item-left',
        ]"
        v-for="item in contactInfoMoreList"
        :key="item.key"
      >
        <div
          :class="[
            'TUI-contact-info-more-item-label',
            !isPC && 'TUI-contact-info-h5-more-item-label',
          ]"
        >
          {{ `${TUITranslateService.t(item.label)}` }}
        </div>
        <div
          :class="[
            'TUI-contact-info-more-item-content',
            !isPC && 'TUI-contact-info-h5-more-item-content',
          ]"
        >
          <div
            :class="[
              'TUI-contact-info-more-item-content-text',
              !isPC && 'TUI-contact-info-h5-more-item-content-text',
            ]"
            v-if="!item.editing"
          >
            <div
              :class="[
                'TUI-contact-info-more-item-content-text-data',
                !isPC && 'TUI-contact-info-h5-more-item-content-text-data',
              ]"
            >
              {{ item.data }}
            </div>
            <div
              @click="setEditing(item)"
              v-if="item.editable"
              :class="[
                'TUI-contact-info-more-item-content-text-icon',
                !isPC && 'TUI-contact-info-h5-more-item-content-text-icon',
              ]"
            >
              <Icon :file="editSVG" width="14px" height="14px" />
            </div>
          </div>
          <input
            :class="[
              'TUI-contact-info-more-item-content-input',
              !isPC && 'TUI-contact-info-h5-more-item-content-input',
            ]"
            v-else-if="item.editType === CONTACT_INFO_MORE_EDIT_TYPE.INPUT"
            type="text"
            v-model="item.data"
            @confirm="onContactInfoEmitSubmit(item)"
            @keyup.enter="onContactInfoEmitSubmit(item)"
          />
          <textarea
            :class="[
              'TUI-contact-info-more-item-content-textarea',
              !isPC && 'TUI-contact-info-h5-more-item-content-textarea',
            ]"
            confirm-type="done"
            v-else-if="item.editType === CONTACT_INFO_MORE_EDIT_TYPE.TEXTAREA"
            v-model="item.data"
          ></textarea>
          <div
            v-else-if="item.editType === CONTACT_INFO_MORE_EDIT_TYPE.SWITCH"
            @click="onContactInfoEmitSubmit(item)"
          >
            <SwitchBar :value="item.data" />
          </div>
        </div>
      </div>
    </div>
    <div
      :class="[
        'TUI-contact-info-button',
        !isPC && 'TUI-contact-info-h5-button',
      ]"
    >
      <button
        v-for="item in contactInfoButtonList"
        :class="[
          'TUI-contact-info-button-item',
          !isPC && 'TUI-contact-info-h5-button-item',
          item.type === CONTACT_INFO_BUTTON_TYPE.CANCEL
            ? `TUI-contact-info-button-item-cancel`
            : `TUI-contact-info-button-item-submit`,
        ]"
        :key="item.key"
        @click="onContactInfoButtonClicked(item)"
      >
        {{ TUITranslateService.t(item.label) }}
      </button>
    </div>
  </div>
</template>
<script setup lang="ts">
import TUIChatEngine, {
  TUIStore,
  StoreName,
  TUITranslateService,
  TUIGlobal,
} from "@tencentcloud/chat-uikit-engine";
import { ref, computed } from "../../../adapter-vue";
import {
  generateAvatar,
  generateContactInfoName,
  generateContactInfoBasic,
  isFriend,
  isApplicationType,
} from "../utils/index";
import {
  contactMoreInfoConfig,
  contactButtonConfig,
} from "./contact-info-config";
import Icon from "../../common/Icon.vue";
import editSVG from "../../../assets/icon/edit.svg";
import backSVG from "../../../assets/icon/back.svg";
import SwitchBar from "../../common/SwitchBar/index.vue";
import {
  IBlackListUserItem,
  IContactInfoMoreItem,
  IContactInfoButton,
} from "../../../interface";
import {
  CONTACT_INFO_LABEL_POSITION,
  CONTACT_INFO_MORE_EDIT_TYPE,
  CONTACT_INFO_BUTTON_TYPE,
} from "../../../constant";
import { deepCopy } from "../../TUIChat/utils/utils";

const emits = defineEmits(["switchConversation"]);

const isPC = ref(TUIGlobal.getPlatform() === "pc");

const contactInfoData = ref<any>({});
const contactInfoBasicList = ref<Array<{ label: string; data: string }>>([]);
const contactInfoMoreList = ref<Array<IContactInfoMoreItem>>([]);
const contactInfoButtonList = ref<Array<IContactInfoButton>>([]);
const showContactInfo = computed((): boolean => {
  for (let i in contactInfoData.value) {
    return true;
  }
  return false;
});

const setEditing = (item: any) => {
  item.editing = true;
};

// 是否为群组类型
const isGroup = computed((): boolean =>
  contactInfoData?.value?.groupID ? true : false
);
// 是否为申请类型
const isApplication = computed((): boolean => {
  return isApplicationType(contactInfoData?.value);
});
// 是否为双向好友关系, 若为群组类型则一直保持false
const isBothFriend = ref<boolean>(false);
// 是否是群成员（包含普通成员、管理员、群主）
const isGroupMember = computed((): boolean => {
  return contactInfoData?.value?.selfInfo?.userID ? true : false;
});
// 是否是黑名单用户, 若为群组类型则一直保持false
const isInBlackList = computed((): boolean => {
  return (
    !isGroup.value &&
    blackList.value?.findIndex(
      (item: IBlackListUserItem) =>
        item?.userID === contactInfoData?.value?.userID
    ) >= 0
  );
});

const blackList = ref<Array<IBlackListUserItem>>([]);

const resetContactInfoUIData = () => {
  contactInfoData.value = {};
  contactInfoBasicList.value = [];
  contactInfoMoreList.value = [];
  contactInfoButtonList.value = [];
};

const resetContactSearchingUIData = () => {
  TUIStore.update(StoreName.CUSTOM, "currentContactInfo", {});
  TUIStore.update(StoreName.CUSTOM, "currentContactSearchingStatus", false);
  TUIGlobal?.closeSearching && TUIGlobal?.closeSearching();
};

TUIStore.watch(StoreName.USER, {
  userBlacklist: (userBlacklist: Array<IBlackListUserItem>) => {
    blackList.value = userBlacklist;
  },
});

const onContactInfoEmitSubmit = (item: any) => {
  item.editSubmitHandler &&
    item.editSubmitHandler({
      item,
      contactInfoData: contactInfoData.value,
      isBothFriend: isBothFriend.value,
      isInBlackList: isInBlackList.value,
    });
};

const onContactInfoButtonClicked = (item: any) => {
  item.onClick &&
    item.onClick({
      contactInfoData: contactInfoData.value,
      contactInfoMoreList: contactInfoMoreList.value,
    });
  if (
    item.key === "enterGroupConversation" ||
    item.key === "enterC2CConversation"
  ) {
    emits("switchConversation", contactInfoData.value);
    // 清空当前 contact 相关数据信息
    resetContactSearchingUIData();
  }
};

const generateMoreInfo = async () => {
  // 非 好友申请类 信息展示
  if (!isApplication.value) {
    // 非好友关系 / 非群成员 且 不在黑名单中 且 不是直播群，展示 申请加群/加好友验证信息 填写
    if (
      (!isGroup.value && !isBothFriend.value && !isInBlackList.value) ||
      (isGroup.value &&
        !isGroupMember.value &&
        contactInfoData?.value?.type !== TUIChatEngine?.TYPES?.GRP_AVCHATROOM)
    ) {
      contactMoreInfoConfig.setWords.data = "";
      contactInfoMoreList.value.push(contactMoreInfoConfig.setWords);
    }
    // 用户界面，展示 备注名 设置，包含：
    // 1. 若为好友关系，直接修改好友相关资料
    // 2. 若非好友关系，且非黑名单用户，提供 申请加好友备注信息 填写
    if (!isGroup.value && !isInBlackList.value) {
      contactMoreInfoConfig.setRemark.data =
        contactInfoData?.value?.remark || "";
      contactMoreInfoConfig.setRemark.editing = false;
      contactInfoMoreList.value.push(contactMoreInfoConfig.setRemark);
    }
    // 用户界面，展示 黑名单 设置，包含：
    // 1. 若为好友关系，提供设置黑名单入口
    // 2. 若已在黑名单中，提供接触黑名单入口
    if (!isGroup.value && (isBothFriend.value || isInBlackList.value)) {
      contactMoreInfoConfig.blackList.data = isInBlackList.value || false;
      contactInfoMoreList.value.push(contactMoreInfoConfig.blackList);
    }
  } else {
    // 对方 好友请求/群聊申请
    contactMoreInfoConfig.displayWords.data =
      contactInfoData?.value?.wording || "";
    contactInfoMoreList.value.push(contactMoreInfoConfig.displayWords);
  }
};

const generateButton = () => {
  // 黑名单用户，在解除黑名单前没有任何操作button
  if (isInBlackList.value) {
    return;
  }
  // 非黑名单用户
  if (isApplication.value) {
    if (
      contactInfoData?.value?.type ===
      TUIChatEngine?.TYPES?.SNS_APPLICATION_SENT_TO_ME
    ) {
      contactInfoButtonList?.value?.push(
        contactButtonConfig.refuseFriendApplication
      );
      contactInfoButtonList?.value?.push(
        contactButtonConfig.acceptFriendApplication
      );
    }
  } else {
    if (isGroup.value && isGroupMember.value) {
      // 群聊信息页+是群成员
      switch (contactInfoData?.value?.selfInfo?.role) {
        case "Owner":
          contactInfoButtonList?.value?.push(contactButtonConfig.dismissGroup);
          break;
        default:
          contactInfoButtonList?.value?.push(contactButtonConfig.quitGroup);
          break;
      }
      contactInfoButtonList?.value?.push(
        contactButtonConfig.enterGroupConversation
      );
    } else if (!isGroup.value && isBothFriend.value) {
      // 用户信息页+是好友关系
      contactInfoButtonList?.value?.push(contactButtonConfig.deleteFriend);
      contactInfoButtonList?.value?.push(
        contactButtonConfig.enterC2CConversation
      );
    } else {
      // 群聊信息页+不是群成员 / 用户信息页 + 非好友关系
      if (isGroup.value) {
        contactInfoButtonList?.value?.push(
          contactInfoData?.value?.type === TUIChatEngine?.TYPES?.GRP_AVCHATROOM
            ? contactButtonConfig.joinAVChatGroup
            : contactButtonConfig.joinGroup
        );
      } else {
        contactInfoButtonList?.value?.push(contactButtonConfig.addFriend);
      }
    }
  }
};

// 总: 当前联系人信息页数据源获取
TUIStore.watch(StoreName.CUSTOM, {
  currentContactInfo: async (data: object) => {
    // 去重
    if (
      contactInfoData.value &&
      data &&
      JSON.stringify(contactInfoData.value) === JSON.stringify(data)
    ) {
      return;
    }
    resetContactInfoUIData();
    // deep clone
    contactInfoData.value = deepCopy(data) || {};
    if (Object.keys(contactInfoData.value)?.length === 0) {
      return;
    }
    contactInfoBasicList.value = generateContactInfoBasic(
      contactInfoData.value
    );
    isBothFriend.value = await isFriend(contactInfoData.value);
    generateMoreInfo();
    generateButton();
  },
});
</script>
<style lang="scss" scoped src="./style/index.scss"></style>
