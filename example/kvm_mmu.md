Finally, I reached my original purpose to learn systemtap.

> Investigate how kvm mmu system works

Now let systemtap to show its power.

# Investigate kvm_mmu_pages and mmu_page_path in mmu_zap_unsync_children

To utilize memory efficiently, kvm would zap unsync children. This is achieved
in mmu_zap_unsync_children() with two helpers: kvm_mmu_pages and mmu_page_path.

With the help of systemtap, we could have a close look at how they works.

Here it the systemtap script

```
global started
probe begin { started = 0 }
probe module("kvm").statement("*@arch/x86/kvm/mmu.c:2605")
{
	level_2 = 0
        for (i = 0; i < $pages->nr; i++) {
		if ($pages->page[i]->sp->role->level == 2)
			level_2++
        }
	printf("level2 %d\n", level_2);
        if (level_2 < 2)
                next;
	started = $parent;
        printf("Dump pages after mmu_unsync_walk \n");
	printf("--------------------------------\n");
        printf("SP Index:  0  1  2  3  4  5  6  7  8 9 10 11 12 13 14 15\n");
        printf("SP Level:");
        for (i = 0; i < $pages->nr; i++) {
		printf("%3d", $pages->page[i]->sp->role->level);
        }
        printf("\n");
        for (i = 0; i < $pages->nr; i++) {
		printf("[%d]: %x\n", i, $pages->page[i]->sp);
        }
}

probe module("kvm*").function("mmu_zap_unsync_children").return
{
	if (started == @entry($parent))
		exit();
}

probe module("kvm*").statement("*@arch/x86/kvm/mmu.c:2609")
{
	if (!started)
		next;
        printf("Dump parents after for_each_sp \n");
	printf("--------------------------------\n");
	printf("Current sp   : [%d]%x\n", $i, $sp);
	printf("Level in Parents:");
	for (idx = 0; idx < 5; idx++) {
		if (!$parents->parent[idx])
			break;
		printf("%3d", $parents->parent[idx]->role->level);
	}
	printf("\n");
	printf("SP in Parents:\n")
	for (idx = 0; idx < 5; idx++) {
		if (!$parents->parent[idx])
			break;
		printf("[%d]: %x\n", idx, $parents->parent[idx]);
	}
}
```

Then is the result

```
Dump pages after mmu_unsync_walk
--------------------------------
SP Index:  0  1  2  3  4  5  6  7  8 9 10 11 12 13 14 15
SP Level:  4  3  2  1  1  1  3  2  1
[0]: ffff8dd3aea02aa0
[1]: ffff8dd3c8ab18c0
[2]: ffff8dd3d48a4140
[3]: ffff8dd3c7fd3aa0
[4]: ffff8dd3b1152280
[5]: ffff8dd3b1152460
[6]: ffff8dd3c8ab1000
[7]: ffff8dd3c322bb40
[8]: ffff8dd3b4b46f00
Dump parents after for_each_sp
--------------------------------
Current sp   : [3]ffff8dd3c7fd3aa0
Level in Parents:  2  3  4
SP in Parents:
[0]: ffff8dd3d48a4140
[1]: ffff8dd3c8ab18c0
[2]: ffff8dd3aea02aa0
Dump parents after for_each_sp
--------------------------------
Current sp   : [4]ffff8dd3b1152280
Level in Parents:  2  3  4
SP in Parents:
[0]: ffff8dd3d48a4140
[1]: ffff8dd3c8ab18c0
[2]: ffff8dd3aea02aa0
Dump parents after for_each_sp
--------------------------------
Current sp   : [5]ffff8dd3b1152460
Level in Parents:  2  3  4
SP in Parents:
[0]: ffff8dd3d48a4140
[1]: ffff8dd3c8ab18c0
[2]: ffff8dd3aea02aa0
Dump parents after for_each_sp
--------------------------------
Current sp   : [8]ffff8dd3b4b46f00
Level in Parents:  2  3  4
SP in Parents:
[0]: ffff8dd3c322bb40
[1]: ffff8dd3c8ab1000
[2]: ffff8dd3aea02aa0
```

