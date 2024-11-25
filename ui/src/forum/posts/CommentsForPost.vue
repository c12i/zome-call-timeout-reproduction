<template>
  <progress v-if="loading"></progress>
  <div class="alert" v-if="error">
    Error fetching the comments: {{ error }}.
  </div>
  <div v-else-if="hashes && hashes.length > 0">
    <CommentDetail
      v-for="(hash, i) in hashes"
      :key="i"
      :comment-hash="hash"
      @comment-deleted="fetchComments()"
    >
    </CommentDetail>
  </div>
  <div class="alert" v-else>No comments found for this post.</div>
</template>

<script lang="ts">
import { ActionHash, AgentPubKey, AppClient, HolochainError, Link, Record, SignalType } from "@holochain/client";
import { decode } from "@msgpack/msgpack";
import { ComputedRef, defineComponent, inject, toRaw } from "vue";
import CommentDetail from "./CommentDetail.vue";
import { PostsSignal } from "./types";

export default defineComponent({
  components: {
    CommentDetail,
  },
  props: {
    postHash: {
      type: Object,
      required: true,
    },
  },
  data(): { hashes: Array<ActionHash> | undefined; loading: boolean; error: any } {
    return {
      hashes: [],
      loading: false,
      error: undefined,
    };
  },
  async mounted() {
    if (this.postHash === undefined) {
      throw new Error(`The postHashHash input is required for the CommentsForPost element`);
    }

    await this.fetchComments();

    toRaw(this.client).on("signal", async signal => {
      if (!(SignalType.App in signal)) return;
      if (signal.App.zome_name !== "posts") return;
      const payload = signal.App.payload as PostsSignal;
      if (!(payload.type === "EntryCreated" && payload.app_entry.type === "Comment")) return;
      await this.fetchComments();
    });
  },
  methods: {
    async fetchComments() {
      this.loading = true;
      try {
        const links: Array<Link> = await this.client.callZome({
          role_name: "forum",
          zome_name: "posts",
          fn_name: "get_comments_for_post",
          payload: this.postHash,
        });
        this.hashes = links.map(l => l.target);
      } catch (e) {
        this.error = e as HolochainError;
      }
      this.loading = false;
    },
  },
  setup() {
    const client = (inject("client") as ComputedRef<AppClient>).value;
    return { client };
  },
});
</script>
