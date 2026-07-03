# amdgpu VRAM P2P DMA Patches

The upstream amdgpu driver (DKMS 6.10.5, ROCm 7.1) pins DMA-buf exported
buffers into GTT (system memory), preventing true P2P DMA between NVMe and
GPU VRAM. These three patches modify `amdgpu_dma_buf.c` to enable VRAM pinning.

## Patches

| # | File | Function | Required | Description |
|---|------|----------|----------|-------------|
| 1 | `0001-bypass-p2p-distance-check.patch` | `amdgpu_dma_buf_attach` | Topology-dependent | Bypass `pci_p2pdma_distance` check |
| 2 | `0002-pin-to-vram-instead-of-gtt.patch` | `amdgpu_dma_buf_pin` | Yes | Pin to VRAM instead of GTT |
| 3 | `0003-accept-vram-in-dma-buf-map.patch` | `amdgpu_dma_buf_map` | Yes | Accept `TTM_PL_VRAM` in map |

Patches 2 and 3 are required. Patch 1 is only needed if `pci_p2pdma_distance`
returns a negative value on your platform.

## Applying

```bash
AMDGPU_SRC=/usr/src/amdgpu-<version>
cd $AMDGPU_SRC

# Apply (review each patch before applying)
for p in /path/to/uGDS/patches/amdgpu/00*.patch; do
    patch -p1 --dry-run < $p   # preview
    patch -p1 < $p             # apply
done

# Rebuild and reload
cd amd/amdgpu
make -C /lib/modules/$(uname -r)/build M=$(pwd) modules
sudo rmmod amdgpu
sudo insmod amdgpu.ko
```

## Verified on

- MI210 (gfx90a), kernel 6.8.0-124, amdgpu DKMS 6.10.5, ROCm 7.1.1
- Samsung 990 PRO NVMe, VRAM P2P DMA address `0x220fxxx` (GPU BAR)
- uGDS functional tests: 20/20 PASS
