# 网站图片优化之 —— WebP格式
JPEG 2000，JPEG XR和WebP是与JPEG和PNG图像相比较，具有更好的压缩和更高质量。以这些格式而不是JPEG或PNG编码图像会有更快的加载速度。
例如同等品质的图片WebP格式比JPEG格式文件大小会减少25%-35%。

## 如何将图片转换为WebP格式

### 工具-cwebp安装
	yum install libwebp-tools


### 转换图片命令 shell
	for file in images/*; do cwebp "$file" -o "${file%.*}.webp"; done

### html代码，兼容不支持webp格式的浏览器
	<picture>
	  <source type="image/webp" srcset="flower.webp">
	  <source type="image/jpeg" srcset="flower.jpg">
	  <img src="flower.jpg" alt="beautiful flower">
	</picture>