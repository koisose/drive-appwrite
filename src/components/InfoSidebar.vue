<template>
  <div class="min-w-[400px] max-w-[400px] flex flex-col">
    <div v-if="$store.state.showInfo" class="mx-5 mb-3">
      <div class="my-4">
        <FeatherIcon
          name="x"
          class="h-4 cursor-pointer"
          @click="$store.commit('setShowInfo', false)" />
      </div>
      <div class="flex items-center">
        <svg
          v-if="entity.is_group"
          :style="{ fill: entity.color }"
          :draggable="false"
          class="h-5 mr-2.5"
          viewBox="0 0 46 40"
          fill="#32a852"
          xmlns="http://www.w3.org/2000/svg">
          <path
            d="M0.368197 17.3301C0.17175 15.5535 1.56262 14.0004 3.35002 14.0004H42.65C44.4374 14.0004 45.8283 15.5535 45.6318 17.3301L43.7155 34.6599C43.3794 37.6999 40.8104 40.0004 37.7519 40.0004H8.24812C5.18961 40.0004 2.62062 37.6999 2.28447 34.6599L0.368197 17.3301Z" />
          <path
            d="M43.125 11V9C43.125 6.79086 41.3341 5 39.125 5H20.312C20.1914 5 20.0749 4.95643 19.9839 4.8773L14.6572 0.245394C14.4752 0.0871501 14.2422 0 14.001 0H6.87501C4.66587 0 2.87501 1.79086 2.87501 4V11C2.87501 11.5523 3.32272 12 3.87501 12H42.125C42.6773 12 43.125 11.5523 43.125 11Z" />
        </svg>
        <img
          v-else
          :src="getIconUrl(formatMimeType(entity.mime_type))"
          :draggable="false"
          class="h-5 mr-2.5" />
        <div class="font-semibold truncate text-2xl">
          {{ entity.title }}
        </div>
      </div>
    </div>
    <div
      class="flex cursor-pointer text-base"
      :class="$store.state.showInfo ? 'min-h-[45px]' : 'min-h-[48px]'">
      <div
        class="w-1/2 flex border-b"
        :class="tab ? 'text-gray-600' : 'border-blue-500'"
        @click="tab = 0">
        <div class="m-auto">Detail</div>
      </div>
      <div
        class="w-1/2 flex border-b"
        :class="tab ? 'border-blue-500' : 'text-gray-600'"
        @click="tab = 1">
        <div class="m-auto">Comments</div>
      </div>
    </div>
    <div
      v-if="!tab"
      class="px-5 py-6 space-y-7 h-full flex flex-col z-0 overflow-y-auto">
      <FileRender
        v-if="isImage && $store.state.showInfo"
        :preview-entity="entity" />
      <div v-if="entity.owner === 'me'">
        <div class="text-lg font-medium mb-4">Manage Access</div>
        <div class="flex flex-row">
          <Button class="h-7" @click="showShareDialog = true">Share</Button>
        </div>
      </div>
      <div v-if="entity.owner === 'me' || $resources.entityTags.data?.length">
        <div class="text-lg font-medium mb-4">Tag</div>
        <div class="flex flex-wrap gap-2">
          <Tag
            v-for="tag in $resources.entityTags.data"
            :key="tag"
            :tag="tag"
            :entity="entity"
            @success="
              () => {
                $resources.userTags.fetch();
                $resources.entityTags.fetch();
              }
            " />
          <Button
            v-if="!addTag && entity.owner === 'me'"
            class="h-6 text-[12px] text-gray-800"
            icon-left="plus"
            @click="addTag = true">
            Add tag
          </Button>
        </div>

        <TagInput
          v-if="addTag"
          :class="{ 'mt-2': $resources.entityTags.data.length }"
          :entity="entity"
          :unadded-tags="unaddedTags"
          @success="
            () => {
              $resources.userTags.fetch();
              $resources.entityTags.fetch();
              addTag = false;
            }
          "
          @close="addTag = false" />
      </div>
      <div class="grow">
        <div class="text-lg font-medium mb-4">Properties</div>
        <div class="flex text-base">
          <div class="w-1/2 text-gray-600 space-y-2">
            <div>Type</div>
            <div>Size</div>
            <div>Modified</div>
            <div>Created</div>
            <div>Owner</div>
          </div>
          <div class="w-1/2 space-y-2">
            <div>{{ formattedMimeType }}</div>
            <div>{{ entity.file_size }}</div>
            <div>{{ entity.modified }}</div>
            <div>{{ entity.creation }}</div>
            <div>{{ entity.owner }}</div>
          </div>
        </div>
      </div>
      <div class="text-gray-600 text-base">Viewers can download this file.</div>
    </div>
    <div v-else class="px-5 py-6 h-full overflow-y-auto">
      <div v-if="entity.allow_comments" class="space-y-5">
        <div v-if="userId != 'Guest'" class="flex items-center gap-3">
          <Avatar :label="fullName" :image-u-r-l="imageURL" class="h-7 w-7" />
          <input
            v-model="newComment"
            type="text"
            placeholder="Add comment or update..."
            class="grow h-10 bg-white focus:bg-white border border-gray-200 focus:border-gray-200 rounded-lg text-[13px] placeholder-gray-600"
            @keydown.enter="postComment" />
        </div>
        <div
          v-for="comment in $resources.comments.data"
          :key="comment"
          class="flex gap-3">
          <Avatar
            :label="comment.comment_by"
            :image-u-r-l="comment.user_image"
            class="h-7 w-7" />
          <div>
            <span class="mb-1">
              <span class="text-[14px] font-medium">
                {{ comment.comment_by }}
              </span>
              <span class="text-gray-500 text-base">{{ " ∙ " }}</span>
              <span class="text-gray-700 text-base">
                {{ comment.creation }}
              </span>
            </span>
            <div class="text-base">{{ comment.content }}</div>
          </div>
        </div>
      </div>
      <div v-else class="text-gray-600 text-base">
        Comments have been disabled for this
        {{ entity.is_group ? "folder" : "file" }} by its owner.
      </div>
    </div>
    <ShareDialog
      v-if="showShareDialog"
      v-model="showShareDialog"
      :entity-name="entity.name" />
  </div>
