# VE DMA sample

This repository include an exmple to show how to use [VE
DMA](https://veos-sxarr-nec.github.io/libsysve/group__vedma.html).

## Usage of VE DMA


### 1. Create SHM on VH.

You have to create System V Shared Memory (shm) with huge table.

Example:

```
    int key = 0x19761215;
    size_t size = 256 * 1024 * 1024;
    shmget(key, size, IPC_CREAT | IPC_EXCL | SHM_HUGETLB | 0600);
```


### 2. Attach the shm created on VH and register it to DMAATB on VE.

```
    int key = 0x19761215;
    size_t size = 256 * 1024 * 1024;
    int shmid = vh_shmget(key, size, SHM_HUGETLB);

    void* vehva_vh;
    void* p = vh_shmat(shmid, NULL, 0, &vehva_vh);
```

### 3. Init VE DMA.

```
    ve_dma_init();
```

### 4. Allocate 64 byte aligned VE memory

```
    size_t align = 64 * 1024 * 1024;
    void* vemva;
    posix_memalign(&vemva, align, size);
```

### 5. Register VE memory to DMAATB.

    uint64_t vehva_ve = ve_register_mem_to_dmaatb(vemva, size);
    if (vehva_ve == (uint64_t)-1) {
        perror("ve_register_mem_to_dmaatb");
        return 1;
    }

### 6. Post DMA.

Transfer size have to ve less than 128MB.


```
    size_t transfer_size = 64 * 1024 * 1024;

    // read
    ve_dma_post_wait(vehva_ve, (uint64_t)vehva_vh, transfer_size);

    // write
    ve_dma_post_wait((uint64_t)vehva_vh, vehva_ve, transfer_size);
```

### 7. Dettach the shm and unregister VE memory from DMAATB.

```
    vh_shmdt(p);
    ve_unregister_mem_from_dmaatb(vehva_ve);
```

### 8. Remove the shm.

```
   % ipcrm -M 0x19761215
```

## Example

## References

- [VE DMA](https://veos-sxarr-nec.github.io/libsysve/group__vedma.html)
- [VH-VE SHM](https://veos-sxarr-nec.github.io/libsysve/group__vhshm.html)
