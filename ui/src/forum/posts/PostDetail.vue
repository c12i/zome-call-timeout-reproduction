<template>
  <div v-if="!loading">
    <div v-if="editing">
      <EditPost
        :original-post-hash="postHash"
        :current-record="record"
        @post-updated="editing = false; fetchPost()"
        @edit-canceled="editing = false"
      />
    </div>
    <section v-else-if="record">
      <div>
        <span><strong>Title: </strong></span>
        <span>{{ post?.title }} </span>
      </div>

      <div>
        <span><strong>Content: </strong></span>
        <span>{{ post?.content }} </span>
      </div>

      <div>
        <button @click="editing = true">edit</button>
        <button @click="deletePost">delete</button>
      </div>
    </section>
    <div class="alert" v-else>The requested post was not found.</div>
  </div>
  <progress v-else></progress>
  <div class="alert" v-if="error">Error: {error.message}</div>
</template>

<script lang="ts">
import { ActionHash, AgentPubKey, AppClient, DnaHash, EntryHash, HolochainError, Record } from "@holochain/client";
import { decode } from "@msgpack/msgpack";
import { ComputedRef, defineComponent, inject } from "vue";
import EditPost from "./EditPost.vue";
import { Post } from "./types";

export default defineComponent({
  components: {
    EditPost,
  },
  props: {
    postHash: {
      type: Object,
      required: true,
    },
  },
  data(): {
    record: Record | undefined;
    loading: boolean;
    error: HolochainError | undefined;
    editing: boolean;
  } {
    return {
      record: undefined,
      loading: true,
      error: undefined,
      editing: false,
    };
  },
  computed: {
    post() {
      if (!this.record) return;
      return decode((this.record.entry as any).Present.entry) as Post;
    },
  },
  async mounted() {
    if (this.postHash === undefined) {
      throw new Error(`The postHash input is required for the PostDetail element`);
    }
    await this.fetchPost();
  },
  methods: {
    async fetchPost() {
      try {
        this.loading = true;
        this.record = await this.client.callZome({
          role_name: "forum",
          zome_name: "posts",
          fn_name: "get_latest_post",
          payload: this.postHash,
        });
      } catch (e) {
        this.error = e as HolochainError;
      } finally {
        this.loading = false;
      }
    },
    async deletePost() {
      try {
        this.loading = true;
        await this.client.callZome({
          role_name: "forum",
          zome_name: "posts",
          fn_name: "delete_post",
          payload: this.postHash,
        });
        this.$emit("post-deleted", this.postHash);
      } catch (e: any) {
        this.error = e as HolochainError;
      } finally {
        this.loading = false;
      }
    },
  },
  emits: ["post-deleted"],
  setup() {
    const client = (inject("client") as ComputedRef<AppClient>).value;
    return { client };
  },
});
</script>
