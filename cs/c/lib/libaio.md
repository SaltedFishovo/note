#### linux aio syscall

Linux提供了一组异步io的系统调用，libaio在此基础之上进行部分封装。

1. 使用 *io_setup(unsigned nr, aio_context_t \*ctxp)* 生成一个 *aio_context_t* 类型的io上下文，用于代表一批发送给内核的io请求。

   - nr为该批次io数目
   - ctxp 就是io上下文，本质上是一个无符号long
   - 返回值：
     - 成功返回0
     - 失败时返回一个负数作为错误代码，而不设置errno（原生syscall返回-1，设置errno）；

2. 使用 *io_submit(aio_context_t ctx, long nr, struct iocb \*iocbpp[])* 提交一批io请求。
   - *nr* 为本批 *io* 的个数；
   - *iocbpp* 是一个 *struct iocb* *的数组的指针， *nr* 就是该数组中元素的个数；
     - 一个 *strcut iocb* 代表一个 *io* 请求，表示对一个文件进行什么操作；
     - 定义一个新的 *struct iocb* 结构体时要将该结构体清；
   - 返回值：
     -  On success, io_submit() returns the number of iocbs submitted
        (which may be less than nr, or 0 if nr is zero)；
     - 失败同上

3. 使用 *int io_getevents(io_context_t ctx, long min_nr, long nr, struct io_event \*events, struct timespec \*timeout)* 查询对某一批 *io* 操作的结果。
   - 通过 *ctx* 查询具体哪一批被提交的 *io*；
   - 当完成的 *io* 请求数目大于等于第二个参数 *min_nr* 时，该函数才会返回，否则阻塞等待完成的 *io* 数目达到这个值；
   - 第三个参数 *nr* 为本批 *io* 总数；
   - 如果没有完成 *min_nr* 个 *io*请求，又不想永久等待，设置 *timeout* 作为最大等待时间，如果此参数为NULL则永久等待，为0则不阻塞；
   - 内核为每一个完成的 *io* 请求都创建一个 *struct io_event* 结构体, 在至少完成 *min_nr* 个 *io* 请求后在 *events* 中放一个 *struct io_event* 数组的首地址；

   https://man7.org/linux/man-pages/man2/io_getevents.2.html



#### 数据结构

```c
struct iocb {
	PADDEDptr(void *data, __pad1);	/* Return in the io completion event */
	PADDED(unsigned key, __pad2);	/* For use in identifying io requests */
 
	short		aio_lio_opcode;	
	short		aio_reqprio;
	int		aio_fildes;
 
	union {
		struct io_iocb_common		c;
		struct io_iocb_vector		v;
		struct io_iocb_poll		poll;
		struct io_iocb_sockaddr	saddr;
	} u;
};

struct io_iocb_common {
	PADDEDptr(void	*buf, __pad1); 	//buf start ptr
	PADDEDul(nbytes, __pad2);	//buf size
	long long	offset;		//file offset
	long long	__pad3, __pad4;
};

typedef enum io_iocb_cmd {
	IO_CMD_PREAD = 0,
	IO_CMD_PWRITE = 1,
 
	IO_CMD_FSYNC = 2,
	IO_CMD_FDSYNC = 3,
 
	IO_CMD_POLL = 5,
	IO_CMD_NOOP = 6,
} io_iocb_cmd_t;



struct io_event {
	PADDEDptr(void *data, __pad1);
	PADDEDptr(struct iocb *obj,  __pad2);
	PADDEDul(res,  __pad3);
	PADDEDul(res2, __pad4);
};
```



#### libaio

libaio将这组系统调用进行了封装，使得使用aio更加便捷。

```c
static inline void io_prep_pread(struct iocb *iocb, int fd, void *buf, size_t count, long long offset)
{       
        memset(iocb, 0, sizeof(*iocb));
        iocb->aio_fildes = fd; 
        iocb->aio_lio_opcode = IO_CMD_PREAD;
        iocb->aio_reqprio = 0;
        iocb->u.c.buf = buf;
        iocb->u.c.nbytes = count;
        iocb->u.c.offset = offset;
}

static inline void io_prep_pwrite(struct iocb *iocb, int fd, void *buf, size_t count, long long offset);
static inline void io_prep_preadv(struct iocb *iocb, int fd, const struct iovec *iov, int iovcnt, long long offset)
{       
        memset(iocb, 0, sizeof(*iocb));
        iocb->aio_fildes = fd; 
        iocb->aio_lio_opcode = IO_CMD_PREADV;
        iocb->aio_reqprio = 0;
        iocb->u.c.buf = (void *)iov;
        iocb->u.c.nbytes = iovcnt;
        iocb->u.c.offset = offset;
}

static inline void io_prep_pwritev(struct iocb *iocb, int fd, const struct iovec *iov, int iovcnt, long long offset);
```



1. *setup*

2. 定义*N* 个 *struct iocb*  和一个 *struct iocb \* array[N]*，将这些 *iocb* 的指针加入这个数组。
3. 使用 *libaio* 提供的 *io_prep_\** 系列函数初始化每个 *struct iocb*。
4. *io_submit（）*
5. *io_getenvts()* 
