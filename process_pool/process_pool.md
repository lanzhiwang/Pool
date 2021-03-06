## 进程池开发过程笔记
```
pool初始化
=============================method __init__ in class Pool 
=============================method _setup_queues in class Pool
# 从 _inqueue 队列中 get 任务，然后执行，将任务执行结果 put 进 _outqueue 队列
self._inqueue:  <multiprocessing.queues.SimpleQueue object at 0x7fb8723f5f90> type: <class 'multiprocessing.queues.SimpleQueue'>
# 从  _outqueue 队列获取结果，更新 ApplyResult 对象的相关属性
self._outqueue:  <multiprocessing.queues.SimpleQueue object at 0x7fb872407910> type: <class 'multiprocessing.queues.SimpleQueue'>
self._quick_put:  <built-in method send of _multiprocessing.Connection object at 0x55cf81ac62f0> type: <type 'builtin_function_or_method'>
self._quick_get:  <built-in method recv of _multiprocessing.Connection object at 0x55cf81a8cdb0> type: <type 'builtin_function_or_method'>
=============================method __init__ in class Pool 
# 将外部的任务 put 进 _taskqueue 队列用于后续多个 process 处理
self._taskqueue:  <Queue.Queue instance at 0x7fb8723ab7a0> type: <type 'instance'>
# Pool 对象和 ApplyResult 对象共享数据
self._cache:  {} type: <type 'dict'>
self._state:  0 type: <type 'int'>
self._maxtasksperchild:  None type: <type 'NoneType'>
self._initializer:  None type: <type 'NoneType'>
self._initargs:  () type: <type 'tuple'>
self._processes:  2 type: <type 'int'>
self._pool:  [] type: <type 'list'>
=============================method _repopulate_pool in class Pool 
# worker process 处理任务进程
# 从 _inqueue 队列中 get 任务，然后执行，将任务执行结果 put 进 _outqueue 队列
<Process(Process-1, initial)>
<Process(Process-2, initial)>
self._pool:  [<Process(PoolWorker-1, started daemon)>, <Process(PoolWorker-2, started daemon)>] type: <type 'list'>
=============================method __init__ in class Pool 
self._worker_handler:  <Thread(Thread-1, started daemon 140430153918208)> type: <class 'threading.Thread'>
# 该线程从 _taskqueue 队列获取任务，将任务 put 进 _inqueue 队列
self._task_handler:  <Thread(Thread-2, started daemon 140430145525504)> type: <class 'threading.Thread'>
# 该线程从  _outqueue 队列获取结果，更新 ApplyResult 对象的相关属性
self._result_handler:  <Thread(Thread-3, started daemon 140430062647040)> type: <class 'threading.Thread'>
self._terminate:  <Finalize object, callback=_terminate_pool, args=(<Queue.Queue instance at 0x7fb8723ab7a0>, <multiprocessing.queues.SimpleQueue object at 0x7fb8723f5f90>, <multiprocessing.queues.SimpleQueue object at 0x7fb872407910>, [<Process(PoolWorker-1, started daemon)>, <Process(PoolWorker-2, started daemon)>], <Thread(Thread-1, started daemon 140430153918208)>, <Thread(Thread-2, started daemon 140430145525504)>, <Thread(Thread-3, started daemon 140430062647040)>, {}), exitprority=15> type: <class 'multiprocessing.util.Finalize'>


def apply(self, func, args=(), kwds={}):
    return self.apply_async(func, args, kwds).get()

def apply_async(self, func, args=(), kwds={}, callback=None):
    result = ApplyResult(self._cache, callback)
	self._taskqueue.put(([(result._job, None, func, args, kwds)], None))  # 此时 self._task_handler 开始执行
	return result

class ApplyResult(object):

    """
    ApplyResult(self._cache, callback)

    self._cond
    self._job
    self._cache
    self._ready
    self._callback
    self._success
    self._value
    """
    def __init__(self, cache, callback):
        print '=============================method __init__ in class ApplyResult '
        self._cond = threading.Condition(threading.Lock())
        self._job = job_counter.next()  # job_counter = itertools.count()
        self._cache = cache
        self._ready = False
        self._callback = callback
        cache[self._job] = self

========================================================================
### 实现apply和map方法
```bash
lanzhiwang@lanzhiwang-dev2:~/rzx_project/cpython/Lib$ python -m multiprocessing.process_pool multiprocessing/process_pool.py
12331
12332
<__main__.Pool object at 0x7f5f82dec4d0>
=============================method __init__ in class ApplyResult 
self._cond:  <Condition(<thread.lock object at 0x7f5f84b94390>, 0)> type: <class 'threading._Condition'>
self._job:  0 type: <type 'int'>
self._cache:  {0: <__main__.MapResult object at 0x7f5f82d91c90>} type: <type 'dict'>
self._ready:  False type: <type 'bool'>
self._callback:  None type: <type 'NoneType'>
{0: <__main__.MapResult object at 0x7f5f82d91c90>}
self._success:  True type: <type 'bool'>
self._value:  [None, None, None, None, None, None, None, None, None, None] type: <type 'list'>
self._chunksize:  2 type: <type 'int'>
self._number_left:  5 type: <type 'int'>
self._ready:  False type: <type 'bool'>
(<generator object <genexpr> at 0x7f5f82d9c7d0>, None)
<type 'tuple'>
thread handle_tasks:  <generator object <genexpr> at 0x7f5f82d9c7d0> None
enumerate thread handle_tasks:  0 (0, 0, <function mapstar at 0x7f5f84acb0c8>, ((<function sqr at 0x7f5f82deb9b0>, (0, 1)),), {})
worker process:  (0, 0, <function mapstar at 0x7f5f84acb0c8>, ((<function sqr at 0x7f5f82deb9b0>, (0, 1)),), {})
worker process:  (True, [0, 1])
enumerate thread handle_tasks:  1 (0, 1, <function mapstar at 0x7f5f84acb0c8>, ((<function sqr at 0x7f5f82deb9b0>, (2, 3)),), {})
worker process:  (0, 1, <function mapstar at 0x7f5f84acb0c8>, ((<function sqr at 0x7f5f82deb9b0>, (2, 3)),), {})
worker process:  (True, [4, 9])
enumerate thread handle_tasks:  2 (0, 2, <function mapstar at 0x7f5f84acb0c8>, ((<function sqr at 0x7f5f82deb9b0>, (4, 5)),), {})
worker process:  (0, 2, <function mapstar at 0x7f5f84acb0c8>, ((<function sqr at 0x7f5f82deb9b0>, (4, 5)),), {})
worker process:  (True, [16, 25])
enumerate thread handle_tasks:  3 (0, 3, <function mapstar at 0x7f5f84acb0c8>, ((<function sqr at 0x7f5f82deb9b0>, (6, 7)),), {})
worker process:  (0, 3, <function mapstar at 0x7f5f84acb0c8>, ((<function sqr at 0x7f5f82deb9b0>, (6, 7)),), {})
worker process:  (True, [36, 49])
enumerate thread handle_tasks:  4 (0, 4, <function mapstar at 0x7f5f84acb0c8>, ((<function sqr at 0x7f5f82deb9b0>, (8, 9)),), {})
worker process:  (0, 4, <function mapstar at 0x7f5f84acb0c8>, ((<function sqr at 0x7f5f82deb9b0>, (8, 9)),), {})
worker process:  (True, [64, 81])
handle_results:  (0, 0, (True, [0, 1]))
success:  True type: <type 'bool'>
result:  [0, 1] type: <type 'list'>
self._value:  [0, 1, None, None, None, None, None, None, None, None] type: <type 'list'>
handle_results:  (0, 1, (True, [4, 9]))
success:  True type: <type 'bool'>
result:  [4, 9] type: <type 'list'>
self._value:  [0, 1, 4, 9, None, None, None, None, None, None] type: <type 'list'>
handle_results:  (0, 2, (True, [16, 25]))
success:  True type: <type 'bool'>
result:  [16, 25] type: <type 'list'>
self._value:  [0, 1, 4, 9, 16, 25, None, None, None, None] type: <type 'list'>
handle_results:  (0, 3, (True, [36, 49]))
success:  True type: <type 'bool'>
result:  [36, 49] type: <type 'list'>
self._value:  [0, 1, 4, 9, 16, 25, 36, 49, None, None] type: <type 'list'>
handle_results:  (0, 4, (True, [64, 81]))
success:  True type: <type 'bool'>
result:  [64, 81] type: <type 'list'>
self._value:  [0, 1, 4, 9, 16, 25, 36, 49, 64, 81] type: <type 'list'>
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
lanzhiwang@lanzhiwang-dev2:~/rzx_project/cpython/Lib$ 

```








```