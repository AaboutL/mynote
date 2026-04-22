---
title: 【ubuntu】命令
updated: 2024-01-30 07:42:32Z
created: 2024-01-19 07:34:37Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

1.  杀掉正在运行程序对应的进程: ps -ef | grep muti.py | awk '{print $2}' | xargs kill -9
    
    ```
    ps -ef | grep 05_similarity_faiss_muti.py | awk '{print $2}' | xargs kill -9
    ```