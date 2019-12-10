# vmstat

## Basic Introduction

vmstat (virtual memory statistics) is a system utility that collects and displays information about system memory, 
processes, interrupts, paging and block I/O. Using vmstat, you can specify a sampling interval to observe system activity in near-real time.

## Example

```c
[JOHNSONPC]$ cat /proc/vmstat 
nr_free_pages 988432
nr_inactive_anon 40007
nr_active_anon 26426
nr_inactive_file 524353
nr_active_file 362887
nr_unevictable 0
nr_mlock 0
nr_anon_pages 62093
nr_mapped 15870
nr_file_pages 891579
nr_dirty 17
nr_writeback 0
nr_slab_reclaimable 53746
nr_slab_unreclaimable 4908
nr_page_table_pages 2683
nr_kernel_stack 417
nr_unstable 0
nr_bounce 0
nr_vmscan_write 0
nr_writeback_temp 0
nr_isolated_anon 0
nr_isolated_file 0
nr_shmem 4339
nr_anon_transparent_hugepages 0
pgpgin 11111337
pgpgout 13551616
pswpin 0
pswpout 0
pgalloc_dma 30287
pgalloc_normal 9931551
pgalloc_high 153256157
pgalloc_movable 0
pgfree 164207611
pgactivate 3271687
pgdeactivate 555098
pgfault 378791991
pgmajfault 136420
pgrefill_dma 0
pgrefill_normal 517850
pgrefill_high 37248
pgrefill_movable 0
pgsteal_dma 0
pgsteal_normal 1363749
pgsteal_high 0
pgsteal_movable 0
pgscan_kswapd_dma 0
pgscan_kswapd_normal 1364160
pgscan_kswapd_high 0
pgscan_kswapd_movable 0
pgscan_direct_dma 0
pgscan_direct_normal 0
pgscan_direct_high 0
pgscan_direct_movable 0
pginodesteal 0
slabs_scanned 3820800
kswapd_steal 1363749
kswapd_inodesteal 1281148
kswapd_low_wmark_hit_quickly 2818
kswapd_high_wmark_hit_quickly 2875
kswapd_skip_congestion_wait 0
pageoutrun 27749
allocstall 0
pgrotated 503
compact_blocks_moved 0
compact_pages_moved 0
compact_pagemigrate_failed 0
compact_stall 0
compact_fail 0
compact_success 0
htlb_buddy_alloc_success 0
htlb_buddy_alloc_fail 0
unevictable_pgs_culled 7673
unevictable_pgs_scanned 0
unevictable_pgs_rescued 6732
unevictable_pgs_mlocked 9439
unevictable_pgs_munlocked 9439
unevictable_pgs_cleared 0
unevictable_pgs_stranded 0
unevictable_pgs_mlockfreed 0

```