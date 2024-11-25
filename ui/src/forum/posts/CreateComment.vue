<template>
  <div>
    <h3>Create Comment</h3>

    <div>
      <label for="Comment">Comment</label>
      <textarea name="Comment" v-model="comment" required></textarea>
    </div>

    <button :disabled="!isCommentValid" @click="createComment">
      Create Comment
    </button>
  </div>
</template>

<script lang="ts">
import { ActionHash, AgentPubKey, AppClient, DnaHash, EntryHash, Record } from "@holochain/client";
import { ComputedRef, defineComponent, inject } from "vue";
import { Comment } from "./types";

export default defineComponent({
  data(): {
    comment: string;
  } {
    return {
      comment: "",
    };
  },
  props: {
    postHash: {
      type: null,
      required: true,
    },
  },
  computed: {
    isCommentValid() {
      return true && this.comment !== "";
    },
  },
  mounted() {
    if (this.postHash === undefined) {
      throw new Error(`The postHash input is required for the CreateComment element`);
    }
  },
  methods: {
    async createComment() {
      const comment: Comment = {
        comment: this.comment!,
        post_hash: this.postHash!,
      };

      try {
        const record: Record = await this.client.callZome({
          role_name: "forum",
          zome_name: "posts",
          fn_name: "create_comment",
          payload: comment,
        });
        this.$emit("comment-created", record.signed_action.hashed.hash);
      } catch (e: any) {
        alert(e);
      }
    },
  },
  emits: ["comment-created"],
  setup() {
    const client = (inject("client") as ComputedRef<AppClient>).value;
    return { client };
  },
});
</script>
