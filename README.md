# My Hexo Blog Source Codes

## Some Commands
### new 
```bash
$ hexo new [layout] <title> 
```

Creates a new article. If no `layout` is provided, Hexo will use the `default_layout` from [_config.yml](./_config.yml). If the `title` contains spaces, surround it with quotation marks.

### genarate
```bash
$ hexo generate
```

Generates static files.

### publish
```bash
$ hexo publish [layout] <filename>
```

Publishs a draft.

### server
```bash
$ hexo server
```
Starts a local server. By default, this is at [http://localhost:4000](http://localhost:4000)

### deploy
```bash
$ hexo deploy
```

Deploys your website.

## Or Just Run
```bash
$ npm run deploy
```