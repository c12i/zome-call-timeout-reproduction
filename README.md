# how to reproduce

- Setup

```bash
nix develop
pnpm install
pnpm start
```

- The hApp makes an initial zome call when it starts to fetch all posts, so you will notice this timeout a little while after it loads up and fails to fetch. You can also try reproduce the time-out by inserting a post.
