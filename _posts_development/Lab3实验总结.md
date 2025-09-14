---
title: Lab3实验总结
tags:
  - Lab
categories:
  - [计组]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-06-05 20:52:07
---

# 1 前言

这个编程实验需要实现一个简单的cache, 并尝试实现各种替换算法来优化程序的性能. 在代码目录下执行

```bash
make
```

来编译, 生成可执行文件 `a.out` . 其运行方式如下：

```bash
./a.out [-r seed] [trace]
```

其中 `seed` 是随机种子, 可以用于确定性回放帮助调试, 缺省时会用系统时间作为种子; `trace` 是 `bz2` 压缩格式的访存序列, 缺省时会产生随机访存序列来测试.

> 具体见讲义：https://zhong-kangwei.gitee.io/ics-pa-gitbook-2022/lab3.html

# 2 实现

## init_cache （20分）

* cache结构设计

  * 思路

    主存$32KB=2^{15}B$，块$64B=2^6B$，cache容量$16KB=2^{14}B$，行数$16KB/64B=2^8B$；由于四路组相联，所以cache组号占$2^8B/4=6$位，所以Tag占$15-6-6=3$位。

  * 代码

    ```c
    typedef struct{
    	bool valid_bit:1;//valid bit
    	bool tag:3;//Tag
    	bool data[64];//block: data
    	bool dirty_bit:1;//dirty bit
    }Cache;
    
    Cache *cache = NULL;
    ```

* cache初始化

  * 思路
    算出总行数，然后遍历cache并将所有valid bit置为无效即可。

  * 代码

    ```c
    void init_cache(int total_size_width, int associativity_width) {
    	int lines = exp2( total_size_width - BLOCK_WIDTH );//number of lines??
    	//printf("%d\n", lines);
    	cache = malloc( lines * sizeof(Cache) );//init a cache
    	
    	//Set all valid bits as invalid
    	for( int i=0; i < lines; i++ ){
    		cache[i].valid_bit = false;
    	}
    }
    ```

## cache_read (30分)

* 思路：首先将主存地址进行划分，然后分情况讨论：

  * 命中成功，在相关cache组中找到符合条件(tag位相同且有效位有效)的cache行，则直接从cache中读出；
  * 命中失败，但在该组中有空闲行，则到主存读取一块信息并设置标记位；
  * 命中失败且在该组中未找到空闲行，则任意替代该组的某一行并设置cache。

* 代码

  ```c
  //get data from cache
  uint32_t get_data(int num, int offset){
  	uint32_t ret = cache[num].data[offset] + 
  		(cache[num].data[offset + 1] << 8) + 
  		(cache[num].data[offset + 2] << 16) + 
  		(cache[num].data[offset + 3] << 24);
  	return ret;
  }
  
  uint32_t cache_read(uintptr_t addr) {
  	try_increase(1);	//visit cache times ++
  	
  	//Memory address division
  	addr = addr & 0x7FFF;
  	uint32_t tag, cache_group_num, block_offset, block_num;	
  	block_offset = addr & 0x3C;//block addr: 6 bits, 4 bytes alignment
  	cache_group_num = (addr >> 6) & 0x3F;//cache group number: 6 bits
  	tag = (addr >> 12) & 0x7;//tag: 3 bits
  	block_num = (tag << 6) | cache_group_num;//memory block number
  
  	uint32_t paddr, ret_data;
  	bool flag = false;//flag hit or not
  	int start = cache_group_num * 4;//the start of a group
  	int	end   = start + 3;          //the end of a group
  	
  	//Find the corresponding group	
  	for( int i = start ; i <= end ; i++ ){
  		//Successful hit
  		if( cache[i].tag == tag && cache[i].valid_bit ){
  			hit_increase(1);
  			flag = true;
  			ret_data = get_data(i, block_offset);
  			break;
  		}
  	}
  	//failed hit
  	if( !flag ){
  		bool flg = false;// flag that is there a free line
  		for( int i = start ; i <= end; i++ ){
  			//find a free line
  			if( !cache[i].valid_bit  ){
  				flg = true;
  				//set cache
  				mem_read(block_num, cache[i].data);//read data from memory
  				cache[i].tag = tag;
  				cache[i].valid_bit = true;
  				cache[i].dirty_bit = false;
  				ret_data = get_data(i, block_offset); 
  				break;
  			}
  		}
  		//cannot find a free line, 
  		if( !flg ){
  			int rand_line = start + rand()%4;//find a random line
  			//if dirty, write first
  			if( cache[rand_line].dirty_bit ){
  				paddr = ( cache[rand_line].tag << 6 ) | cache_group_num ;
  				mem_write(paddr, cache[rand_line].data);
  			}
  			//set cache
  			mem_read(block_num, cache[rand_line].data);
  			cache[rand_line].tag = tag;
  			cache[rand_line].valid_bit = true;
  			cache[rand_line].dirty_bit = false;
  			ret_data = get_data(rand_line, block_offset);
  		}
  	}
  
  	return ret_data;
  
  	//return 0;
  }
  ```

