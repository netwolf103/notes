# inotifywait图片监控 & 转webp格式

### 需要工具cwebp & inotifywait
	# cwebp安装
	yum -y install libwebp-tools

	# inotifywait 安装
	yum -y install inotify-tool

### vim img2webp.sh
	#!/bin/sh
	# -----------------------------------------------------------------------------------------------
	# Filename: img2webp.sh
	# Version: 0.0.1
	# Date: 2019/11/30 2:05:10
	# Author: Zhang Zhao
	# Description: Monitor newly added png and jpg images in a directory and convert to webp format.
	# -----------------------------------------------------------------------------------------------

	IMG_DIR=/var/www/html/media

	/usr/bin/inotifywait -mrq -e create $IMG_DIR | while read path action file;
	do
	    OLDFILE="$path$file"
	    NEWFILE="$OLDFILE.webp"

	    if [[ $(file -b $OLDFILE) =~ ^('PNG '|'JPEG ') ]]; then
	        cwebp $OLDFILE -o $NEWFILE
	        chown apache:apache $NEWFILE
	    fi
	done

### 赋予可执行权限
	chmod +x img2webp.sh

### 执行 & 添加进系统启动自运行
	/command path/img2webp.sh & echo "/bin/sh /command path/img2webp.sh &" >> /etc/rc.local