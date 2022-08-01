# carmelo0x99.github.io
Personal notes, small guides, ramblings I'd like to store somewhere public

----

### Hugo primer
`brew install hugo`

#### Start Hugo server in "draft" mode
`hugo server -D [--bind="0.0.0.0"]`

#### Edit posts
`vi content/posts/<filename>`

#### Convert draft pages into .html files
`hugo -D`

#### Copy public hierarchy (e.g. posts/, images/, 404.html) to top level directory
`cp -a public/* .`

#### Commit!!!

----

### When cloning
Make sure you re-import the theme submodule as such:
```
$ git submodule init

$ git submodule update
```

Finally, re-start `hugo`.

----

### Useful links
- [Hugo](https://gohugo.io)
  - [Quick Start](https://gohugo.io/getting-started/quick-start/)
  - [Hosting & deployment](https://gohugo.io/hosting-and-deployment/)
  - [Links and Cross References](https://gohugo.io/content-management/cross-references/)

