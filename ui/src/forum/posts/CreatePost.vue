<template>
  <div>
    <h3>Create Post</h3>

    <div>
      <label for="Title">Title</label>
      <input name="Title" v-model="title" required />
    </div>
    <div>
      <label for="Content">Content</label>
      <textarea name="Content" v-model="content" required></textarea>
    </div>

    <button :disabled="!isPostValid" @click="createPost">
      Create Post
    </button>
  </div>
</template>

<script lang="ts">
import { ActionHash, AgentPubKey, AppClient, DnaHash, EntryHash, Record } from "@holochain/client";
import { ComputedRef, defineComponent, inject } from "vue";
import { Post } from "./types";

export default defineComponent({
  data(): {
    title: string;
    content: string;
  } {
    return {
      title: "",
      content: "",
    };
  },
  computed: {
    isPostValid() {
      return true && this.title !== "" && this.content !== "";
    },
  },
  mounted() {
  },
  methods: {
    async createPost() {
      const post: Post = {
        title: this.title!,
        content: this.content!,
      };

      try {
        const record: Record = await this.client.callZome({
          role_name: "forum",
          zome_name: "posts",
          fn_name: "create_post",
          payload: post,
        });
        this.$emit("post-created", record.signed_action.hashed.hash);
      } catch (e: any) {
        alert(e);
      }
    },
  },
  emits: ["post-created"],
  setup() {
    const client = (inject("client") as ComputedRef<AppClient>).value;
    return { client };
  },
});
</script>
