<template>
  <div v-if="!loading">
    <section v-else-if="record">
      <div>
        <span><strong>Comment: </strong></span>
        <span>{{ comment?.comment }} </span>
      </div>

      <div>
        <button @click="deleteComment">delete</button>
      </div>
    </section>
    <div class="alert" v-else>The requested comment was not found.</div>
  </div>
  <progress v-else></progress>
  <div class="alert" v-if="error">Error: {error.message}</div>
</template>

<script lang="ts">
import { ActionHash, AgentPubKey, AppClient, DnaHash, EntryHash, HolochainError, Record } from "@holochain/client";
import { decode } from "@msgpack/msgpack";
import { ComputedRef, defineComponent, inject } from "vue";
import { Comment } from "./types";

export default defineComponent({
  props: {
    commentHash: {
      type: Object,
      required: true,
    },
  },
  data(): {
    record: Record | undefined;
    loading: boolean;
    error: HolochainError | undefined;
  } {
    return {
      record: undefined,
      loading: true,
      error: undefined,
    };
  },
  computed: {
    comment() {
      if (!this.record) return;
      return decode((this.record.entry as any).Present.entry) as Comment;
    },
  },
  async mounted() {
    if (this.commentHash === undefined) {
      throw new Error(`The commentHash input is required for the CommentDetail element`);
    }
    await this.fetchComment();
  },
  methods: {
    async fetchComment() {
      try {
        this.loading = true;
        this.record = await this.client.callZome({
          role_name: "forum",
          zome_name: "posts",
          fn_name: "get_comment",
          payload: this.commentHash,
        });
      } catch (e) {
        this.error = e as HolochainError;
      } finally {
        this.loading = false;
      }
    },
    async deleteComment() {
      try {
        this.loading = true;
        await this.client.callZome({
          role_name: "forum",
          zome_name: "posts",
          fn_name: "delete_comment",
          payload: this.commentHash,
        });
        this.$emit("comment-deleted", this.commentHash);
      } catch (e: any) {
        this.error = e as HolochainError;
      } finally {
        this.loading = false;
      }
    },
  },
  emits: ["comment-deleted"],
  setup() {
    const client = (inject("client") as ComputedRef<AppClient>).value;
    return { client };
  },
});
</script>