</template>

<script>
import { FeatherIcon, Avatar, call } from "frappe-ui";
import ShareDialog from "@/components/ShareDialog.vue";
import TagInput from "@/components/TagInput.vue";
import Tag from "@/components/Tag.vue";
import FileRender from "@/components/FileRender.vue";
import { formatMimeType, formatDate } from "@/utils/format";
import getIconUrl from "@/utils/getIconUrl";

export default {
  name: "InfoSidebar",
  components: {
    FeatherIcon,
    Avatar,
    ShareDialog,
    TagInput,
    Tag,
    FileRender,
  },

  props: {
    entity: {
      type: Object,
      required: true,
    },
  },

  setup() {
    return { formatMimeType, getIconUrl };
  },

  data() {
    return {
      tab: 0,
      newComment: "",
      showShareDialog: false,
      addTag: false,
    };
  },

  computed: {
    userId() {
      return this.$store.state.auth.user_id;
    },
    fullName() {
      return this.$store.state.user.fullName;
    },
    imageURL() {
      return this.$store.state.user.imageURL;
    },
    isImage() {
      return this.entity.mime_type?.startsWith("image/");
    },
    formattedMimeType() {
      if (this.entity.is_group) return "Folder";
      const file = formatMimeType(this.entity.mime_type);
      return file.charAt(0).toUpperCase() + file.slice(1);
    },
    /*     folderColor() {
      return this.entity.is_group ? this.entity.color : null
    }; */
    unaddedTags() {
      return this.$resources.userTags.data.filter(
        ({ name: id1 }) =>
          !this.$resources.entityTags.data.some(({ name: id2 }) => id2 === id1)
      );
    },
  },

  methods: {
    async postComment() {
      try {
        await call("frappe.desk.form.utils.add_comment", {
          reference_doctype: "Drive Entity",
          reference_name: this.entity.name,
          content: this.newComment,
          comment_email: this.userId,
          comment_by: this.fullName,
        });
        this.newComment = "";
        this.$resources.comments.fetch();
      } catch (e) {
        console.log(e);
      }
    },
  },

  resources: {
    comments() {
      return {
        url: "drive.api.files.list_entity_comments",
        params: { entity_name: this.entity.name },
        onSuccess(data) {
          data.forEach((comment) => {
            comment.creation = formatDate(comment.creation);
          });
        },
        onError(error) {
          console.log(error);
        },
        auto: true,
      };
    },
    userTags() {
      return {
        url: "drive.api.tags.get_user_tags",
        onError(error) {
          if (error.messages) {
            console.log(error.messages);
          }
        },
        auto: true,
      };
    },
    entityTags() {
      return {
        url: "drive.api.tags.get_entity_tags",
        params: { entity: this.entity.name },
        onError(error) {
          if (error.messages) {
            console.log(error.messages);
          }
        },
        auto: true,
      };
    },
  },
};
</script>
