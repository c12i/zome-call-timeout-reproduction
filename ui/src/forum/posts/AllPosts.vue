<template>
  <progress v-if="loading"></progress>
  <div v-else>
    <div class="alert" v-if="error">
      Error fetching the posts: {{ error.message }}.
    </div>
    <div v-else-if="hashes && hashes.length > 0">
      <PostDetail v-for="(hash, i) in hashes" :key="i" :post-hash="hash" @post-deleted="fetchPost()">
      </PostDetail>
    </div>
    <div class="alert" v-else>No posts found.</div>
  </div>
</template>

<script lang="ts">
import {
  ActionHash,
  AppClient,
  HolochainError,
  Link,
  Record,
  SignalType,
} from "@holochain/client";
import { ComputedRef, defineComponent, inject, toRaw } from "vue";
import PostDetail from "./PostDetail.vue";
import { PostsSignal } from "./types";

export default defineComponent({
  components: {
    PostDetail,
  },
  data(): { hashes: Array<ActionHash> | undefined; loading: boolean; error: any } {
    return {
      hashes: undefined,
      loading: false,
      error: undefined,
    };
  },
  async mounted() {
    await this.fetchPost();
    toRaw(this.client).on("signal", signal => {
      if (!(SignalType.App in signal)) return;
      if (signal.App.zome_name !== "posts") return;
      const payload = signal.App.payload as PostsSignal;
      if (payload.type !== "EntryCreated") return;
      if (payload.app_entry.type !== "Post") return;
      if (this.hashes) this.hashes.push(payload.action.hashed.hash);
    });
  },
  methods: {
    async fetchPost() {
      try {
        this.loading = true;
        const links: Array<Link> = await this.client.callZome({
          role_name: "forum",
          zome_name: "posts",
          fn_name: "get_all_posts",
        });
        this.hashes = links.map(l => l.target);
      } catch (e) {
        this.error = e as HolochainError;
      } finally {
        this.loading = false;
      }
    },
  },
  setup() {
    const client = (inject("client") as ComputedRef<AppClient>).value;
    return { client };
  },
});
</script>
