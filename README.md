# python simple use ringbuffer

python实现简单的环形缓冲区，仅供参考！

该算法用于生产者与消费者模式，在两个进程之间传递大对象使用

使用python内置mmap共享内存方法配合yield使用。用于定位数据定位

原理

```shell
pool = pooling(2000)
next(pool)
print(pool.send(1000)) # [0, (0, 1000)]
print(pool.send(200))  # [1000, (0, 200)]
print(pool.send(1000)) # [1200, (800, 200)]
print(pool.send(500))  # [200, (0, 500)]
print(pool.send(500))  # [700, (0, 500)]
print(pool.send(500))  # [1200, (0, 500)]
print(pool.send(500))  # [1700, (300, 200)]
print(pool.send(500))  # [200, (0, 500)]
print(pool.send(500))  # [700, (0, 500)]
```

将该定位数组通过pipe或其它方法传递到另一进程

**欢迎提出改进意见**

## 环形缓冲区算法

```python
class CircularBuffer:
    def __init__(self, ipc_mmap:mmap, ipc_mmap_size: int):
        """
        parameter:
            ipc_mmap: none_mmap object
            ipc_mmap_size: int, mmap size int type
        """
        self.mm:t.BinaryIO[t.IO(bytes)] = ipc_mmap
        self.size:int = ipc_mmap_size
        self.locations = self.__data_location()
        next(self.locations)

    def __data_location(self) -> DATA_LOCATION:

        is_full:bool = False
        start_location: HEAD = 0
        end_location: TAIL = (0, 0)
        size:int = self.size

        while True:
            datasize: int = yield [start_location, end_location]
            if datasize > size:
                logger.error(f'data size exceed mmap size, data size: {datasize}')
                continue

            if is_full:
                start_location = end_location[1]
                is_full = False
            else:
                start_location = (start_location + end_location[1]) % size

            end_location = (0, datasize)

            if (current_datasize := start_location + sum(end_location)) > size:
                end_location = (size - start_location, abs(size - current_datasize))
                is_full = True

```
具体代码详见 [ringbuffer.py](https://github.com/kulongwangzhi85/ringbuffer/blob/main/ringbuffer.py)
