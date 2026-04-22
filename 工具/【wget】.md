---
title: 【wget】
updated: 2023-11-07 09:00:01Z
created: 2023-07-03 11:03:50Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

wget -c -r -np -nc -L -p -R index.html ftp://ftp-trace.ncbi.nlm.nih.gov

注意：大小写敏感！大写和小写命令代表不同操作
-P 表示下载到哪个目录
-r 表示递归下载，下载指定网页某一目录下（包括子目录）的所有文件
-np 不要追溯到父目录
-k 表示将下载的网页里的链接修改为本地链接.（下载整个站点后脱机浏览网页，最好加上这个参数
-p 获得所有显示网页所需的元素，如图片等
-c 断点续传
-nd 递归下载时不创建一层一层的目录，把所有的文件下载到当前目录
-o 将log日志指定保存到文件（新建一个文件）
-a, –append-output=FILE 把记录追加到FILE文件中
-A 指定要下载的文件样式列表，多个样式用逗号分隔
-A zip 只下载指定文件类型（zip）
-N 不要重新下载文件除非比本地文件新
-O test.zip 下载并以不同的文件名保存
-nc 不要覆盖存在的文件或使用.#前缀
-m, –mirror 等价于 -r -N -l inf -nr
-L 递归时不进入其它主机，如wget -c -r www.xxx.org/ 如果网站内有一个这样的链接： www.yyy.org，不加参数-L，就会像大火烧山一样，会递归下载www.yyy.org网站
-i 后面跟一个文件，文件内指明要下载的URL（常用于多个url下载
-nc 不要重复下载已存在的文件 --no-clobber
