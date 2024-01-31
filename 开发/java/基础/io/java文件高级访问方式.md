<font color=red>注意：由于jvm存在，java  传统io永远都要多一层拷贝(堆外内存、堆内内存之间的拷贝)，一次读取或者写入共3层拷贝</font>

### 直接IO

<font color=red>少一层用户缓存到内核缓存的拷贝，一次读取或者写入共2层拷贝</font>

跳过内核缓存（pageCache），用户缓存直接与磁盘交互，一般用于数据库软件、消息队列软件等（为了实现更好的缓存命中率），编码复杂，容易出问题。官方JDK没有支持，需要使用三方框架或自己封装C语言。

### 直接内存

<font color=red>少一层JVM堆内到堆外的拷贝，一次读取或者写入共2层拷贝</font>

```java
/**
	 * 直接内存方式
	 * @param filename
	 * @param dest
	 */
	public static void copyDirect(String filename,String dest) {
		long start=System.currentTimeMillis();
		FileChannel in=null;
		FileChannel out=null;
		try {
			in=FileChannel.open(Paths.get(filename),StandardOpenOption.READ);
			//这个通道 不仅要在缓存区读数据  还要向磁盘写数据  所以要同时拥有读写功能    StandardOpenOption.CREATE_NEW的意思是如果有该文件报异常  如果没有创建新文件
			out=FileChannel.open(Paths.get(dest),StandardOpenOption.WRITE,StandardOpenOption.CREATE);
			ByteBuffer buf = ByteBuffer.allocateDirect(1024);
			while(in.read(buf)!=-1){
				buf.flip();//切换成读数据模式
				out.write(buf);
				buf.clear();//将所有指针置为最初位置
			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			if(in!=null){
				try {
					in.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			if(out!=null){
				try {
					out.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
		long end=System.currentTimeMillis();
		System.out.println("直接内存"+(end-start));
	}
```

### 内存映射（mmap）

<font color=red>少一层用户缓存到内核缓存的拷贝，使用用户内核共享缓存，一次读取或者写入共2层拷贝</font>

```java
/***
	 * 内存映射方式
	 * @param filename 要赋值的文件路径
	 * @param dest  文件赋值的目的地  如果有该文件报异常  如果没有创建新文件
	 * 该方法直接操作文件本身，非常快，该内存的管理由操作系统管理，
	 *  对于该内存的释放只有JVM虚拟机断开对该内存的引用时会被回收，不可控制，不稳定，占用内存大
	 */
	public static void copyXMap(String filename,String dest) {
		long start=System.currentTimeMillis();
		FileChannel in=null;
		FileChannel out=null;
		try {
			in=FileChannel.open(Paths.get(filename),StandardOpenOption.READ);
			//这个通道 不仅要在缓存区读数据  还要向磁盘写数据  所以要同时拥有读写功能    StandardOpenOption.CREATE_NEW的意思是如果有该文件报异常  如果没有创建新文件
			out=FileChannel.open(Paths.get(dest),StandardOpenOption.READ,StandardOpenOption.WRITE,StandardOpenOption.CREATE);
			/*in.size其实是整个文件的大小对于大多数操作系统，将文件映射到内存中比通过通常的read和write方法
			 * 读取或写入几十千字节的数据更 昂贵 。 从性能的角度来看，通常只能将较大的文件映射到内存中
			 */
			MappedByteBuffer inMap = in.map(MapMode.READ_ONLY, 0, in.size());//相当于ByteBuffer.allocateDirect(capacity)
			MappedByteBuffer outMap = out.map(MapMode.READ_WRITE, 0, in.size());//相当于ByteBuffer.allocateDirect(capacity)
			byte[] dst = new byte[1024];
			inMap.get(dst);
			outMap.put(dst);
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			if(in!=null){
				try {
					in.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			if(out!=null){
				try {
					out.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
		long end=System.currentTimeMillis();
		System.out.println("内存映射"+(end-start));
	}
```

### 零拷贝

<font color=red>直接在内核内对接管道，零拷贝</font>

方法将当前通道中的数据传送到目标通道target中，在支持Zero-Copy的linux系统中，transferTo()的实现依赖于 sendfile()调用。
```java
/**
	 * 零拷贝方式
	 * @param filename 要赋值的文件路径
	 * @param dest   文件赋值的目的地  如果有该文件报异常  如果没有创建新文件
	 * 可以将字节直接从源通道传输到文件系统高速缓存中，而无需实际复制它们
	 */
	public static void copyXTranform(String filename,String dest) {
		long start=System.currentTimeMillis();
		FileChannel in=null;
		FileChannel out=null;
		try {
			in=FileChannel.open(Paths.get(filename),StandardOpenOption.READ);
			//这个通道 不仅要在缓存区读数据  还要向磁盘写数据  所以要同时拥有读写功能    StandardOpenO
			//ption.CREATE_NEW的意思是如果有该文件报异常  如果没有创建新文件
			//可以将字节直接从源通道传输到文件系统高速缓存中，而无需实际复制它们
			out=FileChannel.open(Paths.get(dest),StandardOpenOption.WRITE,StandardOpenOption.CREATE);
			in.transferTo(0, in.size(), out);
			//out.transferFrom(in, 0, in.size());
		}catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			if(in!=null){
				try {
					in.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			if(out!=null){
				try {
					out.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
		long end=System.currentTimeMillis();
		System.out.println("transfer方法"+(end-start));
	}
```