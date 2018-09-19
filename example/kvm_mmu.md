Finally, I reached my original purpose to learn systemtap.

> Investigate how kvm mmu system works

Now let systemtap to show its power.

# Dump level of page in kvm_mmu_pages

To utilize memory efficiently, kvm would zay unsync children. It will traverse
the SP and form a kvm_mmu_pages to record potential unsync children.

Actually, the level of those pages in kvm_mmu_pages has a patten. Now let's
use systemtap to prove it.

Here it the systemtap script

```
probe module("kvm").function("mmu_pages_first")
{
        if ($pvec->nr <= 4)
                next;
        printf("kvm_mmu_pages level (%s)\n", pp());
        printf("SP Index:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15\n");
        printf("SP Level:");
        for (i = 0; i < $pvec->nr; i++) {
                printf("%3d", $pvec->page[i]->sp->role->level);
        }
        printf("\n");
        exit();
}
```

Then is the result

```
kvm_mmu_pages level (module("kvm").function("mmu_pages_first@arch/x86/kvm/mmu.c:2264"))
SP Index:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15
SP Level:  4  3  2  1  1  3  2  1  1  3  2  1
```

Do you find something in the output?

Every time, it traverses to the leaf (level 1) and then go to another subtree.
To be simple, this is a depth first traverse.

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