Do you find something in the output?

```
Dump pages after mmu_unsync_walk
--------------------------------
SP Index:  0  1  2  3  4  5  6  7  8 9 10 11 12 13 14 15
SP Level:  4  3  2  1  1  1  3  2  1
```
First in "Dump pages after mmu_unsync_walk" part, we found there is a patten of
page level in kvm_mmu_pages. When mmu_unsync_walk traverses the tree, it
traverses to leaf (level 1) and then go to another subtree (level 3 in case it
has). To be simple, this is a depth first traverse.

```
Dump parents after for_each_sp
--------------------------------
Current sp   : [3]ffff8dd3c7fd3aa0
Level in Parents:  2  3  4
SP in Parents:
[0]: ffff8dd3d48a4140
[1]: ffff8dd3c8ab18c0
[2]: ffff8dd3aea02aa0
```

Then take a look at the "Dump parents after for_each_sp" part. This output
tries to show the effect of for_each_sp.

Each iteration of for_each_sp, mmu_page_path will contain **a path** from root
to a leaf node (SP in Parents).

And the leaf (Current sp) will be passed to zap process.

That's interesting.

# Dump vcpu root_hpa

I am curious about the content of root_hpa of each vcpu in a kvm, then tried this example.

```
probe module("kvm*").function("vcpu_run")
{
        kvm = $vcpu->kvm;
        for (idx = 0; kvm->vcpus[idx]; idx++) {
                printf("vcpu[%d] root_hpa %x\n",
                        kvm->vcpus[idx]->vcpu_id,
                        kvm->vcpus[idx]->arch->mmu->root_hpa);
        }
        exit();
}
```

Then found 

> ROOT_HPA in vcpu are not the same

The result looks:

```
vcpu[0] root_hpa 2f394000
vcpu[1] root_hpa 2dd82000
vcpu[2] root_hpa 29f71000
vcpu[3] root_hpa 2b044000
vcpu[4] root_hpa 30ba3000
vcpu[5] root_hpa 29f9c000
vcpu[6] root_hpa 2abd7000
vcpu[7] root_hpa 29cc7000
```

And another round it looks:

```
vcpu[0] root_hpa a5c3f000
vcpu[1] root_hpa a1948000
vcpu[2] root_hpa 1004a0000
vcpu[3] root_hpa af305000
vcpu[4] root_hpa c9f83000
vcpu[5] root_hpa 15605a000
vcpu[6] root_hpa 1d4a51000
vcpu[7] root_hpa 12eea0000
```

This means the root_hpa always change. Originally, I think they will not change
in their life time.

# Investigate ROOT_HPA Invalidation

Each mmu->root_hpa is actually the cr3 in guest. With each task switch in
guest, mmu->root_hpa should change accordingly.

Current kvm implements cr3 cache in mmu->prev_roots[], so it will search the
cache first to see whether the cr3 is there. While if not, mmu->root_hpa will
be set to invalid.

And then, on vcpu_enter_guest(), kvm_mmu_load() will be invoked since
mmu->root_hpa is invalid to create a new cr3.

My understanding is more tasked running in guest, more kvm_mmu_load() will be
invoked to get a new cr3. Here is a systemtap script to observe the case.

```
global sum, mmu_load
probe module("kvm*").function("kvm_mmu_load")
{
        sum++
}
probe timer.s(1)
{
        mmu_load <<< sum;
        sum = 0;
}
probe timer.s(10)
{
        print(@hist_linear(mmu_load, 0, 100, 10));
        delete mmu_load
}
```

Every 1 second the script will gather the total amounts in this period and
every 10 seconds, it will dump the statistics.

The result is obvious.

When guest is idle:

```
value |-------------------------------------------------- count
    0 |@@@@@@@@@@                                         10
   10 |                                                    0
   20 |                                                    0
```

which means task switch is not frequent.

When guest is building kernel:

```
value |-------------------------------------------------- count
   90 |                                                    0
  100 |                                                    0
 >100 |@@@@@@@@@@                                         10
```

which means guest is running heavily.

While I am wondering is this a heavy burden? or we could improve at this point?
