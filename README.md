# VE DMA sample

This repository includes an exmple to show how to use [VE
DMA](https://veos-sxarr-nec.github.io/libsysve/group__vedma.html).

## Usage of VE DMA

VE DMA can transfer data between System V Shared Memory (shm) on VH and local memory on VE.

### 1. Create SHM on VH

You have to create shm with huge table before using VE DMA.

Example:

```
    int key = 0x19761215;
    size_t size = 256 * 1024 * 1024;
    shmget(key, size, IPC_CREAT | IPC_EXCL | SHM_HUGETLB | 0600);
```


### 2. Attach the shm created on VH and register it to DMAATB on VE

The DMA address translation buffer (DMAATB) is used for address translation from a VE
host virtual address to a VE memory. 

See details on SX-Aurora TSUBASA Architecture Manual (but not yet publically avaiable).

```
    int key = 0x19761215;
    size_t size = 256 * 1024 * 1024;
    int shmid = vh_shmget(key, size, SHM_HUGETLB);

    void* vehva_vh;
    void* p = vh_shmat(shmid, NULL, 0, &vehva_vh);
```

### 3. Init VE DMA

```
    ve_dma_init();
```

### 4. Allocate 64 byte aligned VE memory

The default page size of VE is 64B. Then you have to use 64B aligned buffer.

```
    size_t align = 64 * 1024 * 1024;
    void* vemva;
    posix_memalign(&vemva, align, size);
```

### 5. Register VE memory to DMAATB

```
    uint64_t vehva_ve = ve_register_mem_to_dmaatb(vemva, size);
```

### 6. Post DMA

Transfer size have to be less than 128MB.


```
    size_t transfer_size = 64 * 1024 * 1024;

    // read
    ve_dma_post_wait(vehva_ve, (uint64_t)vehva_vh, transfer_size);

    // write
    ve_dma_post_wait((uint64_t)vehva_vh, vehva_ve, transfer_size);
```

### 7. Dettach the shm and unregister VE memory from DMAATB

```
vh_shmdt(p);
ve_unregister_mem_from_dmaatb(vehva_ve);
```

### 8. Remove the shm

```
% ipcrm -M 0x19761215
```

## Example

- [[mkshm.c]]: creates shm on VH
- [[dma.c]]: transfers data using VE DMA
- [[rdshm.c]]: print data on shm


```
% make
% ./mkshm.x86
% ipcs # you see the created shm with key=0x19761215
% ./dma.ve
% ./rdshm.x86
% ipcrm -M 0x19761215 # remove the shm.
```

## References

- [VE DMA](https://veos-sxarr-nec.github.io/libsysve/group__vedma.html)
- [VH-VE SHM](https://veos-sxarr-nec.github.io/libsysve/group__vhshm.html)