## cache_write (30分)

* 思路：首先将主存地址进行划分，然后分情况讨论(与读类似)：

  * 命中成功，在相关cache组中找到符合条件(tag位相同且有效位有效)的cache行，则根据写掩码写入cache并置脏位；
  * 命中失败，但在该组中有空闲行，则根据写掩码写入cache并设置标记位；
  * 命中失败且在该组中未找到空闲行，则随机取一行；判断脏位是否为1，若为1则将当前的cache行写回主存，然后根据写掩码写入cache并设置cache。

* 代码

  ```c
  //set the data of cache by wmask
  void set_cache(int num , uint32_t offset , uint32_t wmask , uint32_t data){
  	uint32_t standard_data;
  	standard_data = get_data(num, offset); 
  	standard_data = ( standard_data & ( ~ wmask) ) | (data & wmask);
  
  	cache[num].data[offset]     = standard_data         & 0xFF;
  	cache[num].data[offset + 1] = (standard_data >>  8) & 0xFF;
  	cache[num].data[offset + 2] = (standard_data >> 16) & 0xFF;
  	cache[num].data[offset + 3] = (standard_data >> 24) & 0xFF;
  }
  
  void cache_write(uintptr_t addr, uint32_t data, uint32_t wmask) {
  	try_increase(1);
  	//Memory address division
  	addr = addr & 0x7FFF;		
  	uint32_t tag, cache_group_num, block_offset, block_num;
  	block_offset = addr & 0x3C;			
  	cache_group_num = (addr >> 6) & 0x3F;
  	tag = (addr >> 12) & 0x7;
      block_num = (tag << 6) | cache_group_num;
  
  	uint32_t paddr;
  	bool flag = false;// flag write hit or not
  	int start = cache_group_num * 4;//the start of a group
  	int end   = start + 3;			//the end of a group
  
  	//Find the corresponding group
  	for( int i = start ; i <= end; i++ ){
  		//successful hit
  		if( cache[i].tag == tag && cache[i].valid_bit ){
  			hit_increase(1);
  			flag = true;
  			set_cache(i, block_offset, wmask, data);//write data
  			cache[i].dirty_bit = true;
  			break;
  		}
  	}
  	//failed hit
  	if( !flag ){
  		bool flg = false;//flag that is there a free line
  		for( int i = start ; i <= end; i++ ){
  			//find a free line
  			if( !cache[i].valid_bit ){
  				flg = true;
  				mem_read(block_num, cache[i].data);//read data from memory
  				set_cache(i, block_offset, wmask, data);//write data to cache
  				cache[i].tag = tag;	
  				cache[i].valid_bit = true;
  				cache[i].dirty_bit = true;
  				break;
  			}
  		}
  		//cannot find a free line
  		if( !flg ){
  			int rand_line = start + rand()%4;//find a random line
  			//if dirty, write first
  			if( cache[rand_line].dirty_bit ){
  				paddr = ( cache[rand_line].tag << 6 ) | cache_group_num;
  				mem_write(paddr, cache[rand_line].data);
  			}
  			//set cache
  			mem_read(block_num, cache[rand_line].data);
  			set_cache(rand_line, block_offset, wmask , data);
  			cache[rand_line].tag = tag;
  			cache[rand_line].valid_bit = true;
  			cache[rand_line].dirty_bit = true;
  		}
  	}
  }
  ```

## 最终结果截图（20分）

<img src="https://s2.loli.net/2022/06/05/TkSoKYvaZm7dDq1.jpg" width = "800" height = "200" alt="图片名称" align=center id=206 />

## 思考题

* **问**：数据对齐和存储层次结构：想一想, 为什么编译器为变量分配存储空间的时候一般都会对齐? 访问一个没有对齐的存储空间会经历怎么样的过程?
* **答**：需要字节对齐的根本原因在于CPU访问数据的效率问题。若存储空间未对齐，则可能出现多次访存，然后组合成目标数据，因此带来指令执行效率的降低。
* **参考**：
  * [为什么需要字节对齐](https://blog.csdn.net/hunanchenxingyu/article/details/53942407)
