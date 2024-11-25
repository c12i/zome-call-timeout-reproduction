<template>
  <div>
    <progress v-if="loading"></progress>
    <div v-else>
      <h2>Welcome to the Forum hApp</h2>
      <AllPosts></AllPosts>
      <CreatePost></CreatePost>
    </div>
  </div>
</template>

<script lang="ts">
import { AppClient, AppWebsocket } from "@holochain/client";
import { computed, defineComponent } from "vue";
import AllPosts from "./forum/posts/AllPosts.vue";
import CreatePost from "./forum/posts/CreatePost.vue";

export default defineComponent({
  components: {
    AllPosts,
    CreatePost,
  },
  data(): {
    client: AppClient | undefined;
    loading: boolean;
  } {
    return {
      client: undefined,
      loading: false,
    };
  },
  async mounted() {
    this.loading = true;
    try {
      this.client = await AppWebsocket.connect();
    } catch (e) {
      console.error(e);
    } finally {
      this.loading = false;
    }
  },
  provide() {
    return {
      client: computed(() => this.client),
    };
  },
});
</script>
