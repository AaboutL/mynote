---
title: 【python】multiprocessing pool.map_async 运行无反应
updated: 2023-05-25 12:55:34Z
created: 2023-05-25 12:54:25Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

所以，我正在开发一个应用程序，每次启动时，它都必须对照哈希列表检查大约50gb的数据。显然这需要并行化，我不希望应用程序挂在“加载…”屏幕上一分钟半。
我使用multiprocessing.Pool的map_async来处理这个问题；主线程调用map_async(checkfiles, path_hash_pairs, callback)并提供一个回调，告诉它在发现不匹配时抛出一个警告。
问题是。。。什么都没发生。使用我的任务管理器查看Python进程，可以看到它们生成，然后立即终止，而不做任何工作。他们从不打印任何东西，当然也从不完成并调用回调。
这个缩小的例子也显示了同样的问题：

def printme(x):
    time.sleep(1)
    print(x)
    return x**2

if __name__ == "__main__":
    l = list(range(0,512))

    def print_result(res):
        print(res)

    with multiprocessing.Pool() as p:
        p.map_async(printme, l, callback=print_result)
    p.join()
    time.sleep(10)

运行它，然后。。。什么都没发生。将map_async替换为map完全符合预期。
我只是犯了个愚蠢的错误还是什么？
最佳答案

让我们看看会发生什么：
您正在使用上下文管理器自动“关闭”Pool，但是，重要的是，如果您检查Pool.__exit__的源代码，您将发现：

def __exit__(self, exc_type, exc_val, exc_tb):
    self.terminate()

它只调用terminate而不是close。所以您仍然需要显式地关闭Pool然后join它。
with multiprocessing.Pool() as p:
    p.map_async(printme, l, callback=print_result)
    p.close()
    p.join()

但在这种情况下，使用context manager是没有意义的，只需使用普通形式：
p = multiprocessing.Pool()
p.map_async(printme, l, callback=print_result)
p.close()
p.join()

为什么它与map一起工作？因为map将阻止所有工作完成。