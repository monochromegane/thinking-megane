# THINKING MEGANE [![wercker status](https://app.wercker.com/status/93e52aa93b59f08d1009b060773a7b29/s/master "wercker status")](https://app.wercker.com/project/bykey/93e52aa93b59f08d1009b060773a7b29)

My blog contents for Hugo.

- [THINKING MEGANE](https://blog.monochromegane.com)

## Getting start

```sh
$ git clone https://github.com/monochromegane/thinking-megane.git
$ git clone https://github.com/monochromegane/hugo-theme-thinking-megane.git themes/thinking-megane
$ hugo server --theme=thinking-megane --watch
```

And access to [http://localhost:1313](http://localhost:1313)

## Add entry

```sh
$ hugo new post/`date '+%Y'`/`date '+%m'`/`date '+%d'`/NEW_POST_NAME.md
$ vi content/!$
```

or

```sh
$ hugo new diary/`date '+%Y%m%d'`.md
$ vi content/!$
```

## Publish

```sh
$ git add .
$ git commit -m "COMMIT LOG"
$ git push origin master
```

And Wercker generates and publish blog contents to GitHub Pages.

See: [wercker.yml](https://github.com/monochromegane/thinking-megane/blob/master/wercker.yml)

## Author

**monochromegane**
