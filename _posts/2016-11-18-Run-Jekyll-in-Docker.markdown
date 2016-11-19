---
layout: post
title: "Run Jekyll in Docker"
date: 2016-11-18 06:15:20
categories: Jekyll Docker
---

`--incremental` argument don't work in Windows, But the `force_polling` tag solve this problem. Check out this [caveats](https://github.com/jekyll/docker/wiki/Usage:-Running#caveats)

>If you are on Windows or OS X using Docker-Machine or Boot2Docker you will need to --force_polling or send the environment variable POLLING=true because there is no built-in support for NTFS/HFS notify events to inotify and the verse, you'll be on two different file systems so only the basic API's are implemented. This also applies to docker-machine which either uses boot2docker or is like-it.

Start command:

	docker run --rm --label=jekyll --volume=$(pwd):/srv/jekyll -it -p 4000:4000 jekyll/jekyll jekyll s --force_polling

## Attention

 The Time zone of jekyll image is GMT, so the post you just write might be in the future in the perspective of Jekyll.
 So it will not generate the content.

 I have config the timezone in _config.yml, but seems don't work as well.