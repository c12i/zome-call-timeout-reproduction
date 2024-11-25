<template>
  <section>
    <div>
      <label for="Title">Title</label>
      <input name="Title" v-model="title" required />
    </div>
    <div>
      <label for="Content">Content</label>
      <textarea name="Content" v-model="content" required></textarea>
    </div>
    <div>
      <button @click="$emit('edit-canceled')">Cancel</button>
      <button
        :disabled="!isPostValid"
        @click="updatePost"
      >
        Edit Post
      </button>
    </div>
  </section>
</template>

<script lang="ts">
import { ActionHash, AgentPubKey, AppClient, DnaHash, EntryHash, HolochainError, Record } from "@holochain/client";
import { decode } from "@msgpack/msgpack";
import { ComputedRef, defineComponent, inject } from "vue";
import { Post } from "./types";

export default defineComponent({
  data(): {
    title: string;
    content: string;
  } {
    const currentPost = decode((this.currentRecord.entry as any).Present.entry) as Post;
    return {
      title: currentPost.title,
      content: currentPost.content,
    };
  },
  props: {
    originalPostHash: {
      type: null,
      required: true,
    },
    currentRecord: {
      type: Object,
      required: true,
    },
  },
  computed: {
    currentPost() {
      return decode((this.currentRecord.entry as any).Present.entry) as Post;
    },
    isPostValid() {
      return true && this.title !== "" && this.content !== "";
    },
  },
  mounted() {
    if (this.currentRecord === undefined) {
      throw new Error(`The currentRecord input is required for the EditPost element`);
    }
    if (this.originalPostHash === undefined) {
      throw new Error(`The originalPostHash input is required for the EditPost element`);
    }
  },
  methods: {
    async updatePost() {
      const post: Post = {
        title: this.title,
        content: this.content,
      };

      try {
        const updateRecord: Record = await this.client.callZome({
          role_name: "forum",
          zome_name: "posts",
          fn_name: "update_post",
          payload: {
            original_post_hash: this.originalPostHash,
            previous_post_hash: this.currentRecord.signed_action.hashed.hash,
            updated_post: post,
          },
        });
        this.$emit("post-updated", updateRecord.signed_action.hashed.hash);
      } catch (e) {
        alert((e as HolochainError).message);
      }
    },
  },
  emits: ["post-updated", "edit-canceled"],
  setup() {
    const client = (inject("client") as ComputedRef<AppClient>).value;
    return { client };
  },
});
</script>
